# golang-strudy
## fmt
### 基础的格式化输出
- %t 是格式化输出
- %v 相应值的默认格式,是万能占位符，可以在不知道变量是什么类型的时候使用。
    - %#v 相应值的Go语法表示
    - %+v 打印结构体时，会添加字段名
- %T 打印类型
- %s 字符串
- %p 指针
### fmt 从终端获取用户的输入
- `fmt.Scanf(format string, a… interface{})` 格式话输入，空格作为分隔符，`,`占位符，使用时，最好以`\n`结尾，不然会读入`\n`给下一个`Scanf`如，`fmt.Scanf("%b\n", &b)`
- `fmt.Scan(a… interface{})`: 从终端获取用户输入，存储在`Scan`中的参数里，空格和换行符作为分隔符
- `fmt.Scanln(a… interface{})`: 从终端获取用户输入，存储在`Scanln`中的参数里，空格符作为分隔符，遇到换行符结束
### fmt 从字符串中获取输入
用法如上，只是源换成了字符串，多了一个字符串参数
- fmt.Ssacnf
- fmt.Sscan
- fmt.Sscanln
## 字符串
- 字符串不可修改，想要修改得创建一个新的字符串.可以先
### 字符串长度
>	字符串的底层时byte数组，len()计算字符串长度计算的是byte数组的长度
	而string是utf8编码的，`故字符数不一定等于字符串长度特别是含有中文的时候`。
	rune类型用来表示utf8字符的，一个rune字符由一个或者多个byte组成。
### 转换
- 字符串与字节切片的转换
```golang
var buff  []byte
buff = []byte("hello")
str := string(buff)
```
- 字符串转rune切片
```golang
var runeSlice []rune
runeSlice = []rune(str)
len(runeSlice)  //得到字符数
```
### 字符串拼接
- `strings.Builder`
    - 速度快，内存分配多一点，需要拼接大量字符串优先考虑
- `strings.Join`拼接
    - 比+号连接快一点
- `+`号拼接
    - 便利，简短拼接考虑
- `fmt.Sprintf`拼接
    - 速度最慢
    - 需要格式化拼接时优先考虑
## for range细节
range关键字是Go语言中一个非常有用的迭代array，slice，map, string, channel中元素的内置关键字。
- 使用方式`for xxx := range [range表达式]`
    - 这里用[range表达式]用来方便说明
    - 1 range表达式只会在for语句开始执行时被求值一次，无论后边会有多少次迭代
    - 2 range表达式的求值结果会被复制，也就是说，被迭代的对象是range表达式结果值的副本而不是原值。如果是指针/引用类型，得到的副本是对应的指针副本，否则则是新的拷贝
    > 这样就会造成，range过程中，slice变长时，e得到的值范围不会增加
    > range过程中，数组内部变化，e得到的值不会改变
    - 3 对于channel，range会求值多次，毕竟channel本身就是变化的
    > 就是说，range会在内部保存一个range表达式的副本，它会遍历这个副本，对外是透明的。
```golang
func test1() {
	// 示例2。
	// 值类型，range第一次就确认了e的值，且是拷贝的副本，及e的值
	// 只能是1，2，3，4，5，6
	numbers2 := [...]int{1, 2, 3, 4, 5, 6}
	maxIndex2 := len(numbers2) - 1

	// e 的值已经在for range 开始时，就已经确定为 1 2 3 4 5 6
	// 因为range 的是numbers2的副本，而numbers是数组
	for i, e := range numbers2 {
		if i == maxIndex2 {
			numbers2[0] += e
		} else {
			numbers2[i+1] += e // 改变下一下标的值，但range下次遍历时，e是改变前的值
		}
	}
	fmt.Println(numbers2) // [7 3 5 7 9 11] 即 [1+6, 1+2, 2+3, 3+4, 4+5, 5+6]
	fmt.Println()
}

func test2() {
	// 示例2。
	// 引用类型，e得到的是每个位置的指针/引用，故slic中的值变化，迭代到时
	// e的值相应变化
	numbers2 := []int{1, 2, 3, 4, 5, 6}
	maxIndex2 := len(numbers2) - 1

	// 表达式的值已经在for range 开始时，就已经确定为numbers 切片的副本
	// 因为range 的是numbers2的副本，而numbers是切片，则在里面改变，会影响到源切片
	// 可以理解为，range 会拿到numbers2_副本，我们在外看到的numbers2还是原来哪一个，但在range
	// 实现的内部，它会遍历numbers2_副本，而这个副本对我们是透明的
	for i, e := range numbers2 {
		if i == maxIndex2 {
			numbers2[0] += e
		} else {
			numbers2[i+1] += e // 改变下一下标的值，但range下次遍历时，e是改变前的值
		}
	}
	fmt.Println(numbers2) // [7 3 5 7 9 11] 即 [1+6, 1+2, 2+3, 3+4, 4+5, 5+6]
	fmt.Println()
}
```
## switch中的细节
- 1，switch表达式结果、case表达式结果的类型要一致
    - switch语句会进行有限的类型转换
    - 若是无类型的常量，如4，switch语句会这个常量会被自动地转换为此种常量的默认类型的值，int 4
- 2，不同case间一般不能有重叠，如果有重叠，会按照`先上后下的顺序`，执行上面的case，而不执行下面的case
- 3，switch 用于类型判断，则必须要直接由类型字面量表示，而无法通过间接的方式表示
![switch_细节01](./picture/switch_细节01.png)
图中 uint8是byte的别名，他们的字面量是一样的，因此不能通过编译
