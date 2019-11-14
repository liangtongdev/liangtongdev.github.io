---
layout: post
title: Go语言笔记
date: 2019-11-13 19:29
catalog: true
categories: go
tags: note

---


缘起，一直对go有浓烈的兴趣，从2018年接触go语言，中间因为没有实际应用，经历了多次的学了忘，忘了再学。
直到某天在github上看到了某个牛人总结的go的学习路线，于是决定抽空对go进行系统的学习，这里首先记录下Go语言的环境，基础，并发，OOP及高级操作。其中后续的记录以代码为主。后续更新吧


### go介绍

#### 目录结构

通常情况下，GoPath中包含3个目录。bin中存放经编译的可执行文件；pkg存放过程中产生的.a文件、mod文件；src中存放的是源码

```bash
go
| -- bin
| -- pkg
| -- src
|    | -- App1
|    | -- App2
```



#### 常用命令

+ go run : 编译并直接运行程序，会产生临时文件但不产生可执行文件
+ go build： 会产生临时文件和可执行文件
+ go install：编译导入包，编译主程序，将编译结果放到bin目录下，编译后的包文件放到pkg目录下
+ go test: 测试，运行*_test.go中的测试函数



### 基础

#### 变量

```go
var age int //变量声明
age = 10 //赋值

var age int = 10 //变量声明并初始化

var age = 10 //定义变量并初始化，使用类型推断

var width, height int//多变量声明
width = 100
height = 50
var width, heigth int = 100, 50 //定义多个变量
var width, height = 100,50 //使用类型推断

age := 10 //简短声明
name, age := "liangtong", "20"
```

注意：**简短声明的左边，必须至少有一个变量是新声明的** 。 尤其要注意 **:=** 和 **=** 的区别

```go
package main

import (  
    "fmt"
    "strconv"
)

func main() {  
    i := 2 
    s := "1000"
    if len(s) > 1 {
        i, _ := strconv.Atoi(s)//[注1]
        i = i + 5
    }
    fmt.Println(i)
}

// 输出 2 。原因 [注1] 处声明新的变量 i， 作用域为大括号
```

#### 类型

基本类型包括 

+ 布尔类型
+ 数字类型
  + int8，int16, int32, int64, int
  + uint8, uint16, uint32, uint64, uint
  + float32, float64
  + complex64, complex128
  + byte : 可以理解为unit8的别名
  + rune：可以理解为int32的别名
+ 字符串

```go
i := 10 // int
j := 20.3 //float64
sum := i + int(j) //将j的类型转化成int
```



#### 常量

```go
const a = 55 // 声明, 关键字 const
const b = math.Sqrt(4) // NG . 运行时才能确定是不允许的
```



#### 函数

```go
//定义样式，可以有多个（也可以没有）参数和返回值
func functionname(parametername type) returntype {  
 //function body
}

//例如 两个返回值
func rectProps(length, width float64)(float64, float64) {  
    var area = length * width
    var perimeter = (length + width) * 2
    return area, perimeter
}
//指定返回值
func rectProps(length, width float64)(area, perimeter float64) {  
    area = length * width
    perimeter = (length + width) * 2
    return //no explicit return value
}
//下划线
func main() {  
    area, _ := rectProps(10.8, 5.6) // perimeter is discarded
    fmt.Printf("Area %f ", area)
}
```

#### 包

每个可执行的应用都必须在**main package**中包含一个**main** 函数

包引用，通过 **import** 关键字。 **包中首字母大写的方法/函数/变量才能被引用包使用**

可以在import中对引用包起别名，例如

```go
import(
    "fmt"
    goTest "github.com/liangtongdev/go-test"
)
```

#### if else

```go
if condition {  
} else if condition {
} else {
}

if statement; condition {  
}
//condition 可以是bool值/表达式
```

注：  **if  后边的大括号'{}'是必须的，else必须与前边的后大括号'}'在同一行**。 以下代码会编译不通过

```go
package main

import (  
    "fmt"
)

func main() {  
    num := 10
    if num % 2 == 0 { //checks if number is even
        fmt.Println("the number is even") 
    }  
    else {
        fmt.Println("the number is odd")
    }
}

//原因是go会自动在if大括号后边{}添加分号;程序会变成
if num%2 == 0 {  
      fmt.Println("the number is even") 
};  //; inserted by Go
else {  
      fmt.Println("the number is odd")
}
```

#### 循环

```go
for initialisation; condition; post {  //循环
}
for {  //无限循环
}
```

#### Switch

**case** 中可以包含多个匹配，case后可以跟表达式。

与C等其他语言不通的是case匹配后，直接跳出switch块，如果需要继续类似C语言的行为，可以通过添加**fallthrough**关键字

```go
package main

import (  
    "fmt"
)

func number() int {  
        num := 15 * 5 
        return num
}

func main() {

    switch num := number(); { //num is not a constant
    case num < 50:
        fmt.Printf("%d is lesser than 50\n", num)
        fallthrough
    case num < 100:
        fmt.Printf("%d is lesser than 100\n", num)
        fallthrough
    case num < 200:
        fmt.Printf("%d is lesser than 200", num)
    }
}
```

#### 数组和切片

```go
var a [3]int // 整形数组，长度为3
a := [3]int{10, 20, 30} //[10 20 30]
a := [3]int{10} //[10 0 0]
a := [...]int{10, 20, 30} //[10 20 30]，长度由编译时决定
```

**数组是值类型**

数组的遍历，可以通过for循环和range语法

```go
package main

import "fmt"

func main() {  
    a := [...]float64{67.7, 89.8, 21, 78}
    for i := 0; i < len(a); i++ { //looping from 0 to the length of the array
        fmt.Printf("%d th element of a is %.2f\n", i, a[i])
    }
    
    sum := float64(0)
    for i, v := range a {//range returns both the index and value
        fmt.Printf("%d the element of a is %.2f\n", i, v)
        sum += v
    }
    fmt.Println("\nsum of all elements of a",sum)
}
```

切片：**T类型的切片通过 []T 表示**

```go
var b []int //切片
b := []int{6, 7, 8}

darr := [...]int{57, 89, 90, 82, 100, 78, 67, 69, 59}
dslice := darr[2:5]
```

通过 **make** 语法创建切片 ***func make([]T, len, cap) []T* **

```go
package main

import (  
    "fmt"
)

func main() {  
    i := make([]int, 5, 5)
    fmt.Println(i)
  	i = append(i, 1, 2, 3)
  
    veggies := []string{"potatoes","tomatoes","brinjal"}
    fruits := []string{"oranges","apples"}
    food := append(veggies, fruits...)//注意...的使用
}
```

数组是值类型，可以将数组切片以参数传递给其他函数来修改数组的内容。

**当使用数组切片的时候，只要切片存在于内存中，数组就不能被回收。可以借助于copy函数来避免这种情况发生**

#### 可变参函数

```go
func hello(a int, b ...int) {  
}
```

**可变参函数的是将可变的参数转换为对应参数类型的切片**

切片与可变参函数，注意**···语法糖**

```go
package main

import (  
    "fmt"
)

func find(num int, nums ...int) {  
    fmt.Printf("type of nums is %T\n", nums)
    found := false
    for i, v := range nums {
        if v == num {
            fmt.Println(num, "found at index", i, "in", nums)
            found = true
        }
    }
    if !found {
        fmt.Println(num, "not found in ", nums)
    }
    fmt.Printf("\n")
}
func main() {  
    nums := []int{89, 90, 95}
    find(89, nums...)
}
```

#### 字典

是 **引用** 类型，通过make函数创建

```go
personSalary := make(map[string]int)  
```

字典的操作。 通过 **value, ok := map[key] ** 来判断key是否存在与map中；通过**for range**来遍历

通过**delete(map, key)** 来删除map中特定key， **如果key不存在，则无任何影响**



```go
package main

import (  
    "fmt"
)

func main() {  
    personSalary := make(map[string]int)
    personSalary["steve"] = 12000
    personSalary["jamie"] = 15000
    personSalary["mike"] = 9000
    employee := "jamie"
    fmt.Println("Salary of", employee, "is", personSalary[employee])
  
    newEmp := "joe"
    value, ok := personSalary[newEmp]
    if ok == true {
        fmt.Println("Salary of", newEmp, "is", value)
    } else {
        fmt.Println(newEmp,"not found")
    }
  
    for key, value := range personSalary {
        fmt.Printf("personSalary[%s] = %d\n", key, value)
    }
}
```

#### 字符串

字符串是字节的切片。用引号包含。字符串是不可变的

**rune** Unicode中超出UTF-8范围的编码字符，可以通过rune来表示

#### 指针

存储变量地址的变量，通过 ***T** 来定义指针

```go
b := 255
var a *int = &b
```

通过 **new** 函数创建 指针

```go
package main

import (  
    "fmt"
)

func main() {  
    size := new(int)
    fmt.Printf("Size value is %d, type is %T, address is %v\n", *size, size, size)
    *size = 85
    fmt.Println("New size value is", *size)
}
```

指针作为函数参数

```go
package main

import (  
    "fmt"
)

func change(val *int) {  
    *val = 55
}
func main() {  
    a := 58
    fmt.Println("value of a before function call is",a)
    b := &a
    change(b)
    fmt.Println("value of a after function call is", a)
}
```

**尽管将数组指针和数组切片作为函数传递均能解决实际问题。但建议使用数组切片作为函数参数传递，因为使用切片更清晰，更常用**

**go语言不支持指针运算**。 例如以下代码会出现错误

```go
package main

func main() {  
    b := [...]int{109, 110, 111}
    p := &b
    p++
}
```

#### 结构体

一组数据的集合

```go
type Employee struct {  
    firstName string
    lastName  string
    age       int
}
//多个参数同一行
type Employee struct {  
    firstName, lastName string
    age, salary         int
}
```

通过点`.`操作符来访问结构体变量

结构体匿名字典

```go
type Person struct {  
    string
    int
}
```

**即使结构体匿名字段没有名字，那些匿名的字段的默认名字是他们的类型。** 例如上述可以通过`string`和`int`来访问对应的字段

结构体嵌套，通过`.`操作符来访问。（嵌套结构体）字段的提升

```go
type Address struct {  
    city, state string
}
type Person struct {  
    name string
    age  int
    Address //提升字段，可以直接访问其中的字段
}

//例如以下代码
func main() {  
    var p Person
    p.name = "Naveen"
    p.age = 50
    p.Address = Address{
        city:  "Chicago",
        state: "Illinois",
    }
    fmt.Println("Name:", p.name)
    fmt.Println("Age:", p.age)
    fmt.Println("City:", p.city) //city is promoted field
    fmt.Println("State:", p.state) //state is promoted field
}
```

#### 方法

与函数的区别，是方法添加了接收者

```go
func (t Type) methodName(parameter list) {  
}
```

为什么要使用方法

+ go语言不是一个纯粹的面相对象语言。并不支持`class`， 因此用基于类型的方法实现`class`的类似行为。
+ 在不同类型中可以创建相同名称的方法，但是函数不可以。

方法中值接收者和指针接收者的区别

+ 方法内的变更直接影响指针接收者，而值接收者不受影响

#### 接口

`interface`关键字，接口定义对象的行为。

**go语言中，接口是一系列方法的签名。如果一个类型实现了接口中的所有方法，也就是实现了该接口**，与其他语言不同的是，不需要类似`implement`关键字

```go
package main

import (  
    "fmt"
)

type SalaryCalculator interface {  
    CalculateSalary() int
}

type Permanent struct {  
    empId    int
    basicpay int
    pf       int
}

type Contract struct {  
    empId  int
    basicpay int
}

//salary of permanent employee is sum of basic pay and pf
func (p Permanent) CalculateSalary() int {  
    return p.basicpay + p.pf
}

//salary of contract employee is the basic pay alone
func (c Contract) CalculateSalary() int {  
    return c.basicpay
}

/*
total expense is calculated by iterating though the SalaryCalculator slice and summing  
the salaries of the individual employees  
*/
func totalExpense(s []SalaryCalculator) {  
    expense := 0
    for _, v := range s {
        expense = expense + v.CalculateSalary()
    }
    fmt.Printf("Total Expense Per Month $%d", expense)
}

func main() {  
    pemp1 := Permanent{1, 5000, 20}
    pemp2 := Permanent{2, 6000, 30}
    cemp1 := Contract{3, 3000}
    employees := []SalaryCalculator{pemp1, pemp2, cemp1}
    totalExpense(employees)

}
```

**接口可以认为是元组(type, value)**。 其中type这个接口的具体类型，value是该类型的值

```go
package main

import (  
    "fmt"
)

func describe(i interface{}) {  
    fmt.Printf("Type = %T, value = %v\n", i, i)
}

func main() {  
    s := "Hello World"
    describe(s)
    i := 55
    describe(i)
    strt := struct {
        name string
    }{
        name: "Naveen R",
    }
    describe(strt)
}
```

```bash
Type = string, value = Hello World  
Type = int, value = 55  
Type = struct { name string }, value = {Naveen R} 
```

类型推断用来获取接口的值 **i.(T)**，其中i为接口，T为具体类型，例如

```go
var s interface{} = 56
i := s.(int) //get the underlying int value from i
```

如果类型匹配的话，一切均正常。但是当类型不同时，就会出错。

通过**`v, ok := i.(T)`** 来解决这个问题，如果能投提取到对应值，则ok为true。在错误判断时尤其常用到

```go
package main

import (  
    "fmt"
)

func assert(i interface{}) {  
    v, ok := i.(int)
    fmt.Println(v, ok)
}
func main() {  
    var s interface{} = 56
    assert(s)
    var i interface{} = "Steven Paul"
    assert(i)
}

//out put
56 true  
0 false  
```

go不提供继承，但是可以通过嵌入接口来包含其他接口

```go
type SalaryCalculator interface {  
    DisplaySalary()
}

type LeaveCalculator interface {  
    CalculateLeavesLeft() int
}

type EmployeeOperations interface {  
    SalaryCalculator
    LeaveCalculator
}
```



### 并发

并发是go语言与生俱来的。在go中通过 `Goroutines` 和 `Channels` 来实现。

#### Goroutines

Goroutines是并发运行的函数或方法。

Goroutines可以被认为是轻量级的线程；同线程相比，创建Goroutines的代价是微小的，因此go应用在高并发支持上很有优势。

Goroutines 同线程比较的优势

+ Goroutines 非常cheap，只占用几kb的栈空间，并且可以根据应用的需求进行增长或收缩。线程的栈空间必须明确而且是固定的
+ Goroutines 被多路复用到更少的OS线程中。一个程序可能只有一个线程，而程序可以有成千上万个Goroutines。例如线程中的Goroutines表示等待用户输入，则创建一个新的OS线程，将其他的Goroutines移动到新的OS线程。这些均有运行时处理，程序员不用关系
+ Goroutines使用通道进行通信。通道的设计可以防止共享内存时出现死锁。

Goroutines 的启动，通过 `go` 关键字。启动后Goroutines立刻返回；不用等待。主程序完成后，程序退出。

```go
package main

import (  
    "fmt"
    "time"
)

func numbers() {  
    for i := 1; i <= 5; i++ {
        time.Sleep(250 * time.Millisecond)
        fmt.Printf("%d ", i)
    }
}
func alphabets() {  
    for i := 'a'; i <= 'e'; i++ {
        time.Sleep(400 * time.Millisecond)
        fmt.Printf("%c ", i)
    }
}
func main() {  
    go numbers()
    go alphabets()
    time.Sleep(3000 * time.Millisecond)
    fmt.Println("main terminated")
}

//out put
1 a 2 3 b 4 c 5 d e main terminated  
```

#### Channels

Channels可以认为是Goroutines沟通的通道。通道的类型通过`chan T`来声明。通过make定义

```go
package main

import "fmt"

func main() {  
    var a chan int
    if a == nil {
        fmt.Println("channel a is nil, going to define it")
        a = make(chan int)
        fmt.Printf("Type of a is %T", a)
    }
}
```

 channel中数据的存取。**默认情况下是阻塞的**

```go
data := <- a // 读取数据  
a <- data // 写入数据  
```

```go
package main

import (  
    "fmt"
)

func hello(done chan bool) {  
    fmt.Println("Hello world goroutine")
    done <- true
}
func main() {  
    done := make(chan bool)
    go hello(done)
    <-done
    fmt.Println("main function")
}
```

**死锁**：如果一个Goroutine 在channel中写入了数据，然后期望其他Goroutine读取数据，如果没有被读取，那么程序就会发生死锁。

```go
package main


func main() {  
    ch := make(chan int)
    ch <- 5
}
```

单向channel

```go
package main

import "fmt"

func sendData(sendch chan<- int) {  
    sendch <- 10
}

func main() {  
    chnl := make(chan int)
    go sendData(chnl)
    fmt.Println(<-chnl)
}
```

通道发送者可以关闭channel。接收者可以使用额外的变量来判断channel是否关闭

```go
v, ok := <- ch  
```

如果 OK 是 true， 表示读取的是一个成功发送到通道的值，false表示是一个关闭的Channel。

使用 `for range` 格式循环也可以接受通道的值直到通道关闭

```go
package main

import (  
    "fmt"
)

func producer(chnl chan int) {  
    for i := 0; i < 10; i++ {
        chnl <- i
    }
    close(chnl)
}
func main() {  
    ch := make(chan int)
    go producer(ch)
  
  //可以通过v, ok := <-ch 读取通道数据。与下二选一
    for {
        v, ok := <-ch
        if ok == false {
            break
        }
        fmt.Println("Received ", v, ok)
    }
  
  //可以通过 for range 读取通道数据。与上二选一
    for v := range ch {
        fmt.Println("Received ",v)
    }
}
```

#### 缓存通道和工作池

默认情况下，通道是不具缓存的。通道数据的存取是阻塞的。**缓存通道不阻塞，直到缓存的数据满**

```go
ch := make(chan type, capacity) 
```

capacity 表示缓存的大小，缓存通道的capacity值必须大于0。而无缓存的通道的capacity为0

```go
package main

import (  
    "fmt"
    "time"
)

func write(ch chan int) {  
    for i := 0; i < 5; i++ {
        ch <- i
        fmt.Println("successfully wrote", i, "to ch")
    }
    close(ch)
}
func main() {  
    ch := make(chan int, 2)
    go write(ch)
    time.Sleep(2 * time.Second)
    for v := range ch {
        fmt.Println("read value", v,"from ch")
        time.Sleep(2 * time.Second)

    }
}
```

`WaitGroup` 用来等待一些列的Goroutines 完成执行，程序会阻塞知道所有的Goroutines均执行完成。

```go
package main

import (  
    "fmt"
    "sync"
    "time"
)

func process(i int, wg *sync.WaitGroup) {  
    fmt.Println("started Goroutine ", i)
    time.Sleep(2 * time.Second)
    fmt.Printf("Goroutine %d ended\n", i)
    wg.Done()
}

func main() {  
    no := 3
    var wg sync.WaitGroup
    for i := 0; i < no; i++ {
        wg.Add(1)
        go process(i, &wg)
    }
    wg.Wait()
    fmt.Println("All go routines finished executing")
}
```

缓存通道的一个重要实现是**工作池**。

通常，工作池是一个等待分配任务的线程集合。一旦他们完成了分配的任务，他们就可以再次执行下一个任务。

```go
package main

import (  
    "fmt"
    "math/rand"
    "sync"
    "time"
)

type Job struct {  
    id       int
    randomno int
}
type Result struct {  
    job         Job
    sumofdigits int
}

var jobs = make(chan Job, 10)  
var results = make(chan Result, 10)

func digits(number int) int {  
    sum := 0
    no := number
    for no != 0 {
        digit := no % 10
        sum += digit
        no /= 10
    }
    time.Sleep(2 * time.Second)
    return sum
}
func worker(wg *sync.WaitGroup) {  
    for job := range jobs {
        output := Result{job, digits(job.randomno)}
        results <- output
    }
    wg.Done()
}
func createWorkerPool(noOfWorkers int) {  
    var wg sync.WaitGroup
    for i := 0; i < noOfWorkers; i++ {
        wg.Add(1)
        go worker(&wg)
    }
    wg.Wait()
    close(results)
}
func allocate(noOfJobs int) {  
    for i := 0; i < noOfJobs; i++ {
        randomno := rand.Intn(999)
        job := Job{i, randomno}
        jobs <- job
    }
    close(jobs)
}
func result(done chan bool) {  
    for result := range results {
        fmt.Printf("Job id %d, input random no %d , sum of digits %d\n", result.job.id, result.job.randomno, result.sumofdigits)
    }
    done <- true
}
func main() {  
    startTime := time.Now()
    noOfJobs := 100
    go allocate(noOfJobs)
    done := make(chan bool)
    go result(done)
    noOfWorkers := 10
    createWorkerPool(noOfWorkers)
    <-done
    endTime := time.Now()
    diff := endTime.Sub(startTime)
    fmt.Println("total time taken ", diff.Seconds(), "seconds")
}
```

#### select

`select` 语法用来从多个channel中选择操作。select语法会阻塞直到其中一个case被执行，同switch类似

```go
package main

import (  
    "fmt"
    "time"
)

func process(ch chan string) {  
    time.Sleep(10500 * time.Millisecond)
    ch <- "process successful"
}

func main() {  
    ch := make(chan string)
    go process(ch)
    for {
        time.Sleep(1000 * time.Millisecond)
        select {
        case v := <-ch:
            fmt.Println("received value: ", v)
            return
        default:
            fmt.Println("no value received")
        }
    }

}
//output 
no value received  
no value received  
no value received  
no value received  
no value received  
no value received  
no value received  
no value received  
no value received  
no value received  
received value:  process successful  
```

死锁和default case。以下例子中，如果不添加default，会造成死锁

```go
package main

import "fmt"

func main() {  
    ch := make(chan string)
    select {
    case <-ch:
    default:
        fmt.Println("default case executed")
    }
}
//output
default case executed  
```

#### 互斥

`Mutex` 用来锁定资源防止多个Goroutine同时访问。在sync包里

```go
mutex.Lock()  
x = x + 1  
mutex.Unlock()  
```

+ 使用mutex解决资源冲突的例子

```go
package main  
import (  
    "fmt"
    "sync"
    )
var x  = 0  
func increment(wg *sync.WaitGroup, m *sync.Mutex) {  
    m.Lock()
    x = x + 1
    m.Unlock()
    wg.Done()   
}
func main() {  
    var w sync.WaitGroup
    var m sync.Mutex
    for i := 0; i < 1000; i++ {
        w.Add(1)        
        go increment(&w, &m)
    }
    w.Wait()
    fmt.Println("final value of x", x)
}
```

+ 使用channel解决资源冲突的例子。利用channel的阻塞

```go
package main  
import (  
    "fmt"
    "sync"
    )
var x  = 0  
func increment(wg *sync.WaitGroup, ch chan bool) {  
    ch <- true
    x = x + 1
    <- ch
    wg.Done()   
}
func main() {  
    var w sync.WaitGroup
    ch := make(chan bool, 1)
    for i := 0; i < 1000; i++ {
        w.Add(1)        
        go increment(&w, ch)
    }
    w.Wait()
    fmt.Println("final value of x", x)
}
```



### OOP

#### 结构体 VS Class

go不提供class，但是提供了结构体。方法可以添加给结构体可以实现数据绑定方法这种class类似的行为。

使用`New()`函数来初始化结构体

```go
package employee

import (  
    "fmt"
)

type employee struct {  
    firstName   string
    lastName    string
    totalLeaves int
    leavesTaken int
}

func New(firstName string, lastName string, totalLeave int, leavesTaken int) employee {  
    e := employee {firstName, lastName, totalLeave, leavesTaken}
    return e
}

func (e employee) LeavesRemaining() {  
    fmt.Printf("%s %s has %d leaves remaining", e.firstName, e.lastName, (e.totalLeaves - e.leavesTaken))
}
```

#### 组合 VS 接口

go不支持接口，但支持组合，支持嵌套结构体的提升。

```go
package main

import (  
    "fmt"
)

type author struct {  
    firstName string
    lastName  string
    bio       string
}

func (a author) fullName() string {  
    return fmt.Sprintf("%s %s", a.firstName, a.lastName)
}

type post struct {  
    title   string
    content string
    author
}

func (p post) details() {  
    fmt.Println("Title: ", p.title)
    fmt.Println("Content: ", p.content)
    fmt.Println("Author: ", p.fullName())
    fmt.Println("Bio: ", p.bio)
}

type website struct {  
 posts []post
}
func (w website) contents() {  
    fmt.Println("Contents of Website\n")
    for _, v := range w.posts {
        v.details()
        fmt.Println()
    }
}

func main() {  
    author1 := author{
        "Naveen",
        "Ramanathan",
        "Golang Enthusiast",
    }
    post1 := post{
        "Inheritance in Go",
        "Go supports composition instead of inheritance",
        author1,
    }
    post2 := post{
        "Struct instead of Classes in Go",
        "Go does not support classes but methods can be added to structs",
        author1,
    }
    post3 := post{
        "Concurrency",
        "Go is a concurrent language and not a parallel one",
        author1,
    }
    w := website{
        posts: []post{post1, post2, post3},
    }
    w.contents()
}
```

#### 多态

go语言的多态借助于接口来实现

```go
package main

import (  
    "fmt"
)

type Income interface {  
    calculate() int
    source() string
}

type FixedBilling struct {  
    projectName string
    biddedAmount int
}

type TimeAndMaterial struct {  
    projectName string
    noOfHours  int
    hourlyRate int
}

func (fb FixedBilling) calculate() int {  
    return fb.biddedAmount
}

func (fb FixedBilling) source() string {  
    return fb.projectName
}

func (tm TimeAndMaterial) calculate() int {  
    return tm.noOfHours * tm.hourlyRate
}

func (tm TimeAndMaterial) source() string {  
    return tm.projectName
}

func calculateNetIncome(ic []Income) {  
    var netincome int = 0
    for _, income := range ic {
        fmt.Printf("Income From %s = $%d\n", income.source(), income.calculate())
        netincome += income.calculate()
    }
    fmt.Printf("Net income of organisation = $%d", netincome)
}

func main() {  
    project1 := FixedBilling{projectName: "Project 1", biddedAmount: 5000}
    project2 := FixedBilling{projectName: "Project 2", biddedAmount: 10000}
    project3 := TimeAndMaterial{projectName: "Project 3", noOfHours: 160, hourlyRate: 25}
    incomeStreams := []Income{project1, project2, project3}
    calculateNetIncome(incomeStreams)
}
```

### 高级

#### Defer

`defer` 语法用于在函数结束前执行一个函数或方法。

```go
package main

import (  
    "fmt"
)

func finished() {  
    fmt.Println("Finished finding largest")
}

func largest(nums []int) {  
    defer finished()    
    fmt.Println("Started finding largest")
    max := nums[0]
    for _, v := range nums {
        if v > max {
            max = v
        }
    }
    fmt.Println("Largest number in", nums, "is", max)
}

func main() {  
    nums := []int{78, 109, 2, 563, 300}
    largest(nums)
}
```

```bash
Started finding largest  
Largest number in [78 109 2 563 300] is 563  
Finished finding largest
```

**defer 函数的参数实在执行defer时求值，而不是实际函数结束时求值**

```go
package main

import (  
    "fmt"
)

func printA(a int) {  
    fmt.Println("value of a in deferred function", a)
}
func main() {  
    a := 5
    defer printA(a)
    a = 10
    fmt.Println("value of a before deferred function call", a)

}

//output
value of a before deferred function call 10  
value of a in deferred function 5  
```

**函数存在多个defer调用时，他们被加入了一个栈。后进先出（LIFO）次序**

```go
package main

import (  
    "fmt"
)

func main() {  
    name := "Naveen"
    fmt.Printf("Original String: %s\n", string(name))
    fmt.Printf("Reversed String: ")
    for _, v := range []rune(name) {
        defer fmt.Printf("%c", v)
    }
}
```



#### 错误处理

error是一个接口类型，定义如下

```go
type error interface {  
    Error() string
}
```

通过断言，根据错误类别字段获取错误信息

```go
package main

import (  
    "fmt"
    "os"
)

func main() {  
    f, err := os.Open("/test.txt")
    if err, ok := err.(*os.PathError); ok {
        fmt.Println("File at path", err.Path, "failed to open")
        return
    }
    fmt.Println(f.Name(), "opened successfully")
}
```

通过断言，根据错误类别方法获取错误信息

```go
package main

import (  
    "fmt"
    "net"
)

func main() {  
    addr, err := net.LookupHost("golangbot123.com")
    if err, ok := err.(*net.DNSError); ok {
        if err.Timeout() {
            fmt.Println("operation timed out")
        } else if err.Temporary() {
            fmt.Println("temporary error")
        } else {
            fmt.Println("generic error: ", err)
        }
        return
    }
    fmt.Println(addr)
}
```

#### 自定义错误

使用New函数创建自定义错误

```go
  package errors

  // New returns an error that formats as the given text.
  func New(text string) error {
      return &errorString{text}
  }

  // errorString is a trivial implementation of error.
  type errorString struct {
      s string
  }
  //error 接口实现
  func (e *errorString) Error() string {
      return e.s
  }
```

使用结构体方法提供额外信息

```go
package main

import "fmt"

type areaError struct {  
    err    string  //error description
    length float64 //length which caused the error
    width  float64 //width which caused the error
}

func (e *areaError) Error() string {  
    return e.err
}

func (e *areaError) lengthNegative() bool {  
    return e.length < 0
}

func (e *areaError) widthNegative() bool {  
    return e.width < 0
}

func rectArea(length, width float64) (float64, error) {  
    err := ""
    if length < 0 {
        err += "length is less than zero"
    }
    if width < 0 {
        if err == "" {
            err = "width is less than zero"
        } else {
            err += ", width is less than zero"
        }
    }
    if err != "" {
        return 0, &areaError{err, length, width}
    }
    return length * width, nil
}
```

#### Panic & Recover

通常情况下，在go语言中处理异常情况使用error。但是某些情况下出现异常后程序无法正常运行，此时使用`panic`。当遇到panic的时候，程序停止执行，defer 函数执行，返回至调用者；打印panic信息；堆栈信息。

```go
func panic(interface{})  
```

```go
package main

import (  
    "fmt"
)

func fullName(firstName *string, lastName *string) {  
    defer fmt.Println("deferred call in fullName")
    if firstName == nil {
        panic("runtime error: first name cannot be nil")
    }
    if lastName == nil {
        panic("runtime error: last name cannot be nil")
    }
    fmt.Printf("%s %s\n", *firstName, *lastName)
    fmt.Println("returned normally from fullName")
}

func main() {  
    defer fmt.Println("deferred call in main")
    firstName := "Elon"
    fullName(&firstName, nil)
    fmt.Println("returned normally from main")
}

//output
deferred call in fullName  
deferred call in main  
panic: runtime error: last name cannot be nil

goroutine 1 [running]:  
main.fullName(0x1042bf90, 0x0)  
    /tmp/sandbox060731990/main.go:13 +0x280
main.main()  
    /tmp/sandbox060731990/main.go:22 +0xc0
```

`Recover` 恢复是针对 goroutine的panic

```go
func recover() interface{}  
```

**Recover 仅在deferred 函数中调用时有效，在deferred函数中调用recover()会停止panic**

```go
package main

import (  
    "fmt"
)

func recoverName() {  
    if r := recover(); r!= nil {
        fmt.Println("recovered from ", r)
    }
}

func fullName(firstName *string, lastName *string) {  
    defer recoverName()
    if firstName == nil {
        panic("runtime error: first name cannot be nil")
    }
    if lastName == nil {
        panic("runtime error: last name cannot be nil")
    }
    fmt.Printf("%s %s\n", *firstName, *lastName)
    fmt.Println("returned normally from fullName")
}

func main() {  
    defer fmt.Println("deferred call in main")
    firstName := "Elon"
    fullName(&firstName, nil)
    fmt.Println("returned normally from main")
}

//output
recovered from  runtime error: last name cannot be nil  
returned normally from main  
deferred call in main  
```

**recover 只在调用它的goroutine中起作用**

```go
package main

import (  
    "fmt"
    "time"
)

func recovery() {  
    if r := recover(); r != nil {
        fmt.Println("recovered:", r)
    }
}

func a() {  
    defer recovery()
    fmt.Println("Inside A")
    go b()
    time.Sleep(1 * time.Second)
}

func b() {  
    fmt.Println("Inside B")
    panic("oh! B panicked")
}

func main() {  
    a()
    fmt.Println("normally returned from main")
}

//output
Inside A  
Inside B  
panic: oh! B panicked

goroutine 5 [running]:  
main.b()  
    /tmp/sandbox388039916/main.go:23 +0x80
created by main.a  
    /tmp/sandbox388039916/main.go:17 +0xc0
```

**通过Debug，在recover中获取堆栈信息**

```go
import (  
    "fmt"
    "runtime/debug"
)
func r() {  
    if r := recover(); r != nil {
        fmt.Println("Recovered", r)
        debug.PrintStack()
    }
}
```



#### 第一类函数

Go中的函数可以作为变量，参数，和函数的返回值使用

匿名函数

```go
package main

import (  
    "fmt"
)

func main() {  
    a := func() {
        fmt.Println("hello world first class function")
    }
    a()
    fmt.Printf("%T", a)
  
    func() {
        fmt.Println("hello world first class function")
    }()
  
    func(n string) {
        fmt.Println("Welcome", n)
    }("Gophers")
}
```

用户定义函数类型

```go
type add func(a int, b int) int  
```

```go
package main

import (  
    "fmt"
)

type add func(a int, b int) int

func main() {  
    var a add = func(a int, b int) int {
        return a + b
    }
    s := a(5, 6)
    fmt.Println("Sum", s)
}
```

函数作为参数

```go
package main

import (  
    "fmt"
)

func simple(a func(a, b int) int) {  
    fmt.Println(a(60, 7))
}

func main() {  
    f := func(a, b int) int {
        return a + b
    }
    simple(f)
}
```

函数作为返回值

```go
package main

import (  
    "fmt"
)

func simple() func(a, b int) int {  
    f := func(a, b int) int {
        return a + b
    }
    return f
}

func main() {  
    s := simple()
    fmt.Println(s(60, 7))
}
```

**闭包**，捕获上下文变量

```go
package main

import (  
    "fmt"
)

func main() {  
    a := 5
    func() {
        fmt.Println("a =", a)
    }()
}
```

#### 反射

反射这块介绍几个`reflect`包常用的方法。不做太深入的介绍

```go
package main

import (  
    "fmt"
    "reflect"
)

type order struct {  
    ordId      int
    customerId int
}


func main() {  
    o := order{
        ordId:      456,
        customerId: 56,
    }
    
    t := reflect.TypeOf(o) // main.order 
    k := t.Kind() // struct
    v := reflect.ValueOf(o) // {456 56} 
    num := v.NumField() // 2
    for i := 0; i < v.NumField(); i++ {
         fmt.Printf("Field:%d type:%T value:%v\n", i, v.Field(i), v.Field(i))
    }//Field:0 type:reflect.Value value:456  
		 //Field:1 type:reflect.Value value:56  

}
```



### 文件操作

#### 文件读

##### 直接读到内存中

```go
package main

import (  
    "flag"
    "fmt"
    "io/ioutil"
)

func main() {  
    //data, err := ioutil.ReadFile("test.txt")
    fptr := flag.String("fpath", "test.txt", "file path to read from")
    flag.Parse()
    data, err := ioutil.ReadFile(*fptr)
    if err != nil {
        fmt.Println("File reading error", err)
        return
    }
    fmt.Println("Contents of file:", string(data))
}

```

##### 分块读取

```go
package main

import (  
    "bufio"
    "flag"
    "fmt"
    "log"
    "os"
)

func main() {  
    fptr := flag.String("fpath", "test.txt", "file path to read from")
    flag.Parse()

    f, err := os.Open(*fptr)
    if err != nil {
        log.Fatal(err)
    }
    defer func() {
        if err = f.Close(); err != nil {
            log.Fatal(err)
        }
    }()
    r := bufio.NewReader(f)
    b := make([]byte, 3)
    for {
        n, err := r.Read(b)
        if err != nil {
            fmt.Println("Error reading file:", err)
            break
        }
        fmt.Println(string(b[0:n]))
    }
}
```

##### 一行一行的读

```go
package main

import (  
    "bufio"
    "flag"
    "fmt"
    "log"
    "os"
)

func main() {  
    fptr := flag.String("fpath", "test.txt", "file path to read from")
    flag.Parse()

    f, err := os.Open(*fptr)
    if err != nil {
        log.Fatal(err)
    }
    defer func() {
        if err = f.Close(); err != nil {
        log.Fatal(err)
    }
    }()
    s := bufio.NewScanner(f)
    for s.Scan() {
        fmt.Println(s.Text())
    }
    err = s.Err()
    if err != nil {
        log.Fatal(err)
    }
}
```

#### 写文件

##### 字符串写入

```go
package main

import (  
    "fmt"
    "os"
)

func main() {  
    f, err := os.Create("test.txt")
    if err != nil {
        fmt.Println(err)
        return
    }
    l, err := f.WriteString("Hello World")
    if err != nil {
        fmt.Println(err)
        f.Close()
        return
    }
    fmt.Println(l, "bytes written successfully")
    err = f.Close()
    if err != nil {
        fmt.Println(err)
        return
    }
}
```

##### 字节写入

```go
package main

import (  
    "fmt"
    "os"
)

func main() {  
    f, err := os.Create("/home/naveen/bytes")
    if err != nil {
        fmt.Println(err)
        return
    }
    d2 := []byte{104, 101, 108, 108, 111, 32, 119, 111, 114, 108, 100}
    n2, err := f.Write(d2)
    if err != nil {
        fmt.Println(err)
        f.Close()
        return
    }
    fmt.Println(n2, "bytes written successfully")
    err = f.Close()
    if err != nil {
        fmt.Println(err)
        return
    }
}
```

##### 字符串一行一行的写

```go
package main

import (  
    "fmt"
    "os"
)

func main() {  
    f, err := os.Create("lines")
    if err != nil {
        fmt.Println(err)
                f.Close()
        return
    }
    d := []string{"Welcome to the world of Go1.", "Go is a compiled language.", "It is easy to learn Go."}

    for _, v := range d {
        fmt.Fprintln(f, v)
        if err != nil {
            fmt.Println(err)
            return
        }
    }
    err = f.Close()
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Println("file written successfully")
}
```

##### 文件追加，以追加和只写的形式

```go
package main

import (  
    "fmt"
    "os"
)

func main() {  
    f, err := os.OpenFile("lines", os.O_APPEND|os.O_WRONLY, 0644)
    if err != nil {
        fmt.Println(err)
        return
    }
    newLine := "File handling is easy."
    _, err = fmt.Fprintln(f, newLine)
    if err != nil {
        fmt.Println(err)
                f.Close()
        return
    }
    err = f.Close()
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Println("file appended successfully")
}
```

##### 最后例子，并发写入

```go
package main

import (  
    "fmt"
    "math/rand"
    "os"
    "sync"
)

func produce(data chan int, wg *sync.WaitGroup) {  
    n := rand.Intn(999)
    data <- n
    wg.Done()
}

func consume(data chan int, done chan bool) {  
    f, err := os.Create("concurrent")
    if err != nil {
        fmt.Println(err)
        return
    }
    for d := range data {
        _, err = fmt.Fprintln(f, d)
        if err != nil {
            fmt.Println(err)
            f.Close()
            done <- false
            return
        }
    }
    err = f.Close()
    if err != nil {
        fmt.Println(err)
        done <- false
        return
    }
    done <- true
}

func main() {  
    data := make(chan int)
    done := make(chan bool)
    wg := sync.WaitGroup{}
    for i := 0; i < 100; i++ {
        wg.Add(1)
        go produce(data, &wg)
    }
    go consume(data, done)
    go func() {
        wg.Wait()
        close(data)
    }()
    d := <-done
    if d == true {
        fmt.Println("File written successfully")
    } else {
        fmt.Println("File writing failed")
    }
}
```





































