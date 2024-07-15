- [OOP Trong Kotlin](#oop-trong-kotlin)
  - [Inheritance](#inheritance)
    - [Ghi đè phương thức và thuộc tính](#ghi-đè-phương-thức-và-thuộc-tính)
  - [Abstraction](#abstraction)
    - [Interface](#interface)
    - [Abstact Class](#abstact-class)

# OOP Trong Kotlin
## Inheritance
- Tất cả các `class` trong `Kotlin` đều có 1 superclass chung đó là `Any`
- `Any` có 3 phương thức là `equals()`, `hashCode()`, `toString()` cung cấp cho tất cả các class sử dụng
- Mặc định các `class` trong `Kotlin` là `final` nên chúng không thể kế thừa. Để một lớp có thể kế thừa chúng ta cần khai báo thêm từ khóa `open` vào trước `class`
```Kotlin
open class Animal{

}
class Person : Animal{

}
```
### Ghi đè phương thức và thuộc tính
- Trong `Kotlin`, cũng giống như với các `class`, để ghi đè một phương thức hay thuộc tính của lớp cha thì phương thức/thuộc tính đó cũng phải có được khai báo với từ khóa `open`

>Chúng ta có thể ghi đè một thuộc tính `val` bằng thuộc tính `var` nhưng không thể theo chiều ngược lại.

```Kotlin
open class Shape {
    open fun draw() { /*...*/ }
    fun fill() { /*...*/ }
}

class Circle() : Shape() {
    override fun draw() { /*...*/ }
}
```

- Trong trường hợp một class có nhiều kế thừa có chung một thành phần, compiler sẽ bắt chúng ta phải `override` lại thành phần đó. Để gọi đến các thành phần thuộc các `class`, `interface` cha khác nhau thì ta sử dụng super<Base>. Ví dụ:

```Kotlin
open class Rectangle {
    open fun draw() { /* ... */ }
}

interface Polygon {
    fun draw() { /* ... */ } // interface members are 'open' by default
}

class Square() : Rectangle(), Polygon {
    // The compiler requires draw() to be overridden:
    override fun draw() {
        super<Rectangle>.draw() // call to Rectangle.draw()
        super<Polygon>.draw() // call to Polygon.draw()
    }
}
```
- Nếu ta đang sử dụng 1 `inner class` và muốn truy cập vào các thành phần thuộc lớp cha của lớp bao ngoài nó, thì ta sử dụng  `super@Outer`

```Kotlin
class FilledRectangle: Rectangle() {
    override fun draw() {
        val filler = Filler()
        filler.drawAndFill()
    }

    inner class Filler {
        fun fill() { println("Filling") }
        fun drawAndFill() {
            super@FilledRectangle.draw() // Calls Rectangle's implementation of draw()
            fill()
            println("Drawn a filled rectangle with color ${super@FilledRectangle.borderColor}") // Uses Rectangle's implementation of borderColor's get()
        }
    }
}
```

## Abstraction
### Interface
- `Interface` có thể chứa các `method ảo` và các `method` đã được định nghĩa, tuy nhiên `property `trong `interface` là `public abstract property` (tức là không có giá trị)

```Kotlin
interface IPerson{
    val name : String
    val age : Int = 10 // Loi
}
```
- Tất cả các class `implement interface` thì cần phải `override` tất cả các `method/property ảo` trong `interface` mà class đó `implement`

```Kotlin
class Child(override val name: String) : IPerson{
    override fun print2() {
        TODO("Not yet implemented")
    }
}
```
### Abstact Class
- **Abstract class** (lớp trừu tượng) là một lớp mà không thể tạo đối tượng trực tiếp.
- **Abstract class** được sử dụng để các lớp con kế thừa.
- **Abstract class** không thể tự tạo **instance** cho chính nó.
- **Abstract class** có thể bao gồm **abstract method** và **non-abstract method**.
- Chỉ có thể kế thừa từ duy nhất 1 abstract class.

```Kotlin
abstract class Person(){
    fun print(){
        println("Init Person")
    }
    // Non abstract member function
    abstract fun print2()
    // Abstract member function

    protected val name = "Kotlin"
    // Non abstract member property
    protected abstract val age : Int
    // Abstract member property

}
```
- Khi class con kế thừa một abstract class thì cần phải `override` lại các `Property/Method`

```Kotlin
class Student(override val age: Int) : Person(){
    override fun print2() {
        println("Init Student")
    }
}
```
- Sự khác nhau giữa Abstract Class và Interface

|Abstract Class|Interface|
|---|---|
| 1 Class chỉ có thể extend 1 Abstract Class.|1 Class có thể implement nhiều Interface.|
| Là 1 class.| Không phải 1 Class.|
| Có thể khởi tạo cho các non abstract property|Không thể khởi tạo giá trị cho các property|
|Có thể implement interface.|Không thể kế thừa hay implement|
|Là một template class cho một loạt các class có chức năng tương tự nhau|Là một tính năng có thể xuất hiện ở một số các class|