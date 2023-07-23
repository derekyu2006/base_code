原理比较简单，直接上一个终极版例子：
```
package main

import (
     "context"
     "fmt"
     "net/http"
     "sync"
     "time"
     "github.com/gin-gonic/gin"
)

// 一个请求会触发调用三个服务，每个服务输出一个 int,
// 请求要求结果为三个服务输出 int 之和
// 请求返回时间不超过3秒，大于3秒只输出已经获得的 int 之和
func calHandler(c *gin.Context) {
    var resContainer, sum int
    var success, resChan = make(chan int), make(chan int, 3)
    ctx, cancel := context.WithTimeout(c, 5\*time.Second)
    defer cancel()
 
    // 真正的业务逻辑
    go MyLogic(resChan, success)
 
    for {
        select {
        case resContainer = <-resChan:
            sum += resContainer
            fmt.Println("add", resContainer)
        case <- success:
            c.JSON(http.StatusOK, gin.H{"code":200, "result": sum})
            return
        case <- ctx.Done():
            c.JSON(http.StatusOK, gin.H{"code":200, "result": sum})
            return
        }
    }
}

func main() {
    r := gin.New()
    
    r.GET("/calculate", calHandler)
    http.ListenAndServe(":8008", r)
}

func MyLogic(rc chan\<- int, success chan\<- int) {
    wg := sync.WaitGroup{} // 创建一个 waitGroup 组
    
    wg.Add(3) // 我们往组里加3个标识，因为我们要运行3个任务
    
    go func() {
        rc <- microService1()
        wg.Done() // 完成一个，Done()一个
    }()
    
    go func() {
        rc <- microService2()
        wg.Done()
    }()
    
    go func() {
        rc <- microService3()
        wg.Done()
    }()
    
    wg.Wait() // 直到我们前面三个标识都被 Done 了，否则程序一直会阻塞在这里
    
    success <- 1 // 我们发送一个成功信号到通道中
}

func microService1() int {
    time.Sleep(1*time.Second)
    return 1
}

func microService2() int {
    time.Sleep(2*time.Second)
    return 2
}

func microService3() int {
    time.Sleep(6*time.Second)
    return 3
}
```