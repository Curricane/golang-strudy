# channel并发模式
几个实用的通过done机制防goroutine泄露
## or-done channel
当操作来自系统不同部分的channel，当使用done机制取消channel操作时，如果我们直接`for c := range otherChan`，由于我们不能控制`otherChan`，可能会出现`otherChan`一直阻塞，通道却不关闭，从而出现`goroutine泄漏`。因为我们可以在封装下这个`otherChan`，让支持`or-done`，也支持`range`操作。
- 封装的方式为：封装为一个orDone函数
> `func(done, c <-chan interface{}) <-chan interface{}`
```golang
orDone := func(done, c <-chan interface{}) <-chan interface{} {
    valStream := make(chan interface{})
    go func() {
        defer close(valStream)
        for {
            select {
            // 这个case 实现我们可以自己通过or-done机制控制chan是否继续for循环
            case <-done:
                return
            // 将旧chan的值，给到新chan中，或取消
            case v, ok := <-c:
                if ok == false { // 旧chan被关闭，退出gorutine，触发defer关闭新chan
                    return
                }
                select {
                case valStream <- v:
                case <-done:
                }
            }
        }
    }()
    return valStream // 返回一个新的chan，接受旧chan的值
}

// 封装之后，我们就可以这样使用，这样不仅可以一直接受myChan，还能自己通过done机制控制
for val := range orDone(done, myChan) {
    // do something
}
```