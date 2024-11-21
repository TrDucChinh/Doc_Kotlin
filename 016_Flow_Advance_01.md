- [Flow](#flow)
  - [Flow là gì? Cách tạo ra flow](#flow-là-gì-cách-tạo-ra-flow)
    - [Tính năng nổi bật của Flow](#tính-năng-nổi-bật-của-flow)
    - [Khác nhau giữa Flow, Sequences, Collections](#khác-nhau-giữa-flow-sequences-collections)
    - [Thành phần chính trong luồng dữ liệu](#thành-phần-chính-trong-luồng-dữ-liệu)
    - [Cách tạo ra Flow](#cách-tạo-ra-flow)
    - [Sử dụng flow trong thực tế](#sử-dụng-flow-trong-thực-tế)
  - [Toán tử trong Flow](#toán-tử-trong-flow)
  - [Cold flow vs Hot flow. Flow vs StateFlow vs SharedFlow.](#cold-flow-vs-hot-flow-flow-vs-stateflow-vs-sharedflow)
    - [Cold flow vs Hot flow](#cold-flow-vs-hot-flow)
      - [Cold flow](#cold-flow)
      - [Hot Flow](#hot-flow)
      - [So sánh Cold Flow và Hot Flow](#so-sánh-cold-flow-và-hot-flow)
      - [3. Khi nào dùng Cold Flow hoặc Hot Flow?](#3-khi-nào-dùng-cold-flow-hoặc-hot-flow)
    - [Flow vs StateFlow vs SharedFlow.](#flow-vs-stateflow-vs-sharedflow)
      - [Flow](#flow-1)
      - [StateFlow](#stateflow)
      - [SharedFlow](#sharedflow)
      - [So sánh chi tiết: Flow, StateFlow, SharedFlow](#so-sánh-chi-tiết-flow-stateflow-sharedflow)
      - [Khi nào dùng loại nào?](#khi-nào-dùng-loại-nào)
  - [So sánh với LiveData.](#so-sánh-với-livedata)
    - [Tổng quan](#tổng-quan)
    - [Chi tiết so sánh](#chi-tiết-so-sánh)
      - [Nhận thức vòng đời](#nhận-thức-vòng-đời)
      - [Hỗ trợ luồng dữ liệu](#hỗ-trợ-luồng-dữ-liệu)
      - [Sử dụng trong ViewModel và UI](#sử-dụng-trong-viewmodel-và-ui)
      - [Hỗ trợ API](#hỗ-trợ-api)
    - [Khi nào nên dùng Flow và LiveData?](#khi-nào-nên-dùng-flow-và-livedata)

# Flow
## Flow là gì? Cách tạo ra flow
- `Flow` là một API của thư viện Kotlin Coroutines dùng để xử lý luồng dữ liệu bất đồng bộ (asynchronous stream)
- `Flow` cho phép phát ra `(emit)` một chuỗi các giá trị theo thời gian, thay vì chỉ trả về một giá trị duy nhất như `suspend functions`.
- `Flow` hoạt động theo mô hình `cold stream`, nghĩa là dữ liệu chỉ bắt đầu được phát khi có người thu thập `(collect)` nó. Điều này giúp tiết kiệm tài nguyên và phù hợp với các tình huống luồng dữ liệu có thể thay đổi liên tục.
- `Flow` về cơ bản khá giống `Sequences` trong Kotlin nhưng khác ở chỗ `Sequences` xử lý đồng bộ còn `Flow` xử lý bất đồng bộ

### Tính năng nổi bật của Flow
- **Cold Stream**: Dữ liệu chỉ được phát khi có người dùng gọi `collect`.
- **Asynchronous**: Hoạt động tốt với các tác vụ bất đồng bộ.
- **Composable**: Cung cấp nhiều toán tử (operators) mạnh mẽ để xử lý dữ liệu.
- **Backpressure handling**: Giảm áp lực khi lượng dữ liệu phát ra quá nhiều so với tốc độ xử lý.

### Khác nhau giữa Flow, Sequences, Collections
- **Collections (List)**
```Kotlin
suspend fun foo(): List<Int> {
    val list = mutableListOf<Int>()
    for (i in 1..3) {
        delay(1000)
        list.add(i)
    }

    return list
}

fun main() = runBlocking {
    val time = measureTimeMillis {
        foo().forEach { value -> println(value) }
    }
    println(time)
}
/*
1
2
3
4
5
5053
*/
```
- **Sequences**
```Kotlin
fun foo1(): Sequence<Int> = sequence { // sequence builder
    for (i in 1..5) {
        Thread.sleep(1000)
        yield(i)
    }
}


fun main() = runBlocking {
    val time = measureTimeMillis {
        foo1().forEach { value -> println(value) }
    }
    println(time)
}
/*
1
2
3
4
5
5054
*/
```

>- Output được in ra là giống nhau và thời gian thực hiện cũng bằng nhau.
>- Nhưng khác ở chỗ `List` đợi `add` xong cả 3 phần tử rồi mới in ra (Do dùng suspen function), còn `Sequence` thì cứ mỗi giây thì có phần tử được `yield` và phần tử đó lập tức được in ra ngay mà không phải đợi `yield` xong cả 3 phần tử.

- **Flow**
```Kotlin
fun foo(): Flow<Int> = flow{
    for(i in 1..5){
        delay(1000)
        emit(i)
    }
}

fun main() = runBlocking{
    val time = measureTimeMillis {
        foo().collect { value -> println(value) }
    }
    println(time)
}
/*
1
2
3
4
5
5070
*/
```
>- Về cơ bản thì `Flow` khá giống `Sequences` , thay vì sử dụng `yield` thì `flow` dùng `emit` và nhận các giá trị thông qua `collect` .
>- Nhưng `Flow` xử lý bất đồng bộ còn `Sequences` xử lý đồng bộ.

- **Sequences**: Xử lý đồng bộ
```Kotlin
fun foo(): Sequence<Int> = sequence {
    for(i in 1..3){
        Thread.sleep(1000)
        yield(i)
    }
}

fun main() = runBlocking{
    launch {
        println(Thread.currentThread().name)
        for(i in 1..3){
            delay(1000)
            println("I'm blocked $i")
        }
    }

    val time = measureTimeMillis {
        foo().forEach {value -> println(value)}
    }
    println("$time s")
}

/*
1
2
3
3030 s
main
I'm blocked 1
I'm blocked 2
I'm blocked 3
*/
```
> **Sequences** đã block `MainThread` để chạy in ra hết các giá trị của nó, sau khi chạy xong nó mới UnBlock `MainThread` để in ra `I'm blocked..` 

- **Flow:** Xử lý bất đồng bộ
```Kotlin
fun foo(): Flow<Int> = flow {
    // flow builder
    for (i in 1..3) {
        delay(1000)
        emit(i) // emit next value
    }
}

fun main() = runBlocking {
    // Launch a concurrent coroutine to check if the main thread is blocked
    launch {
        println(Thread.currentThread().name)
        for (k in 1..3) {
            delay(900)
            println("I'm not blocked $k")
        }
    }
    // Collect the flow
    val time = measureTimeMillis {
        foo().collect { value -> println(value) }
    }
    println("$time s")
}

/*
main
I'm not blocked 1
1
I'm not blocked 2
2
I'm not blocked 3
3
4
5
5052 s
*/
```
>- **Sequence** xử lý đồng bộ. Nó sử dụng `Iterator` và `block main thead` trong khi chờ đợi item tiếp theo được `yield`.
>- **Flow** xử lý bất đồng bộ. Nó sử dụng một `suspend function collect` để không block main thread trong khi chờ đợi item tiếp theo được `emit`.

- Khối `flow { }` là một `builder function` giúp ta tạo ra 1 đối tượng `Flow`
- Code bên trong `flow{}` có thể là suspend function => Có thể gọi các `suspen function` trong `flow{}` => Hàm `foo()` không nhất thiết phải là `suspen function`
- Hàm `emit` là 1 `suspen function` để `emit` các giá trị `flow`
- Hàm `collection` là 1 `suspend function` dùng để get các giá trị được `emit`

### Thành phần chính trong luồng dữ liệu

![](https://developer.android.com/static/images/kotlin/flow/flow-entities.png)
- **Producer** **(người cung cấp):** cung cấp data được thêm vào luồng dữ liệu. Flow có thể cung cấp data bất đồng bộ. Hiểu đơn giản là khi bạn `emit()` giá trị vào Flow.
- **Intermediary (người trung gian):** có thể sửa đổi giá trị được emit ở luồng stream. Ở đây có thể hiểu là những toán tử trung gian của `Flow` như `.map()`, `.catch()`, ....
- **Consumer (người sử dụng):** sử dụng value từ luồng stream. Hiểu đơn giản là khi bạn `collect()` Flow.

### Cách tạo ra Flow
- **Dùng flow builder**
```Kotlin
import kotlinx.coroutines.flow.flow
import kotlinx.coroutines.runBlocking

val numberFlow = flow {
    for (i in 1..5) {
        emit(i) // Phát giá trị
        kotlinx.coroutines.delay(1000) // Giả lập tác vụ bất đồng bộ
    }
}

fun main() = runBlocking {
    numberFlow.collect { value -> 
        println(value)
    }
}
```
- **Dùng các hàm thư viện**
  - `flowOf():` Tạo `Flow` từ một danh sách các giá trị.
```Kotlin
import kotlinx.coroutines.flow.flowOf
import kotlinx.coroutines.runBlocking

val flow = flowOf(1, 2, 3, 4, 5)

fun main() = runBlocking {
    flow.collect { value ->
        println(value)
    }
}
```
  - `asFlow():` Chuyển đổi từ một `collection` thành `Flow.`
```Kotlin
import kotlinx.coroutines.flow.asFlow
import kotlinx.coroutines.runBlocking

val list = listOf(1, 2, 3, 4, 5)
val flow = list.asFlow()

fun main() = runBlocking {
    flow.collect { value ->
        println(value)
    }
}
```

- **Sử dụng `channelFlow` cho dữ liệu đồng thời**
  - Nếu cần phát dữ liệu từ nhiều `coroutine` khác nhau thì dùng `channelFlow`.
```Kotlin
fun flow1(): Flow<Int> = flow {
    for (i in 1..5) {
        delay(1000)
        emit(i)
    }
}

fun flow2(): Flow<Int> = flow {
    for (i in 10..15) {
        delay(900)
        emit(i)
    }
}

val chanelFlow = channelFlow {
    launch {
        flow1().collect { value ->
            print("flow1: ")
            send(value)
        }
    }

    launch {
        flow2().collect { value ->
            print("flow2: ")
            send(value)
        }
    }
}

fun main() = runBlocking {
    chanelFlow.collect { value -> println(value) }
}
```
### Sử dụng flow trong thực tế
- **Kết hợp với Firebase Realtime Database:** Flow thường được dùng để lắng nghe dữ liệu thay đổi từ Firebase:
  - Giả sử Realtime DB có cấu trúc
```Json
users/
   user1/
      name: "Alice"
      age: 25
   user2/
      name: "Bob"
      age: 30
```

  - Triển khai với Kotlin: Dùng callbackFlow để chuyển đổi từ listener của Firebase thành một luồng dữ liệu dạng Flow:
```Kotlin
import com.google.firebase.database.*
import kotlinx.coroutines.channels.awaitClose
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.callbackFlow

// Hàm lắng nghe dữ liệu từ Firebase và trả về Flow
fun getUsersFlow(): Flow<DataSnapshot> = callbackFlow {
    val databaseReference = FirebaseDatabase.getInstance().getReference("users")
    
    // Tạo listener
    val listener = object : ValueEventListener {
        override fun onDataChange(snapshot: DataSnapshot) {
            trySend(snapshot) // Phát dữ liệu khi có thay đổi
        }

        override fun onCancelled(error: DatabaseError) {
            close(error.toException()) // Đóng Flow nếu có lỗi
        }
    }
    
    // Gắn listener vào database
    databaseReference.addValueEventListener(listener)
    
    // Đảm bảo listener được hủy khi Flow bị đóng
    awaitClose { databaseReference.removeEventListener(listener) }
}
```
  - Sử dụng Flow trong viewModel
```Kotlin
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch

class UsersViewModel : ViewModel() {
    private val _usersFlow = MutableStateFlow<List<User>>(emptyList()) // Trạng thái dữ liệu
    val usersFlow = _usersFlow.asStateFlow() // Expose dữ liệu dưới dạng StateFlow

    init {
        viewModelScope.launch {
            getUsersFlow().collect { snapshot ->
                val users = snapshot.children.mapNotNull { it.getValue(User::class.java) }
                _usersFlow.value = users // Cập nhật trạng thái
            }
        }
    }
}

// Data class để ánh xạ dữ liệu Firebase
data class User(
    val name: String = "",
    val age: Int = 0
)
```
  - Sử dụng trong UI
```Kotlin
import androidx.fragment.app.Fragment
import androidx.fragment.app.viewModels
import androidx.lifecycle.lifecycleScope
import kotlinx.coroutines.flow.collect
import kotlinx.coroutines.launch

class UsersFragment : Fragment() {
    private val viewModel: UsersViewModel by viewModels()

    override fun onStart() {
        super.onStart()

        // Lắng nghe dữ liệu và cập nhật giao diện
        lifecycleScope.launch {
            viewModel.usersFlow.collect { users ->
                // Cập nhật danh sách người dùng lên RecyclerView hoặc UI
                users.forEach { user ->
                    println("User: ${user.name}, Age: ${user.age}")
                }
            }
        }
    }
}
```
=> **Điểm mạnh:**
  - **Realtime update:** Dữ liệu được tự động cập nhật khi Firebase thay đổi.
  - **Không rò rỉ bộ nhớ:** Listener của Firebase được gỡ bỏ tự động khi Flow đóng.
  - **Kết hợp dễ dàng với ViewModel:** StateFlow giúp quản lý trạng thái dữ liệu dễ dàng hơn.

## Toán tử trong Flow
- **take(param)**: lấy một lượng giới hạn các phần tử được phát ra từ nguồn phát.
```Kotlin
fun main() = runBlocking {
    flowOf(1, 2, 3, 4, 5)
        .take(3) // Lấy 3 giá trị đầu tiên
        .collect { println(it) } // Output: 1, 2, 3
}
```
- **map()**: Chuyển đổi từng giá trị trong Flow.
```Kotlin
fun main() = runBlocking {
    flowOf(1, 2, 3)
        .map { it * 2 } // Nhân đôi mỗi giá trị
        .collect { println(it) } // Output: 2, 4, 6
}
```
- **transform()**: Toán tử này được dùng để biến đổi giá trị được phát ra từ nguồn phát trước khi emit cho nguồn thu nhận nó
```Kotlin
fun main() = runBlocking {
    flowOf(1, 2, 3)
        .transform { value ->
            emit("Value: $value") // Phát ra chuỗi
            emit(value * 2) // Phát ra giá trị đã nhân đôi
        }
        .collect { println(it) }
        // Output: Value: 1, 2, Value: 2, 4, Value: 3, 6
}
```
- **onEach()**: Toán tử này dùng khi ta muốn thực hiện một action gì đó trước khi value từ flow được emit.
```Kotlin
fun main() = runBlocking {
    flowOf(1, 2, 3)
        .onEach { println("Processing: $it") } // In ra log cho mỗi giá trị
        .collect { println("Collected: $it") }
        // Output: 
        // Processing: 1
        // Collected: 1
        // Processing: 2
        // Collected: 2
        // Processing: 3
        // Collected: 3
}
```
- **reduce()**: Tính toán một kết quả duy nhất từ các giá trị trong Flow.
```Kotlin
fun main() = runBlocking {
    val result = flowOf(1, 2, 3)
        .reduce { accumulator, value -> accumulator + value } // Cộng dồn
    println("Sum: $result") // Output: Sum: 6
}
```
- **fold(param)**: Tương tự `reduce()` nhưng có giá trị khởi tạo
```Kotlin
fun main() = runBlocking {
    val result = flowOf(1, 2, 3)
        .fold(10) { accumulator, value -> accumulator + value } // Bắt đầu từ 10
    println("Sum: $result") // Output: Sum: 16
}
```

- **debounce(param)**: Chỉ phát giá trị cuối cùng nếu không có giá trị mới trong một khoảng thời gian. Có thể ứng dụng làm cho `search`
```Kotlin
fun main() = runBlocking {
    flow {
        emit(1)
        delay(100)
        emit(2)
        delay(400)
        emit(3)
    }
        .debounce(300) // Chỉ phát giá trị nếu không có giá trị mới trong 300ms
        .collect { println(it) } // Output: 2, 3
}
```
- **sample()**: Phát ra giá trị gần đây nhất trong khoảng thời gian đều đặn.
```Kotlin
fun main() = runBlocking {
    flow {
        repeat(10) {
            emit(it)
            delay(100)
        }
    }
        .sample(300) // Lấy mẫu mỗi 300ms
        .collect { println(it) } // Output: 2, 5, 8
}
```
- **flatMapMerge()**: Kết hợp nhiều Flow chạy song song.
```Kotlin
fun main() = runBlocking {
    val flow = flowOf(1, 2, 3) // Luồng gốc
    flow.flatMapMerge { value ->
        flow {
            emit("$value - A")
            delay(300) // Mỗi luồng con mất 300ms để hoàn thành
            emit("$value - B")
        }
    }.collect { println(it) }
}

1 - A
1 - B
2 - A
2 - B
3 - A
3 - B
```
- **flatMapConcat()**: Tương tự flatMapMerge() nhưng xử lý tuần tự từng Flow.
```Kotlin
fun main() = runBlocking {
    val flow = flowOf(1, 2, 3) // Luồng gốc
    flow.flatMapConcat { value ->
        flow {
            emit("$value - A")
            delay(300) // Mỗi luồng con mất 300ms để hoàn thành
            emit("$value - B")
        }
    }.collect { println(it) }
}
1 - A
2 - A
3 - A
1 - B
2 - B
3 - B
```
- **[flow].combine([flow])**: phát giá trị ngay khi bất kỳ luồng nào phát ra giá trị mới. Nó kết hợp giá trị mới nhất từ mỗi luồng.
```Kotlin
fun main() = runBlocking {
    val numbers = flow {
        emit(1)
        delay(100)
        emit(2)
        delay(100)
        emit(3)
    }

    val letters = flow {
        delay(50)
        emit("A")
        delay(150)
        emit("B")
    }

    numbers.combine(letters) { number, letter ->
        "$number -> $letter"
    }.collect { println(it) }
}
1 -> A
2 -> A
2 -> B
3 -> B

```
- **[flowA].zip([flowB])**: chỉ phát ra một giá trị khi tất cả các luồng đều phát ra một giá trị mới. Nó ghép cặp từng giá trị tương ứng từ các luồng.
```Kotlin
fun main() = runBlocking {
    val numbers = flow {
        emit(1)
        delay(100)
        emit(2)
        delay(100)
        emit(3)
    }

    val letters = flow {
        emit("A")
        delay(150)
        emit("B")
    }

    numbers.zip(letters) { number, letter ->
        "$number -> $letter"
    }.collect { println(it) }
}
```
- **filter()**: lọc ra các giá trị thỏa mãn điều kiện và bỏ qua các giá trị không thỏa mãn điều kiện từ nguồn phát.
```Kotlin
fun main() = runBlocking {
    flowOf(1, 2, 3, 4, 5)
        .filter { it % 2 == 0 } // Chỉ giữ lại các số chẵn
        .collect { println(it) }
}
```
## Cold flow vs Hot flow. Flow vs StateFlow vs SharedFlow.
### Cold flow vs Hot flow
#### Cold flow
- Đặc điểm chính
  - **Flow lười biếng (lazy):** `Cold Flow` không phát dữ liệu cho đến khi có collector đăng ký và bắt đầu thu thập dữ liệu.
  - **Không chia sẻ dữ liệu:** Mỗi `collector` sẽ nhận được luồng dữ liệu từ đầu, độc lập với các `collector` khác.
  - **Không hoạt động khi không có collector:** `Cold Flow` chỉ tồn tại khi có người dùng đang thu thập dữ liệu. Khi không có collector, nó không phát ra dữ liệu.

- Cách hoạt động của Cold Flow:
  - Tạo Flow bằng cách sử dụng hàm `flow {}`.
  - Flow bắt đầu phát dữ liệu khi có `collector` gọi hàm `collect()`.
  - Mỗi lần `collector` thu thập, `Flow` chạy lại từ đầu.

```Kotlin
fun main() = runBlocking {
    val coldFlow = flow {
        println("Flow started")
        for (i in 1..3) {
            delay(100) // Giả lập công việc tốn thời gian
            emit(i)
        }
    }

    println("Collector 1 starts collecting")
    coldFlow.collect { println("Collector 1 received: $it") }

    println("Collector 2 starts collecting")
    coldFlow.collect { println("Collector 2 received: $it") }
}

Flow started
Collector 1 received: 1
Collector 1 received: 2
Collector 1 received: 3
Collector 2 starts collecting
Flow started
Collector 2 received: 1
Collector 2 received: 2
Collector 2 received: 3
```
#### Hot Flow
- **Đặc điểm chính**
  - **Luôn hoạt động (always active)**: `Hot Flow` phát dữ liệu liên tục ngay cả khi không có `collector `thu thập.
  - **Chia sẻ dữ liệu:** Nhiều `collector` có thể nhận dữ liệu cùng một thời điểm.
  - **Collector nhận dữ liệu tại thời điểm đăng ký:** Không giống Cold Flow, `collector` sẽ không nhận được dữ liệu đã phát trước khi nó đăng ký.

- **Cách hoạt động của Hot Flow**
  - Tạo một luồng dữ liệu (Flow) luôn phát giá trị. Ví dụ: `StateFlow, SharedFlow, hoặc BroadcastChannel.`
  - Dữ liệu được phát ra ngay lập tức khi có hoặc không có collector.
  - `Collector` nhận giá trị tại thời điểm nó đăng ký (hoặc theo cấu hình replay).

```Kotlin
fun main(): Unit = runBlocking {
    val hotFlow = MutableSharedFlow<Int>() // Tạo Hot Flow

    // Luồng phát giá trị
    launch {
        for (i in 1..3) {
            delay(100) // Giả lập thời gian chờ
            println("Emitting $i")
            hotFlow.emit(i) // Phát giá trị
        }
    }

    delay(150) // Collector đăng ký muộn

    // Collector thu thập giá trị từ thời điểm đăng ký
    hotFlow.collect { println("Collector received: $it") }
}
Emitting 1
Emitting 2
Collector received: 2
Emitting 3
Collector received: 3
```
#### So sánh Cold Flow và Hot Flow
|Đặc điểm	|Cold Flow	|Hot Flow|
|---|---|---|
|Hoạt động khi không có collector?	|Không hoạt động	|Luôn hoạt động|
|Phát giá trị lại từ đầu?	|Có, với mỗi collector	|Không, collector nhận từ thời điểm đăng ký|
|Chia sẻ dữ liệu?	|Không	|Có|
|Ứng dụng phổ biến	|Dữ liệu không liên tục (API, tệp, DB)	|Trạng thái UI, sự kiện (event)|
|Ví dụ triển khai	|`flow {}`	|`StateFlow, SharedFlow`|

#### 3. Khi nào dùng Cold Flow hoặc Hot Flow?
|Tình huống	|Cold Flow	|Hot Flow|
|---|---|---|
|Luồng dữ liệu tuyến tính|	✔|	✘|
|Cập nhật trạng thái UI liên tục	|✘	|✔|
|Xử lý sự kiện (event)	|✘	|✔
|Dữ liệu không liên tục (API, DB, file)|	✔|	✘|
|Nhiều collector cần chia sẻ dữ liệu|	✘|	✔|

### Flow vs StateFlow vs SharedFlow.
#### Flow
- **Đặc điểm chính**
  - **Loại:** Cold Flow.
  - **Tính năng:**
    - **Lười biếng (lazy):** Chỉ phát giá trị khi có `collector`.
    - **Không giữ giá trị:** Khi không có `collector`, dữ liệu không được lưu lại.
    - **Không chia sẻ:** Mỗi `collector` sẽ nhận được toàn bộ dữ liệu từ đầu.
    - **Kết thúc:** Flow có thể hoàn thành hoặc bị hủy, không chạy mãi mãi.
- **Khi nào dùng Flow?**
  - Dùng cho luồng dữ liệu không liên tục hoặc không cần lưu trạng thái.
  - **Ví dụ:** Lấy danh sách từ API, đọc dữ liệu từ tệp.

```Kotlin
fun main() = runBlocking {
    val coldFlow = flow {
        for (i in 1..3) {
            emit(i) // Phát giá trị
        }
    }

    println("Collector 1:")
    coldFlow.collect { println(it) } // Collector đầu tiên

    println("Collector 2:")
    coldFlow.collect { println(it) } // Collector thứ hai
}

Collector 1:
1
2
3
Collector 2:
1
2
3
```
#### StateFlow
- **Đặc điểm chính**
  - **Loại:** Hot Flow.
  - Tính năng:
    - **Giữ giá trị mới nhất (state):** Nếu có `collector` mới, nó sẽ nhận ngay giá trị hiện tại.
    - **Luôn hoạt động (always active):** Hoạt động ngay cả khi không có `collector.`
    - **Chỉ phát giá trị mới:** `Collector` chỉ nhận giá trị khi có thay đổi trạng thái.
    - **Không hoàn thành:** `StateFlow` không bao giờ hoàn thành, phù hợp để lưu trữ trạng thái trong thời gian dài (như `ViewModel`).
- **Khi nào dùng StateFlow?**
  - Lưu trữ và theo dõi trạng thái UI trong `ViewModel`.
  - **Ví dụ:** Theo dõi trạng thái tải dữ liệu, danh sách sản phẩm được hiển thị.
```Kotlin
fun main():Unit = runBlocking {
    val stateFlow = MutableStateFlow(0)

    // Bắt đầu phát giá trị
    launch {
        for (i in 1..3) {
            delay(100)
            stateFlow.value = i
        }
    }

    delay(300) // Collector đăng ký muộn
    stateFlow.collect { println("Collector received: $it") }
}
Collector received: 2
Collector received: 3
```
**Ví dụ chi tiết về StateFlow trong ViewModel và Fragment**: hiển thị trạng thái tải dữ liệu (loading, success, error) trong một ứng dụng và cập nhật UI dựa trên trạng thái này.

```Kotlin
//ViewModel
import androidx.lifecycle.ViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.Dispatchers

class MyViewModel : ViewModel() {
    // Tạo StateFlow để lưu trạng thái
    private val _uiState = MutableStateFlow<UIState>(UIState.Loading)
    val uiState: StateFlow<UIState> = _uiState.asStateFlow()

    // Giả lập tải dữ liệu
    fun fetchData() {
        CoroutineScope(Dispatchers.IO).launch {
            _uiState.value = UIState.Loading // Trạng thái ban đầu: đang tải
            delay(2000) // Giả lập thời gian tải dữ liệu
            _uiState.value = UIState.Success("Dữ liệu đã tải thành công!") // Trạng thái thành công
        }
    }

    fun simulateError() {
        CoroutineScope(Dispatchers.IO).launch {
            _uiState.value = UIState.Loading // Trạng thái ban đầu: đang tải
            delay(2000) // Giả lập thời gian tải dữ liệu
            _uiState.value = UIState.Error("Đã xảy ra lỗi khi tải dữ liệu.") // Trạng thái lỗi
        }
    }
}

// Định nghĩa trạng thái UI
sealed class UIState {
    object Loading : UIState()
    data class Success(val message: String) : UIState()
    data class Error(val errorMessage: String) : UIState()
}
```
- **Fragment quan sát**
```Kotlin
import android.os.Bundle
import android.view.View
import android.widget.TextView
import android.widget.ProgressBar
import android.widget.Button
import androidx.fragment.app.Fragment
import androidx.fragment.app.viewModels
import androidx.lifecycle.lifecycleScope
import kotlinx.coroutines.flow.collect

class MyFragment : Fragment(R.layout.fragment_my) {

    private val viewModel: MyViewModel by viewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        // Tham chiếu các view
        val progressBar = view.findViewById<ProgressBar>(R.id.progressBar)
        val messageTextView = view.findViewById<TextView>(R.id.messageTextView)
        val loadButton = view.findViewById<Button>(R.id.loadButton)
        val errorButton = view.findViewById<Button>(R.id.errorButton)

        // Quan sát StateFlow
        lifecycleScope.launchWhenStarted {
            viewModel.uiState.collect { uiState ->
                when (uiState) {
                    is UIState.Loading -> {
                        progressBar.visibility = View.VISIBLE
                        messageTextView.visibility = View.GONE
                    }
                    is UIState.Success -> {
                        progressBar.visibility = View.GONE
                        messageTextView.visibility = View.VISIBLE
                        messageTextView.text = uiState.message
                    }
                    is UIState.Error -> {
                        progressBar.visibility = View.GONE
                        messageTextView.visibility = View.VISIBLE
                        messageTextView.text = uiState.errorMessage
                    }
                }
            }
        }

        // Sự kiện khi nhấn nút
        loadButton.setOnClickListener { viewModel.fetchData() }
        errorButton.setOnClickListener { viewModel.simulateError() }
    }
}
```
- **XML**
```xml
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center"
    android:padding="16dp">

    <ProgressBar
        android:id="@+id/progressBar"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:visibility="gone" />

    <TextView
        android:id="@+id/messageTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:visibility="gone"
        android:textSize="16sp"
        android:padding="8dp" />

    <Button
        android:id="@+id/loadButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Tải dữ liệu" />

    <Button
        android:id="@+id/errorButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Giả lập lỗi" />
</LinearLayout>
```

#### SharedFlow
- **Đặc điểm chính**
  - **Loại:** Hot Flow.
  - **Tính năng:**
    - **Không giữ trạng thái:** Mặc định, không giữ lại giá trị mới nhất.
    - **Chia sẻ giá trị:** Nhiều collector có thể nhận dữ liệu từ cùng một SharedFlow.
    - **Replay**: Có thể cấu hình để phát lại một số giá trị gần đây cho collector mới đăng ký.
    - **Không hoàn thành:** Phù hợp để phát sự kiện (event) hoặc dữ liệu không cần lưu trạng thái.
- **Khi nào dùng SharedFlow?**
  - Dùng để phát sự kiện (như click button, thông báo từ server).
  - Ví dụ: Thông báo thành công/thất bại, sự kiện trong ứng dụng.

```kotlin
fun main():Unit = runBlocking {
    val sharedFlow = MutableSharedFlow<Int>(replay = 2) // Phát lại 2 giá trị gần nhất

    // Phát giá trị
    launch {
        for (i in 1..5) {
            delay(100)
            sharedFlow.emit(i)
        }
    }

    // Collector đăng ký sau khi SharedFlow đã phát giá trị
    delay(300)
    sharedFlow.collect { println("Collector received: $it") }
}

Collector received: 1
Collector received: 2
Collector received: 3
Collector received: 4
Collector received: 5
Do replay = 2 nên lấy lại 2 giá trị 1, 2
```

- **Ví dụ chi tiết về SharedFlow**: phát các sự kiện (event) cho nhiều collector nhưng không lưu trạng thái mới nhất như StateFlow. SharedFlow được sử dụng để chia sẻ một luồng dữ liệu giữa nhiều collector cùng lúc.
```Kotlin
fun main() = runBlocking {
    val sharedFlow = MutableSharedFlow<String>()

    // Collector đầu tiên
    launch {
        sharedFlow.collect { println("Collector 1 received: $it") }
    }

    // Collector thứ hai
    launch {
        sharedFlow.collect { println("Collector 2 received: $it") }
    }

    delay(100) // Đợi các collector sẵn sàng trước khi phát dữ liệu

    // Phát dữ liệu
    sharedFlow.emit("Event 1")
    sharedFlow.emit("Event 2")
}
Collector 1 received: Event 1
Collector 2 received: Event 1
Collector 1 received: Event 2
Collector 2 received: Event 2
```

#### So sánh chi tiết: Flow, StateFlow, SharedFlow
|Đặc điểm	|Flow	StateFlow|	SharedFlow|
|---|---|---|
|Loại Flow	|Cold	Hot|	Hot|
|Lưu giá trị mới nhất	|Không|	Có	|Không (có thể replay)|
|Chia sẻ giá trị	Không	|Có|	Có|
|Bắt đầu từ đầu	|Có	Không|	Không|
|Hoạt động khi không có collector	|Không|	Có|	Có|
|Ứng dụng	|Dữ liệu tuyến tính, không liên tục	|Trạng thái UI (state)	|Sự kiện (event)|

#### Khi nào dùng loại nào?
- **Flow:** Khi cần xử lý luồng dữ liệu tuyến tính và không liên tục, không cần chia sẻ (VD: truy vấn API, đọc dữ liệu từ tệp).

- **StateFlow:** Khi cần lưu trữ trạng thái và phát giá trị mới nhất cho collector mới. Thường dùng trong ViewModel để lưu trạng thái UI.

- **SharedFlow:** Khi cần phát sự kiện hoặc thông báo đến nhiều collector. Phù hợp với các sự kiện không cần lưu trữ trạng thái.

## So sánh với LiveData.
- Cả `Flow` và `LiveData` đều được sử dụng để xử lý dữ liệu không đồng bộ, đặc biệt trong các ứng dụng Android. Tuy nhiên, chúng có một số điểm khác biệt quan trọng về cách hoạt động, khả năng sử dụng, và các tình huống nên áp dụng.
### Tổng quan
|Tiêu chí	|Flow	|LiveData|
|---|---|---|
|Định nghĩa	|API của Kotlin dùng để xử lý luồng dữ liệu không đồng bộ.	|API của Android dùng để quan sát dữ liệu theo vòng đời.|
|Vòng đời nhận biết	|Không nhận biết vòng đời.	|Nhận biết vòng đời (LifecycleAware).|
|Luồng dữ liệu	|Hỗ trợ một hoặc nhiều giá trị phát ra theo thời gian.	|Thường phát giá trị hiện tại (một giá trị tại một thời điểm).|
|Ngữ cảnh sử dụng	|Sử dụng trong mọi lớp (Repository, ViewModel, UseCase, v.v.).	|Chủ yếu sử dụng trong ViewModel và UI.|
### Chi tiết so sánh
#### Nhận thức vòng đời
|Flow	|LiveData|
|---|---|
|Không tự động nhận biết vòng đời. Cần quản lý thủ công bằng cách sử dụng `lifecycleScope` hoặc `repeatOnLifecycle`.	|Tự động tạm dừng khi vòng đời không ở trạng thái active (`STARTED` hoặc `RESUMED`).|

- Với `Flow`, nếu bạn không hủy lắng nghe thủ công, ứng dụng có thể bị rò rỉ bộ nhớ.
- Với `LiveData`, nó tự động hủy lắng nghe khi vòng đời không còn hoạt động.
####  Hỗ trợ luồng dữ liệu

|Flow|	LiveData|
|---|---|
|Hỗ trợ luồng dữ liệu (dòng liên tục) với nhiều giá trị được phát ra.	|Chỉ phát giá trị hiện tại. Không hỗ trợ luồng nhiều giá trị.|
|Có các toán tử mạnh mẽ như` map, filter, combine, flatMapConcat, v.v.`	|Không có hỗ trợ toán tử xử lý dữ liệu mạnh mẽ.|

#### Sử dụng trong ViewModel và UI
|Flow|	LiveData|
|---|---|
|Cần kết hợp với lifecycleScope hoặc viewModelScope để quan sát dữ liệu.	|Dễ dàng kết nối với UI qua observe() mà không cần thêm xử lý.
|Không tự động làm việc với vòng đời UI, cần sử dụng repeatOnLifecycle để tránh rò rỉ bộ nhớ.	|Tự động làm việc với vòng đời UI (ngưng quan sát khi UI bị hủy).|

```Kotlin
//Flow
lifecycleScope.launchWhenStarted {
    flow.collect { value ->
        // Cập nhật giao diện
    }
}

//livedata
liveData.observe(viewLifecycleOwner) { value ->
    // Cập nhật giao diện
}
```

####  Hỗ trợ API
|Flow|	LiveData|
|---|---|
|Hỗ trợ API của Kotlin, dễ dàng tích hợp với suspend, coroutines.	|Chỉ hoạt động tốt trong môi trường Android.|
|Dễ dàng sử dụng trong các lớp không liên quan đến Android, ví dụ Repository, UseCase.	|Chủ yếu sử dụng trong ViewModel và UI.|

### Khi nào nên dùng Flow và LiveData?
- **Nên dùng Flow khi:**
  - Cần xử lý luồng dữ liệu liên tục (streams), ví dụ:
    - Dữ liệu từ API hoặc cơ sở dữ liệu (Room, Firebase).
    - Sự kiện thời gian thực, như nhấn nút hoặc thông báo.
  - Sử dụng trong lớp Repository hoặc UseCase để tách biệt logic không liên quan đến Android.
  - Cần các toán tử mạnh mẽ để xử lý dữ liệu.
- **Nên dùng LiveData khi:**
  - Dữ liệu được liên kết chặt với UI vòng đời (Lifecycle-aware).
  - Cần đơn giản hóa việc cập nhật giao diện mà không phải quản lý vòng đời thủ công.
  - Dữ liệu không phát ra liên tục (ví dụ, trạng thái UI).

|Flow|	LiveData|
|---|---|
|Đa nền tảng (Kotlin).	|Chỉ hoạt động trong môi trường Android.|
|Hỗ trợ xử lý dữ liệu mạnh mẽ.	|Tự động nhận biết vòng đời.|
|Phát nhiều giá trị theo thời gian.	|Chỉ lưu và phát giá trị hiện tại.|
|Quản lý vòng đời thủ công.|	Tự động quản lý vòng đời.|

