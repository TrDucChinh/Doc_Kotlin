# Generic & Scope Function
## Generic
- `Generics` là một tính năng mà cho phép chúng ta có thể định nghĩa và truy cập các `classes`, `methods`, `properties` bằng cách sử dụng các kiểu dữ liệu khác nhau mà vẫn sẽ hoạt động giống nhau.
- Các `class` trong `Kotlin`  có thể chứa các `type parameters` (Kiểu dữ liệu truyền vào) giống như Java.

```Kotlin
class Box<T>(t: T) {
    var value = t
    fun show(){
        print(t)
    }
	
}
```
- Tạo instance

```Kotlin
val box1: Box<Int> = Box<Int>(1)
print(box1.value)         // 1
val box2 = Box("2")
print(box2.value)          // 2
Box<String>("Test").show() // Test
```

### Một số quy ước
> Tên tham số kiểu là các chữ cái viết hoa, đơn lẻ. Việc này nhằm giúp ta phân biệt giữa một biến kiểu và một lớp thông thường hay tên giao diện. Và cũng giúp cho các thành viên trong team dễ review code hơn.

- `T`: Type (Kiểu dữ liệu bất kỳ thuộc Wrapper class: String, Integer, Long, Float, … hoặc các class)
- `E`: Element (phần tử – được sử dụng phổ biến trong Collection Framework)
- `K`: Key (khóa)
- `V`: Value (giá trị)
- `N`: Number (kiểu số: Integer, Double, Float, …)

### Phương sai khai báo (Declaration-site variance)
- Trong `Java`, giả sử có một `interface Source` không có `methods` có tham số là `T` và chỉ có hàm trả về kiểu `T`
```Java
// Java
interface Source<T> {
    T nextT(); //Hàm kiểu trả về là T
}
```
- Việc lưu trữ một tham chiếu của đối tượng `Source<String>` trong một biến có kiểu là `Source<Object>` sẽ an toàn nhưng Java không hiểu và cấm việc đó (Bởi vì trong Java generic là bất biến nên `Source<String>`không phải là con của `Source<Object>` mặc dù String là subtype của Object)

```Java
// Java
void demo(Source<String> strs) {
    Source<Object> objects = strs; // !!! Not allowed in Java
    // ...
}
```
- Trong Kotlin, có một cách để khiến trình biên dịch hiểu là dùng từ khoá `in`, `out`
```Kotlin
interface Source<out T> { // T is covariant type parameter.
    fun nextT(): T
}

fun demo(strs: Source<String>) {
    val objects: Source<Any> = strs // This is OK, since T is an out-parameter
    objects.nextT()
}
```

#### Covariant (out)
- Khi một lớp generic được khai báo là `covariant`, nghĩa là nó cho phép sự thay thế kiểu con. Ví dụ, nếu `A` là một kiểu dữ liệu và `B` là một kiểu con của `A`, khi lớp `generic Container` được khai báo là `covariant`, ta có thể gán một đối tượng `Container<B>` cho một biến kiểu `Container<A>`. Điều này cho phép ta sử dụng các phương thức hoặc thuộc tính của `Container<A>` trên đối tượng `Container<B>`, vì B là một kiểu con của A.

#### Contravariant (in)
- Khi một lớp `generic` được khai báo là `contravariant`, nghĩa là nó cho phép sự thay thế kiểu cha. Ví dụ, nếu `A` là một kiểu dữ liệu và `B` là một kiểu cha của `A`, khi lớp `generic Consumer` được khai báo là `contravariant`, ta có thể gán một đối tượng `Consumer<A>` cho một biến kiểu `Consumer<B>`. Điều này cho phép ta sử dụng các phương thức hoặc thuộc tính của `Consumer<B>` trên đối tượng `Consumer<A>`, vì `A` là một kiểu cha của `B`.
```Kotlin
interface Comparable<in T> {
    operator fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
    x.compareTo(1.0) // 1.0 has type Double, which is a subtype of Number
    // Thus, you can assign x to a variable of type Comparable<Double>
    val y: Comparable<Double> = x // OK!
}
```
### Loại dự báo (Type projections)
#### Phương sai loại dự báo (Use-site variance: type projections)
- Trong thực tế nhiều lớp không chỉ trà về kiểu dữ liệu trừu tượng và một ví dụ điển hình là `Array` và nó không uyển chuyển. Ta có hàm sau để copy dữ liệu từ một array sang array khác.
- Ví dụ ta có hàm `copy Array` này sang `Array` khác
```Kotlin
val integers: Array<Int> = arrayOf(1, 2, 3, 4)
val any = Array<Any>(4) { "" }
copy(integers, any) // Lôỗi

fun copy(from: Array<Any>, to: Array<Any>) {
    assert(from.size == to.size)
    for (i in from.indices)
        to[i] = from[i]
}
```
- Ta thấy `Array<T>` là bất biến nên cả `Array<Any>` và `Array<Int>` đều không là subtype của kiểu kia
- Cách giải quyết
```Kotlin
val integers: Array<Int> = arrayOf(1, 2, 3, 4)
val any = Array<Any>(4) { "" }
copy(integers, any)
any.forEach { println(it) } // 1 2 3 4 5
fun copy(from: Array<out Any>, to: Array<Any>) {
    assert(from.size == to.size)
    for (i in from.indices)
        to[i] = from[i]
}
```
#### Dự báo sao (Star-projections)
- Đôi khi ta không biết về kiểu của đối số truyền vào, nhưng vẫn muốn sử dụng nó một cách an toàn. Giải pháp ở đây là xác định kiểu chung chung, sao cho mọi biểu tượng cụ thể của kiểu chung đó sẽ là một kiểu con của phép dự báo đó.
- Kotlin cung cấp **Star-Projections** cho điều này:
    - Đối với `Foo<out T: TUpper>`, trong đó `T`là tham số `covariant type` với `TUpper` giới hạn trên, `Foo <*>` tương đương với `Foo<out TUpper>`. Điều này có nghĩa là khi `T` không xác định, bạn có thể đọc các giá trị của `TUpper` một cách an toàn từ `Foo <*>`.
    - Đối với `Foo <in T>`, trong đó `T` là tham số `contravariant type`, `Foo <*>` tương đương với `Foo <in Nothing>`. Điều này có nghĩa là bạn không thể viết gì cho `Foo <*>` một cách an toàn khi `T`không rõ.
    - Đối với `Foo <T: TUpper>`, trong đó `T` là tham số kiểu `invariant type` với `TUpper` giới hạn trên, `Foo <*>` tương đương với `Foo <out TUpper`> để đọc giá trị và `Foo <in Nothing>` để ghi giá trị.
- Nếu một kiểu chung có một số tham số kiểu, mỗi tham số trong số chúng có thể được đối chiếu độc lập. Ví dụ: nếu kiểu được khai báo là `Interface Funtion<in T, out U>`, bạn có thể sử dụng các phép `Star-Projections` sau:
    - `Function<*, String>` có nghĩa là `Function<in Nothing, String>`.
    - `Function<Int, *>` có nghĩa là `Function<Int, out Any?>`.
    - `Function<*, *>` có nghĩa là `Function<in Nothing, out Any?>`.

### Hàm Generics (Generic Function)
```Kotlin
fun <T> createList(item: T): List<T> {
    return listOf(item)
}

fun <T> List<T>.customFilter(filterFunction : (T) -> (Boolean)) : List<T>{
    val list = mutableListOf<T>()
    for(it in this){
        if(filterFunction(it)){
            list.add(it)
        }
    }
    return list
}
fun main() {
    val list = createList(1)
    println(list.customFilter { it == 1 })
}
```
### Generic constraints
- `Generic constraints` (ràng buộc generic) trong Kotlin là các quy tắc áp dụng cho kiểu dữ liệu generic để giới hạn loại kiểu mà kiểu dữ liệu generic có thể nhận.
- Khi khai báo một tham số kiểu `generic`, ta có thể áp dụng `Generic constraints` để chỉ định loại kiểu mà tham số kiểu đó có thể nhận. `Generic constraints` giúp ta đảm bảo rằng các phương thức và thuộc tính trong kiểu generic sẽ chỉ hoạt động với các kiểu thỏa mãn ràng buộc đã chỉ định.
- **Ràng buộc kiểu (Type constraints)**: Được áp dụng bằng cách sử dụng từ khóa `where` sau khai báo kiểu generic. Ràng buộc kiểu chỉ định một hoặc nhiều kiểu mà tham số kiểu phải thỏa mãn.
```Kotlin
class Container<T> (private val data : T) where T : Number {
    fun print(){
        println("$data")
    }
}
val container = Container(5)
container.print()  // 5
val container2 = Container("Haha") // Loi
```
Ràng buộc hàm (Function constraints): Được áp dụng bằng cách sử dụng từ khóa T : Typetrực tiếp trên tham số kiểu trong khai báo hàm. Ràng buộc hàm chỉ định rằng tham số kiểu T phải thỏa mãn một ràng buộc kiểu cụ thể. Ví dụ:

```Kotlin
fun <T : Comparable<T>> max(a: T, b: T): T {
    return if (a > b) a else b
}
```
## Extension Function
- `Extension function` trong `Kotlin` cho phép bạn thêm các phương thức vào các lớp đã có sẵn mà không cần phải thay đổi mã nguồn của chúng. Điều này rất hữu ích khi bạn muốn mở rộng chức năng của các lớp trong các thư viện hoặc mã nguồn không được điều khiển bởi bạn
- Các Extension được viết thêm chỉ là các phương thức/thuộc tính có thể gọi thông qua dấu chấm `.`
```Kotlin
fun Int.isPrime() : Boolean{
    for(i in (2..sqrt(this.toDouble()).toInt())){
        if(this % i == 0) return false
    }
    return this >= 2
}
val x = 10
val y = 17
println(x.isPrime()) // false
println(x.isPrime()) // true
```
- `This` ở trên tham chiếu đến đối tượng gọi đến phương thức đó (`x` và `y`)
- Tuy nhiên, `Extension Function` không thể truy cập thành phần `private` của `class`, không thể `override` các hàm thành phần của `class` gốc.

```Kotlin
class Person (val name : String,  val age : Int){
    private val myPrivateName : String = "Mr. $name"
    fun introduce() =  println("Name: $name, Age: $age")
}
fun Person.showPrivateName() : Unit{
    println(myPrivateName) // Loi 
}
fun Person.introduce() = println("Name: $name")
//Van in ra Name + Age chu ko phai Name

```

## Scope Function
- `Scope Funtion` trong `Kotlin` là những hàm cho phép bạn thực thi một đoạn code trên phạm vi một đối tượng. Những đoạn code này thường được gọi là `lambda expression` hoặc `lambda function`
- `Scope functions` cung cấp một cú pháp đặc biệt để truy cập vào đối tượng gốc và thực hiện các tác vụ trên nó một cách ngắn gọn và dễ đọc. Các hàm `scope function` trong Kotlin bao gồm: `let`, `run`, `with`, `apply` và `also`.
- Các Scope Function được phân loại dựa trên:
  - Context Object
  - Return Value

|Function|Object reference|Return value|Is extension function|
|---|---|---|---|
|let|it|Lambda result|Yes|
|run|this|Lambda result|Yes|
|run|-|Lambda result|No: called without the context object|
|with|this|Lambda result|No: takes the context object as an argument.|
|apply|this|Context object|Yes|
|also|it|Context object|Yes|

- Cách sử dụng:
    - Làm việc với một Null Object: `let`
    - Dùng biểu thức như một biến trong Local Scope: `let`
    - Cấu hình lại Object: `apply`
    - Cấu hình lại Object và có giá trị trả về: `run`
    - Không cần có Object nhưng lại có giá trị trả về `run`
    - Thực hiện một chuỗi thao tác bổ sung cho đối tượng: `also`
    - Sử dụng nhiều Funtion trên 1 Object: `with`

### Context Object
- Trong Scope Funtion, mỗi Context Object có thể được truy cập thông qua một tham chiếu ngắn gọn thay vì sử dụng tên của nó. Có 2 cách để truy cập
    - Lambda Receiver (this)
    - Lambda Agrument (it)

#### it
- **`(block: (T) -> R)`**: Đây là một dạng `lambda expression` thông thường, trong đó đối tượng được truyền vào `lambda` như một đối số. Bạn phải sử dụng tham số trong `lambda` để truy cập các thành phần của đối tượng.
- Dùng `it` khi: Sử dụng chính `object` đó như một đầu vào (là tham số duy nhất cho `lambda`), dùng để tham chiếu đến đối tượng hiện tại của `Lambda`.
```Kotlin
fun findSquare() : (Int) -> Unit = {n ->
    println(n * n)
}
val func = findSquare()
func(3)  //9
```
- Ở ví dụ trên, hàm `findSquare()` trả về một `lambda` nhận tham số là một số `Int` và trả về `Unit`. Vì trong hàm `lambda` trả về chỉ có một tham số duy nhất nên ta có thể viết gọn lại bằng cách dùng từ khoá `it`

```Kotlin
fun findSquare() : (Int) -> Unit = {
    println(it * it)
}
val func = findSquare()
func(3)  //9
```
- Nếu hàm `Lambda` có 2 tham số thì không thể dùng từ khoá `it` để thay thế
```Kotlin
fun findSquare(): (Int, Int) -> Int = { x: Int, y: Int ->
    x * y
}
```
#### thís
- Đây là một dạng `lambda expression` có `receiver`, trong đó đối tượng được truyền vào `lambda` và được xem như `receiver`. Bạn có thể trực tiếp truy cập các thành phần của đối tượng mà không cần sử dụng tham số.
- Receiver: `A.(B) -> C` → Các hàm có thể gọi được trên một đối tượng A (là một `receiver` và được tham chiếu qua từ khoá `this`) với tham số truyền vào là B và kiểu trả về là C.
- Khi `receiver` đã được xác định ta có thể bỏ qua từ khoá `this`
```Kotlin
val sum: Int.(Int) -> Int = { x -> x + this }
val res = 6.sum(4) //10


fun sub(): Int.(Int) -> Int = { x ->
    this - x
}

fun main() {
    val result = 6.sub(3)
    println(result) // 3
}
```
- Biến `sum` có kiểu funtion type với `receiver` . Nghĩa là `sum` được gọi trên đối tượng `reicever` là kiểu `Int`, nhận tham số `Int`, trả về `Int`. `x` ở bên trong hàm `lambda` đại diện cho tham số truyền vào (số 4) và `this` đại diện cho `reicever object` (tức là cái số để mình gọi cái hàm `sum`).
- Tuy nhiên vì trong `lambda` chỉ có duy nhất một tham số truyền vào kiểu `Int` nên ta có thể dùng `it` để thay thế :>>
```Kotlin
val sum: Int.(Int) -> Int = { it + this }
fun sub() : Int.(Int) -> Int = {
    this - it
}
```
### Return Value
#### Context Object

```Kotlin
val numberList = mutableListOf<Double>()
numberList.also { println("Populating the list") }
    .apply {
        add(2.71)
        add(3.14)
        add(1.0)
    }
    .also { println("Sorting the list") }
    .sort()
```
- Được sử dụng trong các câu lệnh `Return` của các funtion mà trả về `Context Object`
```Kotlin
fun getRandom() : Int{
    return (1..100).random().also {
        println("Random value: $it")
    }
}
```
#### Lambda Result
```Kotlin
val name: String? = "Zin"
    val formattedName = name?.let {
        val formatted = it.uppercase(Locale.getDefault())
        "Mr. $formatted"
    }

    println(formattedName) // Output: Mr. Zin
```
- Hàm `Let` trả về kết quả của `Lambda` (hiểu đơn giản là Lambda return gì thì hàm `Let` return cái đó). Ở ví dụ trên thì `Lambda` trả về một chuỗi `"Mr. $formatted"` nên let sẽ trả về chuỗi đó và gán cho biến f`ormattedName` 
### Các Scope Funtion
#### Let
- Context Object: `it`
- Return Value: `Lambda result`
- Được sử dụng để gọi hàm trên kết quả của một chuỗi hàm
```Kotlin
val numbers = mutableListOf("one", "two", "three", "four", "five")
numbers.map { it.length }.filter { it > 3 }.let { 
    println(it)
    // and more function calls if needed
}
```
- Sau hàm `map` thì sẽ trả về một đối tượng bao gồm độ dài các phần tử trong list ban đầu, sau `filter` thì cũng trả về một đối tượng là list mà các phần tử trong list có độ dài lớn hơn 3. Như vậy qua 2 hàm `map` và `filter` thì `Object` mà ta nhận được là một `List<Int>` và đối tượng này được làm tham số trong hàm `let`
- Nếu code chỉ chứa một hàm với `it` làm đối số thì có thể sử dụng `::`
```Kotlin
val str = "Hello World!"
val length = str.let(String::length)
println(length)
```
- Thường được sử dụng cho các `Object` có thể `Null`
```Kotlin
val str: String? = "Hello"   
//processNonNullString(str)       // compilation error: str can be null
val length = str?.let { 
    println("let() called on $it")        
    processNonNullString(it)      // OK: 'it' is not null inside '?.let { }'
    it.length
}
```
#### With
- Context Object: `this`
- Return value: `Lambda result`
- Không phải là một `Extension Function`. `Context Object` được truyền vào như là một đối số, nhưng bên trong `lambda`, nó được sử dụng như một `Receiver (This)`
- Nên sử dụng `with funtion` để gọi hàm vào một `context object` mà không cung cấp `lambda result`.
```Kotlin
val numbers = mutableListOf("one", "two", "three")
with(numbers) {
    println("'with' is called with argument $this")
    println("It contains $size elements")
}
```
- Dùng để giới thiệu một đối tượng có các thuộc tính, hàm để tính toán giá trị
```Kotlin
val numbers = mutableListOf("one", "two", "three")
val firstAndLast = with(numbers) {
    "The first element is ${first()}," +
    " the last element is ${last()}"
}
println(firstAndLast)
```
#### Run
- Context Object: `this`
- Return value: `Lambda result`
- Giống `With`, nhưng được triển khai như một `Extension Funtion`. Bạn có thể gọi nó trên `Context Object`
- Hữu dụng khi `lambda` không chỉ khởi tạo đối tượng mà còn có giá trị trả về
```Kotlin
val person = Person("Dung", 30)

  val age = person.run {
      celebrateBirthday()
      calculateAge()
      age
  }

  println("Tuổi của người là: $age") //30
```
- Có thể sử dụng `Run` như là một `non-Extension Funtion` (Không có `Context Object` nhưng vẫn trả về `Lambda Result`)
```Kotlin
val person = run {
	  val name = "John"
    val age = 30
    Person(name, age)
}
val age = person.run {
    celebrateBirthday()
    calculateAge()
    age
}
println("Tuổi của người là: $age")
```
#### Apply
- Context Object: `this`
- Return value: `object itself`
- Vì `Apply` trả về chính nó, nên được khuyến nghị sử dụng cho các đoạn code không trả về một giá trị và chủ yếu là làm việc trên các `members` của đối tượng. Trường hợp phổ biến của `Apply` là cấu hình `Object`
```Kotlin
val person = Person("Dung1", 18).apply {
	      introduce()
        name = "Dung2"
        age = 30
    }
    person.introduce()
//Name: Dung1, Age: 18
//Name: Dung2, Age: 30
```
#### Also
- Context Object: `It`
- Return value: `Object itself`
- `Also` rất hữu ích khi muốn thực hiện một số hành động mà lấy `Context Object` làm đối số. Sử dụng `Also` cho các hành động cần tham chiếu đến đối tượng chứ không phải thuộc tính và phương thức của nó
```Kotlin
val a = mutableListOf(1, 2, 3)
    a.also{
        println(a)
    }.add(4).also{
        println(a)
    }
    // [1, 2, 3]
    // [1, 2, 3, 4]
```