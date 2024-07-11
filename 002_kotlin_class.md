- [Class \&\& Special Class](#class--special-class)
  - [Class](#class)
    - [Cách khai báo](#cách-khai-báo)
    - [Constructor](#constructor)
      - [Primary constructor](#primary-constructor)
      - [Secondary constructor](#secondary-constructor)
    - [Visibility Modifiers - Access Modifier](#visibility-modifiers---access-modifier)
      - [Package](#package)
      - [Class](#class-1)
    - [Properties](#properties)
      - [Backing field](#backing-field)
      - [Getter và Setter](#getter-và-setter)
    - [Class member](#class-member)
      - [Functions](#functions)
        - [Single-expression functions](#single-expression-functions)
        - [Variadic Arguments Function (Số lượng đối số không giới hạn)](#variadic-arguments-function-số-lượng-đối-số-không-giới-hạn)
        - [Infix notation](#infix-notation)
      - [Generic Function](#generic-function)
      - [Lambda expressions](#lambda-expressions)
  - [Special Class](#special-class)
    - [Data Class](#data-class)
      - [Khai báo thuộc tính](#khai-báo-thuộc-tính)
      - [Function copy](#function-copy)
    - [Enum Class](#enum-class)
      - [Cách khai báo](#cách-khai-báo-1)
      - [Anonymous Classes](#anonymous-classes)
    - [Sealed Class](#sealed-class)
      - [So sánh Enum và Sealed class](#so-sánh-enum-và-sealed-class)

# Class && Special Class
## Class
### Cách khai báo
- Các lớp trong Kotlin sử dụng từ khóa `class` để khai báo
```Kotlin
class Person{
}
```
- Khai báo đầy đủ  của 1 class sẽ bao gồm:
    - `class name`
    - `class header` (Nơi định nghĩa tên lớp, các tham số khởi tạo, lớp cha và các kế thừa, iterfaces) class Persion (val name: String, val age: Int)
    - `class body` trong ngoặc nhọn
- `class header` và `class body` là **optional**, nếu class không có `body` thì không cần {}

### Constructor
- `Class` trong Kotlin có 1 `primary constructor` (Hàm khởi tạo chính) và có 1 hoặc nhiều `secondary constructors`
#### Primary constructor
- `primary constructor` được khai báo trong `class header`

```Kotlin
class Person constructor(name : String, age : Int){
}
```

- Nếu `primary constructor` không có bất kì `annotation` (chú thích) hay `visibility modifier` (Từ khóa xác định phạm vi) thì từ khóa `constructor` có thể được bỏ qua (mặc định không khai báo gì thì sẽ là public constructor)

```Kotlin
class Person (name : String, age : Int){
}
```

- Còn nếu có visibility modifier hoặc annotation thì từ khóa constructor sẽ cần phải có
```Kotlin
class Person private constructor(name : String, age : Int){
}
```
- **Note:** Các `param` (tham số khởi tạo) trong `primary constructor` chỉ có thể được sử dụng trong các khối `init`, chứ không thể sử dụng trong các `funtion` của `class`. Để sử dụng được thì ta phải thêm các từ khoá `val`, `var`.

```Kotlin
class Person ( val name : String, val age : Int){
    fun printInfo(){
        println("Name: $name")
        println("Age: $age")
    }
}

```
- Các phần code trong khối `init` sẽ được chạy ngay sau `primary constructor` và theo trình tự từ trên xuống dưới

#### Secondary constructor
- Để khai `secondary constructor`, ta thêm từ khóa `constructor`
```Kotlin
class Person{
    constructor(){
        println("Object created")
    }
    constructor(name : String, age : Int){
        println("Name: $name")
        println("Age: $age")
    }

}
```
- Nếu như class có `primary constructor` thì mỗi `secondary constructor` phải khởi tạo giá trị cho `primary constructor` bằng cách gọi `primary constructor` một cách gián tiếp hoặc trực tiếp bằng từ khóa `this` [(Giải thích)](https://stackoverflow.com/questions/51011574/why-each-secondary-constructor-needs-to-delegate-to-the-primary-constructor-in-k#:~:text=This%20is%20because%20init%20blocks,being%20able%20to%20have%20properties).

```Kotlin
class Person (){
     constructor(name: String, age: Int) : this() {
         println("Name: $name, Age: $age")
     }
		// Truc tiep
    constructor(name: String, age: Int, email: String) : this(name, age) {
        println("Name: $name, Age: $age, Email: $email")
    }
		// Gian tiep
}
```
- **Note:** Không thể khởi tạo các thuộc tính trong `secondary constructor`

```Kotlin
class Person (){
    constructor(val name : String) : this(){ // => Lỗi ở val name
        println("Name: $name")
    }
}
```
### Visibility Modifiers - Access Modifier
- `Class, object, interfaces, constructor, functions, properties, setters` đều cần `visibility modifiers`. `Getter` có `visibility modifiers` giống với thuộc tính của chúng.
- Có 4 `visibility modifiers`: `private`, `protected`, `internal`  giống `default` trong java và `public`. Mặc định sẽ là `public`

1 module là 1 tập các file Kotlin được biên dịch cùng với nhau
protected không được phép cho các khai báo top-level.
|Visibility Modifiers|Trong class|Trong package|Lớp con|Ngoài package|
|---|---|---|---|---|
|private|Y|N|N|N|
|protected|Y|N|Y|N|
|internal|Y|Y|N|N|
|public|Y|Y|Y|Y|

#### Package
- `Function`, `property`, `class`, `object` và `interface` đều có thể khai báo ở mức "top-level" (được hiểu là nó không ở bên trong bất cứ thành phần nào)
- Nếu không xác định rõ ràng `visibility modifier` thì mặc định sẽ là `public`. Điều đó có nghĩa là bạn có thể sử dụng nó ở bất kì đâu.
- Nếu khai báo là `private`, nó chỉ được sử dụng ở trong file mà nó khai báo.
- Nếu khai báo là `internal`, nó chỉ được sử dụng ở các nơi cùng `module` (các tập Kotlin được biên dịch cùng nhau)
- `protected` không được sử dụng khi khai báo ở mức "top-level".

```Kotlin
// file name: example.kt
package foo

private fun foo() { ... } // visible inside example.kt

public var bar: Int = 5 // property is visible everywhere
    private set         // setter is visible only in example.kt

internal val baz = 6    // visible inside the same module
```

#### Class
- Đối với các thành phần khai báo ở trong `Class`:
    - `private` chỉ được sử dụng trong class đó (không thể truy cập qua instance của class đó)
    - `protected` giống với `private` + có thể sử dụng ở trong các subclass (các class kế thừa nó).
    - `internal` khả năng truy cập rộng hơn 2 loại trên và được truy cập bởi các instance của class khai báo nó (các instance và nơi khai báo class thuộc cùng module)
    - `public` rộng nhất và được sử dụng ở bất kì đâu thông qua instance khai báo nó

### Properties
#### Backing field

```Kotlin
var age: Int = 0
        get() = age
        set(value) {
            this.age = value
        }
```
- Thực chất, đoạn code trên khi chạy sẽ bị lỗi StackOverFlow vì chúng ta đang gọi đệ quy liên tục hàm setter với dòng this.age = value (trong Kotlin, setter sẽ được gọi mỗi khi ta thực hiện gán giá trị).
- Vậy nên Kotlin đưa ra cho chúng ta 1 từ khóa là `field` . Từ khóa `field` chỉ được sử dụng để lưu giá trị của thuộc tính và không thể khai báo nó. 

```Kotlin
var age: Int = 0
        get() = field
        set(value) {
            field = value
        }
```
#### Getter và Setter
- Cách truyền thống
```Kotlin
// Person.kt
class Person() {
    private var name: String = ""

    fun getName(): String {
        return this.name
    }

    fun setName(name:String) {
        this.name = name
    }
}

// Main.kt

fun main(args: Array<String>) {
    val a = Person()
    a.setName("Tam")
    print(a.getName()) // Tam
}

```
- Sử dụng Field và Property

```Kotlin
// Person.kt

class Person() {
    private var name: String = "" // field

    var Name: String // property
        get() {
            return this.name
        }
        set(value) {
            this.name = value
        }

    fun getName1(): String {
        return this.name
    }

    fun setName1(name:String) {
        this.name = name
    }
}

// Main.kt

fun main(args: Array<String>) {
    val a = Person()
    a.Name = "Chinh"
    println(a.Name) // Chinh
    print(a.getName1()) // Chinh
}
```
### Class member
#### Functions
- Khai báo bằng từ khóa `fun`: `fun <name>(parameters): <Unit> {}`
##### Single-expression functions
- Khi thân hàm là 1 biểu thức duy nhất, các dấu ngoặc nhọn có thể bỏ qua và phần thân được chỉ định sau `=`: `fun Sum(a :Int, b:Int) = a + b`
##### Variadic Arguments Function (Số lượng đối số không giới hạn)
```Kotlin
fun like (vararg fruits: String){
    for (fruit in fruits){
        println("I like $fruit")
    }
}

like(like(fruits = *arrayOf("apple", "banana", "orange")))
```
##### Infix notation
- Hàm được đánh dấu bằng từ khóa `infix` và có thể được gọi bằng ký hiệu `infix` (bỏ dấu . và ngoặc đơn khi gọi)
```Kotlin
infix fun Int.add(a: Int): Int {
    return this + a
}

1 add 2
```

```Kotlin
class MyStringCollection {
    infix fun add(s: String) { /*...*/ }

    fun build() {
        this add "abc"   // Correct
        add("abc")       // Correct
        //add "abc"        // Incorrect: the receiver must be specified
    }
}
```
#### Generic Function
- Hàm có thể có các tham số chung, được chỉ định bằng dấu ngoặc nhọn trước tên hàm: `fun <T> singletonList(item: T): List<T> { /*...*/ }`

#### Lambda expressions
- là hàm hằng (Là các hàm không được khai báo nhưng được truyền ngay lập tức giá trị dưới dạng biểu thức)
- `val sum: (Int, Int) -> Int = { x: Int, y: Int -> x + y }`

## Special Class
### Data Class
- `Data class` trong Kotlin là một lớp được thiết kế để lưu trữ và đại diện cho dữ liệu. Khi ta khai báo một lớp là `Data class`, `Kotlin` sẽ tự động tạo ra một số phương thức và tính năng mặc định để giúp xử lý dữ liệu một cách đơn giản và tiện lợi hơn.

```Kotlin
// Class User 1 cách bình thường
public class User {
    private String name;
    private int age;

    public User(String name, int age) {
      this.name = name;
      this.age = age;
    }

    public String getName() {
      return name;
    }

    public void setName(String name) {
      this.name = name;
    }

    public int getAge() {
      return age;
    }

    public void setAge(int age) {
      this.age = age;
    }
  }

// Class User khi sử dụng data class
data class User(val name : String, val age : Int)
```
- Khi ta sử dụng từ khóa `data` để khai báo một lớp, `Kotlin` sẽ tự động tạo ra các phương thức
    - **`getter`** **`setter`**
    - **`equals()`**: dùng để so sánh hai đối tượng của lớp với nhau.
    - **`hashCode()`**: dùng để tạo ra mã băm cho đối tượng của lớp.
    - **`toString()`**: dùng để chuyển đổi đối tượng của lớp thành một chuỗi ký tự.
    - **`componentN()`**: dùng để truy cập các thuộc tính của đối tượng của lớp bằng cách sử dụng toán tử **`[]`**
    - **`copy()`**: dùng để tạo ra một bản sao của đối tượng của lớp với một số thuộc tính được cập nhật.

- Tuy nhiên về mặt cú pháp, khi sử dụng `Data Class` cũng yêu cầu một số quy tắc nhất định để đảm bảo những đoạn `code` sinh ra có tính nhất quán và thực hiện đúng ý nghĩa:
    - Hàm `constructor` chính phải có ít nhất một tham số truyền vào.
    - Tất cả các tham số truyền vào trong hàm `constructor` chính phải được khai báo là `val` hoặc `var`.
    - Những `Data Class` không thể là `abstract`, `open`, `sealed` hay là `inner`.

#### Khai báo thuộc tính
- Với những thuộc tính khi chúng ta khai báo ở trong hàm `constructor` của `Data Class`, trình biên dịch sẽ tự động sinh ra cho chúng ta những phương thức tự động. Còn những thuộc tính khi khai báo bên trong `Class` sẽ không được sinh tự động như vậy.
- Vì trình biên dịch chỉ sử dụng các thuộc tính được xác định bên trong hàm `constructor` chính cho các hàm được tạo tự động. Với những thuộc tính không cần có nhu cầu sinh `code` tự động, hãy viết ở trong thân `Class`

```Kotlin
data class User(var name : String, var age : Int){
    var email : String = ""

    constructor(name : String, age : Int, email : String) : this(name, age){
        this.email = email
    }
}

fun main(){
    var user1 = User("Zin", 20, "Zin")
    var user2 = User("Zin", 20, "Zin1")
    println(user1.equals(user2))
    //True ??? So sánh 2 thuộc tính của `primary constructor` ??
}
```

#### Function copy
- Trong trường hợp bạn cần copy một đối tượng nhưng lại muốn thay đổi một số property và giữ nguyên phần còn lại. Hàm `copy()` được sinh ra là để làm điều đó.

```Kotlin
data class User(var name : String, var age : Int){
}

fun main(){
    var user1 = User("Zin", 20)
    var user2 = user1.copy(age = 21)
    println(user2)
    //User(name=Zin, age=21)
}
```

### Enum Class
#### Cách khai báo
```Kotlin
enum class Direction {
    NORTH, SOUTH, WEST, EAST
}
```
- Chúng ta có thể thêm các thuộc tính vào trong các `Enum Constant`

```Kotlin
enum class Direction (val dir: String){
    NORTH("Bac"),
    SOUTH("Nam"),
    EAST("Dong"),
    WEST("Tay")
}

fun main(){
    var dir = Direction.valueOf("NORTH")
    println(dir.name) //NORTH
    println(dir.dir)    // Bac
    println(dir.ordinal) // 0 : index cua North
}
```

#### Anonymous Classes
- Mỗi enum constant cũng có thể khởi tạo `anonymous class` của riêng nó

```Kotlin
enum class ProtocolState {
    WAITING {
        override fun signal() = TALKING
    },

    TALKING {
        override fun signal() = WAITING
    };

    abstract fun signal(): ProtocolState
}

fun main(){
    var protocolState : ProtocolState = ProtocolState.WAITING
    protocolState = protocolState.signal()
    println(protocolState)
    //TALKING
}
```
- `constant WAITING` định nghĩa một `anonymous class` của nó và `override` lại `function signal()`. Nếu chú ý bạn có thể thấy dấu ;, dấu này có tác dụng phân cách giữa các enum constant và các định nghĩa thành phần member(như variable, function) của enum class.

### Sealed Class
- Trong Kotlin, sealed class là một loại lớp đặc biệt được sử dụng để định nghĩa một tập hợp các lớp con có thể mở rộng từ nó. Sealed class giới hạn việc kế thừa, chỉ cho phép các lớp con được định nghĩa trong cùng một tệp hoặc tệp tin được nhìn thấy.

```Kotlin
sealed class SearchResult{
    data class Success(val data: List<String>): SearchResult()
    data class Error(val exception: Exception): SearchResult()
}

```
- Nhìn qua thì `sealed class` same same với `enum class`: tập hợp các giá trị cho kiểu enum cũng được giới hạn, nhưng mỗi enum chỉ được có 1 instance, trong khi một class con của sealed class có thể có nhiều instance.
  - **Sealed Class** cung cấp cho ta 1 hệ thống phân cấp các Class.
  - **Sealed Class** chứa những subclass, object, function hay thậm trí là các **Sealed Class** khác.
  - Trong một số trường hợp, **Sealed Class** cũng khá tương tự **Enum Class.**
  - Các Class muốn kế thừa **Sealed Class** cần được khai báo trong cùng file với **Sealed Class.**
  - Mặc định, các constructors của `sealed class`có modifier là `private`
  - Đối với **Enum Class** thì tất cả các enum không đổi bên trong cần phải có Constructor đồng nhất, còn đối với **Sealed Class** thì không. **Constructor** của các subclass bên trong **Sealed Class** được tự do với số lượng và kiểu của parameter chứ không bị ràng buộc.

```Kotlin
// A sealed class with a string property
sealed class Fruit(val x : String)
{
    // Two subclasses of sealed class defined within
    class Apple : Fruit("Apple")
    class Mango : Fruit("Mango")
}
 
// A subclass defined outside the sealed class
class Pomegranate: Fruit("Pomegranate")
 
// A function to take in an object of type Fruit
// And to display an appropriate message depending on the type of Fruit
fun display(fruit: Fruit)
{
    when(fruit)
    {
        is Fruit.Apple -> println("${fruit.x} is good for iron")
        is Fruit.Mango -> println("${fruit.x} is delicious")
        is Pomegranate -> println("${fruit.x} is good for vitamin d")
    }
}
fun main()
{
    // Objects of different subclasses created
    val obj = Fruit.Apple()
    val obj1 = Fruit.Mango()
    val obj2 = Pomegranate()
 
    // Function called with different objects
    display(obj)
    display(obj1)
    display(obj2)
}

//Apple is good for iron
//Mango is delicious
//Pomegranate is good for vitamin d
```

#### So sánh Enum và Sealed class
- `Sealed class` là phần mở rộng của `enum class` do đó nó sẽ tốt hơn `enum class.`
- **Giống nhau:** `variable` có kiểu nằm trong một tập giới hạn.
- **Khác nhau:**
    - Các `enum constant` chỉ có 1 kiểu khai báo (giống với `enum class`) và tồn tại như 1 `instance` của `enum class`. Các `enum constant` chỉ có thể nằm trong `body` của `enum class.`
    - `Sealed class` có thể có `subclass`. Các `subclass` của `sealed class` có thể khai báo theo nhiều cách khác nhau. Các `subclass` phải được khai báo cùng `file` với `sealed class`.