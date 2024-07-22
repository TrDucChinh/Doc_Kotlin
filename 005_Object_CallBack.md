- [Object \& Callback](#object--callback)
  - [Object](#object)
    - [Object Expression](#object-expression)
      - [Abstract Class, Interface](#abstract-class-interface)
      - [Scraft](#scraft)
      - [Return và Value Type](#return-và-value-type)
        - [Inline, Public](#inline-public)
      - [Accessing variables](#accessing-variables)
    - [Object Declarations](#object-declarations)
      - [Data Object](#data-object)
    - [Companion Object](#companion-object)
    - [Khác biệt giữa object expression và declaration](#khác-biệt-giữa-object-expression-và-declaration)
  - [CallBack](#callback)
    - [Higher-order functions](#higher-order-functions)
      - [Function Type](#function-type)
      - [Các cách gọi](#các-cách-gọi)
        - [Function reference](#function-reference)
        - [Lambda Expression](#lambda-expression)
        - [Function Anonymous](#function-anonymous)
    - [Lambda Funtion](#lambda-funtion)
    - [Passing trailing lambdas](#passing-trailing-lambdas)
    - [It](#it)
    - [Anonymous function](#anonymous-function)
    - [Closures](#closures)
    - [CallBack](#callback-1)

# Object & Callback
## Object
- Trong lập trình, `object` (đối tượng) là một thực thể trong bộ nhớ của máy tính, được định nghĩa bởi một lớp (class). Đối tượng bao gồm dữ liệu (các thuộc tính hay biến) và các phương thức (hàm) để thao tác với dữ liệu đó.
- Khi ta muốn sử dụng một object có một chút thay đổi so với class gốc, và chỉ sử dụng 1 lần . Kotlin đã có 2 khái niệm cho nó `object expression`  và `object declaration`
- Thường thường thì khi muốn có một Object thì phải định nghĩa một Class nào đấy và tạo Instance từ Class đấy nhưng với Object thì ta có thể tạo một đối tượng không cần thông qua Class
### Object Expression
- Tạo bởi đối tượng của **`Anonymous Class`**
- Được sử dụng một lần duy nhất
- Có thể định nghĩa ngay từ đầu, kế thừa các  **`Class`** hoặc triển khai **`Interface`**
- **`Instance`** của **`Anonymous Class`** được gọi là **`Anonymous Object`** vì nó được xác định bằng biểu thức chứ không phải tên
#### Abstract Class, Interface
- Giống Java, ta không thể khởi tạo **`Instance`** cho một **`Abstract Class`** một cách bình thường
```Kotlin
abstract class Doc( val title: String, val author: String, var words: Long = 0L) {
    abstract fun summary(): String
}

val article = Doc("A nice article", "Kai",420) // won't compile!
```
- Chúng ta có thể sử dụng **``Object Expression``** để khởi tạo **`Instance`** của **`Anonymous Class`** kế thừa Abstract Class
```Kotlin
val article = object : Doc(title = "A nice article", author = "Kai", words = 420) {
    override fun summary() = "Title: <$title> ($words words) By $author"
}
```
- Tương tự như **`Abstract  Class`**, ta cũng không thể khợi tạo **`Instance`** cho một **`Interface`** một cách bình thường mà phải sử dụng **`Object Expression`**
```Kotlin
interface Printable {
    val content: String
    fun print(): String
}
val sentence = object : Printable {
    override val content: String = "A beautiful sentence."
    override fun print(): String = "[Print Result]\n$content"
}
```
- Nếu có nhiều thứ để kế thừa thì viết cách nhau bởi dấu **`,`**
```Kotlin
open class A(x: Int) {
    open val y: Int = x
}

interface B { /*...*/ }

val ab: A = object : A(1), B {
    override val y = 15
}
```
#### Scraft
- **`Object expressions`** bắt đầu với tứ khóa **`object`**
- Dùng khi **`object`** cần tạo không cần kế thừa từ **`class`** nào
- Khai báo các **`members`** trong cặp ngoặc nhọn **`{}`** sau **`object`**
```Kotlin
val player = object {
        val name = "Kai"
        val gamePlayed = 6L
        val points = 42L
        fun pointsPerGame() = "$name: AVG points per Game: ${points / gamePlayed}"
    }

println(player.pointsPerGame())
```
- **`Player`** là một đối tượng ẩn danh mà không có **`supertype`** rõ ràng
- Code Java cho ví dụ trên
```Kotlin
var player = new Object() {
    String name = "Kai";
    Long gamePlayed = 6L;
    String pointsPerGame() {
        return "$name: AVG points per Game: ${points / gamePlayed}"
    }
};
player.pointsPerGame();
String playerName = player.name;
```
#### Return và Value Type
- Chúng ta có thể sử dụng **`Anonymous Object`** làm giá trị trả về của một **`Method`**, và nếu **`Anonymous Object`** được trả về bởi một **`Private Method`**, chúng ta có thể truy cập đến **`Member`** của nó
```Kotlin
class PlayerService() {
    private fun giveMeAPlayer() = object {
        val name = "Kai"
        val gamePlayed = 6L
        val points = 42L
        fun pointsPerGame() = "$name: AVG points per Game: ${points / gamePlayed}"
    }

    fun getTheName(): String {
        val thePlayer = giveMeAPlayer()
        print(thePlayer.pointsPerGam())
        return thePlayer.name
    }
}
```
- Trong đoạn code trên **`Method giMeAPlayer()`** trả về một **`Anonymous Object`** và **`Access Modifier`** của nó là **`Private`** nên trong **`method getTheName()`** ta có thể truy cập đếm các **`member`** của nó, nếu không phải là **`Private`** thì không truy cập được
##### Inline, Public
- Nếu `function hay property` là `public` hoặc `private inline`, type của nó sẽ là:
    - `Any` nếu `anonymous object` không có `supertype`
    - Chính là `supertype đã được khai báo` của anonymous object, nếu chỉ có đúng `1 supertype`
    - Là `type được khai báo cụ thể nếu có nhiều hơn 1 supertypes` (nếu không cụ thể type thì sẽ lỗi)
- Trong tất cả những trường hợp trên, members được thêm vào anonymous object đều `not accessible` . `Overriden members` thì `accessible` nếu được khai báo với đúng type của function hay property.

```Kotlin
interface A {
    fun funFromA() {}
}
interface B

class C {
    // The return type is Any; x is not accessible
    fun getObject() = object {
        val x: String = "x"
    }

    // The return type is A; x is not accessible
    fun getObjectA() = object: A {
        override fun funFromA() {}
        val x: String = "x"
    }

    // The return type is B; funFromA() and x are not accessible
    fun getObjectB(): B = object: A, B { // explicit return type is required
        override fun funFromA() {}
        val x: String = "x"
    }
}
```
#### Accessing variables
- Đoạn mã trong các biểu thức object expression có thể truy cập vào các biến từ phạm vi bao quanh (enclosing scope).
### Object Declarations
- Dùng để tạo ra đối tượng Singleton
```Kotlin
object ObjectManager {
    private val objects = mutableListOf<Any>()

    fun addObject(obj: Any) {
        objects.add(obj)
    }

    fun removeObject(obj: Any) {
        objects.remove(obj)
    }

    fun printObjects() {
        for (obj in objects) {
            println(obj.toString())
        }
    }
}
```
- Không giống **`Object Expression`**, **`Object Declaration`** không phải là 1 biểu thức vì vậy nó không thể được sử dụng trong các phép gán biến.
```Kotlin
val ObjectExpression = object{ //OK vi la Expresion
    fun doSomething(){
        println("Doing something...")
    }
}
val ObjectDeclaration = object MySingleton { // Loi
    fun doSomething() {
        println("Doing something...")
    }

}
```
- Để sử dụng, ta truy cập thông qua tên của nó
```Kotlin
val obj1 = object {
        val name = "Object 1"
        val value = 10
    }
ObjectManager.addObject(obj1)

val obj2 = object {
        val name = "Object 2"
        val value = 20
}
ObjectManager.addObject(obj2)
ObjectManager.printObjects()
```
- **`Object declarations`** không thể khai báo local (không thể được lồng (nested) trực tiếp bên trong một function), nhưng có thể được lồng trong  một object declaration khác hoặc một non-inner class.
```Kotlin
object OuterObject {
    private val outerProperty = "Outer Property"

    object InnerObject {
        private val innerProperty = "Inner Property"
        fun test (){
            object Test { // Lỗi
                val x = 1
            }
        }
        fun printProperties() {
            println(outerProperty) // Truy cập outerProperty từ InnerObject
            println(innerProperty)
        }
    }
}

fun main() {
    OuterObject.InnerObject.printProperties()
}
```
- Trong ví dụ này, chúng ta có một object declaration có tên **`OuterObject`**. Bên trong **`OuterObject`**, chúng ta lồng thêm một object declaration khác có tên **`InnerObject`**.
- Trong **`InnerObject`**, chúng ta có một phương thức **`printProperties()`** để in ra các thuộc tính của cả **`OuterObject`** và **`InnerObject`**. Chúng ta có thể truy cập **`outerProperty`** từ **`InnerObject`** mà không cần tạo một instance mới của **`OuterObject`**.
- Trong hàm **`main()`**, chúng ta gọi phương thức **`printProperties()`** trực tiếp từ **`OuterObject.InnerObject`** để in ra các thuộc tính.
#### Data Object
- Khi in một Object Declaration thì sẽ được tên và hash code của nó
```Kotlin
object MyObject

fun main() {
    println(MyObject) // MyObject@1f32e575
}
```
- Giống như `data classes`, có thể đánh dấu một `object declaration` với `data modifier` . Khi đó object sẽ có các functions sẵn có như sau:
    - `toString()` trả về tên của data object
    - `equals()`/`hashCode()` (không tự custom được)
- Không có `fun Copy()` (vì là Singleton)
- Không có `Companion Object` : Vì Object không có Constructor
- Thích hợp với `Sealed Class`
### Companion Object
- Giống Static trong Java
- Được khai báo bên trong Class
- Truy cập thông qua tên class
```Kotlin
class MyClass {
    companion object Factory {
        fun create(): MyClass = MyClass()
    }
}

val instance = MyClass.Factory.create()
```
- Có thể lược bỏ tên
```Kotlin
class MyClass {
    companion object {
        fun create(): MyClass = MyClass()
    }
}
val instance = MyClass.create()
```
- 1 Class chỉ có thể có 1 Companion Object.
- Tuy giống ***Static*** trong Java nhưng **Companion Object** là 1 đối tượng, có thể kế thừa các Class hay implement các ***Interface***.
```Kotlin
interface Factory<T> {
    fun create(): T
}

class MyClass {
    companion object : Factory<MyClass> {
        override fun create(): MyClass = MyClass()
    }
}

val f: Factory<MyClass> = MyClass
```
### Khác biệt giữa object expression và declaration
- Object Expression bản chất nó là một biểu thức (gán được, có trả về), được khởi tạo và thực thi ngay khi Object được khởi tạo
- Object Declaration thì không trả về, nó tạo ra một instance object trong bộ nhớ (singleton), kiểu kiểu Lazy Init (cần thì mới khởi tạo)

## CallBack
### Higher-order functions
- Là funtions có param là function hoặc trả về funtion
```Kotlin
fun <T>List<T>.customFilter(filterFunction : (T) -> (Boolean)) : List<T>{
    var list = mutableListOf<T>()
    for(it in this){
        if(filterFunction(it)){
            list.add(it)
        }
    }
    return list
}
```
- **`customFilter`** là một **`extension funtion`** có một param là một funtion **`filterFuntion`** . **`fitlterFuntion`** là hàm nhận vào một đối tượng kiểu **`T`** và trả về **`boolean`**
#### Function Type
- Là một kiểu dữ liệu thay vì là (Int, Double) thì là hàm
- Để biểu diễn một `Function Type` thì ta có một số quy tắc sau:
    - Tất cả các kiểu dữ liệu phương thức đều phải bao gồm đủ phần ***tham số*** và ***kiểu trả về***
    Ví dụ: `(A, B) -> C)` nghĩa là hàm này cần truyền vào 2 tham số là `A` và `B` và trả về kiểu là `C` (Trong trường hợp không trả về gì thì `C` sẽ phải là `Unit`)
    - Kiểu dữ liệu phương thức có thể sử dụng kiểu ***receiver***
    Ví dụ: `A.(B) -> C` nghĩa là hàm này được gọi từ **receiver** `A`, hàm này có tham số là `B` và trả về kiểu dữ liệu `C`
- Ngoài ra còn một số đặc điểm khác của **Function Type:**
    - Ta có thể khai báo tên cho các tham số của phương thức
    Ví dụ: `(x: Int, y: Int) -> Point`
    - Để khai báo **kiểu dữ liệu phương thức** mà có thể ***null***, ta thêm dấu `?` vào sau
    Ví dụ: `((Int, Int) -> Int)?`
    - Ta có thể đặt tên cho **kiểu dữ liệu phương thức** đó bằng cách sử dụng từ khóa `typealias` 
    Ví dụ: `typealias ClickHandler = (Button, ClickEvent) -> Unit`
> Lưu ý: Dấu mũi tên **`->`** sẽ được xử lý từ phải qua trái. Ví dụ:
>**`(Int) -> (Int) -> Unit`** sẽ tương đương với **`(Int) -> ((Int) -> Unit)`** chứ không phải là **`((Int) -> (Int)) -> Unit`**

#### Các cách gọi
##### Function reference
- Function reference (tham chiếu hàm) trong Kotlin là một cách để tham chiếu đến một hàm mà không gọi nó.
```Kotlin
fun <T>List<T>.customFilter(filterFunction : (T) -> (Boolean)) : List<T>{
    var list = mutableListOf<T>()
    for(it in this){
        if(filterFunction(it)){
            list.add(it)
        }
    }
    return list
}
fun cmp(value : Int) = value % 2 == 0

val list = (1..10).toList()
val evenList = list.customFilter(::cmp)
println(evenList)
```
- Nếu không có `::` thì sẽ lỗi
##### Lambda Expression
```Kotlin
val list = (1..10).toList()
val evenList = list.customFilter({it % 2 == 0})
println(evenList)
```
- Nếu tham số cuối cùng của hàm là một function type, chúng ta có thể tách riêng argument đó để ra bên ngoài
```Kotlin
val list = (1..10).toList()
val evenList = list.customFilter(param){it % 2 == 0}
println(evenList)
```

##### Function Anonymous
- Định nghĩa luôn lúc truyền vào
```Kotlin
val list = (1..10).toList()
val evenList = list.customFilter(fun(value : Int) : Boolean{
    return value % 2 == 0
})
println(evenList)
```
- Gán cho biến rồi truyền vào
```Kotlin
val funCmp = {it : Int -> it % 2 == 0}
val list = (1..10).toList()
val evenList = list.customFilter(funCmp)
println(evenList)
```
### Lambda Funtion
- Syntax
```Kotlin
val lambdaName: (parameters) -> returnType = { arguments -> functionBody }

val sum: (Int, Int) -> Int = { x: Int, y: Int -> x + y }
```
- `Lambda expression` luôn được bao trong cặp dấu ngoặc nhọn.
- `lambdaName` là tên của lambda function (có thể bỏ qua nếu không cần thiết).
- `parameters` là danh sách các tham số đầu vào của lambda function.
- `arguments` là tên các tham số được truyền vào lambda function.
- `functionBody` là phần thân của lambda function.
- `returnType` là kiểu dữ liệu trả về của lambda function.
- Nếu kiểu trả về của một lambda function không phải là `Unit`, expression cuối trong thân lambda (dòng cuối cùng trong body) được lấy làm giá trị trả về
### Passing trailing lambdas
- Nếu tham số cuối cùng của hàm là một function type, chúng ta có thể tách riêng argument đó để ra bên ngoài
```Kotlin
fun <T>List<T>.customFilter(param : T, filterFunction : (T) -> (Boolean)) : List<T>{
    var list = mutableListOf<T>()
    for(it in this){
        if(filterFunction(it)){
            list.add(it)
        }
    }
    return list
}
val list = (1..10).toList()
val evenList = list.customFilter(param){it % 2 == 0}
// val evenList = list.customFilter(param, {it % 2 == 0})
println(evenList)
```
- Nếu truyền vào 1 agrument thì bỏ ()
```Kotlin
val evenList = list.customFilter{it % 2 == 0}
```
### It
- Là tên mặc định cho một tham số duy nhất của Lambda (dùng để tham chiếu đến tham số)
- Tham số không cần phải khai báo và “→”có thể được lược bỏ. 
```Kotlin
fun findSquare() : (Int) -> Unit = {
    println(it * it)
}
fun findSquareFull() : (Int) -> Unit = { number ->
		println(number * number)
}
val func = findSquare()

func(3)  //9
```
### Anonymous function
- Lambda expression syntax ở trên đều thiếu khả năng xác định kiểu trả về của function. Trong hầu hết các trường hợp, điều này là không cần thiết bởi vì trình biên dịch có thể tự động suy ra kiểu trả về. Tuy nhiên, nếu cần `xác định kiểu trả về một cách cụ thể` ta có thể dùng `anonymous function`.

```kotlin
fun(x: Int, y: Int): Int = x + y
```

- `1 anonymous function` nhìn khá giống `1 regular function`, trừ việc tên của nó bị lược bỏ và thân của nó có thể là biểu thức hoặc block.

```kotlin
fun(x: Int, y: Int): Int {
    return x + y
}
```

- Có thể bỏ qua kiểu của tham số nếu có thể được suy ra từ context:

```kotlin
ints.filter(fun(item) = item > 0)
```

- Kiểu trả về của `anonymous function` được suy ra giống các funtion bình thường: kiểu trả về được chỉ định tự động khi `anonymous function` khai báo với `expression body`, nhưng nó phải được chỉ định rõ ràng (hoặc giả định là unit) cho các anonymous có `block body`.

### Closures
- Trong Kotlin, `closures` là khả năng cho phép `lambda expression (anonymous function)` truy cập và thay đổi các biến từ phạm vi bên ngoài của nó.
- Khi một lambda expression được tạo ra, nó có thể tham chiếu và sử dụng các biến được khai báo trong phạm vi bên ngoài của nó. Điều này cho phép lambda expression tạo ra một "closure" xung quanh các biến mà nó tham chiếu.

```kotlin
fun main() {
    val x = 5

    val closure: () -> Unit = {
        println("x = $x")
    }

    closure() // In ra giá trị của biến x từ phạm vi bên ngoài của lambda
		// x = 5

    // Thay đổi giá trị của biến x
    x = 10

    closure() // In ra giá trị mới của biến x
		// x = 10
}
```
### CallBack
- Callback là sử dụng một function type làm tham số (đứa gọi nó gọi là Higher Order Function)
- Thứ tự thực hiện: Higher Order Function → CallBack Function → Higher Order Funtion

```kotlin
fun calc(a : Int, b : Int, f : (Int, Int) -> Int) : Int {
    return f(a,b)
}
fun main() {
    val sum : (Int, Int) -> Int = { a, b -> 
        a + b
    }
    val mul : (Int, Int) -> Int = { a, b -> 
        a * b
    }
    println(calc(1, 1, sum)) // 2
    print(calc(2, 2, mul)) // 4
}
```

- Trong Java, không có function type nên dùng interface để thay thế
- Nếu Interface chỉ có duy nhất một abstract method thì nó được gọi là funtional interface

```kotlin
@FunctionalInterface
interface IOperator {
    void onResult(double result);
}

class Calculator {
    public void add(double a, double b, IOperator callback) {
        double result = a + b;
        callback.onResult(result);
    }

    public void subtract(double a, double b, IOperator callback) {
        double result = a - b;
        callback.onResult(result);
    }

    
}
public class Main {
    public static void main(String[] args) {

        Calculator calculator = new Calculator();

        IOperator callback = new IOperator() {
            @Override
            public void onResult(double result) {
                System.out.println("Result: " + result);
            }
        };
				// Vi la Functional Interface nen co the rut gon thanh lambda
        IOperator callbackWithLambda = (result) -> {
            System.out.println("Result: " + result);
        } ;
	
        calculator.add(5, 3, callback);
        calculator.subtract(5, 3, callback);

        calculator.add(5, 3, callbackWithLambda);
        calculator.subtract(5, 3, callbackWithLambda);
        
    }
}
```
- Lợi ích của callback:
    - Tái sử dụng code, hạn chế lỗi: Có thể tái sử dụng lại code chỉ cần thay đổi lại hàm callback.
    - Tăng cường tính bao đóng của OOP cho hàm callback.
- Điểm trừ của callback:
    - Chương trình chính phải dừng lại đợi chương trình con. Nếu hàm callback mất thời gian sẽ ảnh hưởng tới hiệu suất chương trình.
    - Trở nên rất loạn khi sử dụng đa luồng bất đồng bộ.
    - Khó debug, khó kiểm soát lỗi.