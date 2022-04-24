# go 语言中值类型实现接口和引用类型实现接口

```
ppackage main

import (
	"fmt"
)

type cat struct {
	name      string
	footCount uint8
}

func (c cat) run() {
	fmt.Printf("Cat is running with its %d foots\n", c.footCount)
}

type dog struct {
	name      string
	footCount uint8
}

func (d *dog) run() {

	fmt.Printf("Dog is running with its %d foots\n", d.footCount)
}

type runner interface {
	run()
}

func main() {
	var kitty cat = cat{"kitty", 4}
	var runThing runner = kitty
	runThing.run()

	var dobby dog = dog{"Dobby", 4}
	var runThing1 runner = dobby
	runThing1.run()

}
```
这段代码中，既有值实现接口类型，又有引用实现接口类型，实际运行时，将`kitty`变量赋值给`runThing`是可以正确运行的，并且可以调用其本身的`run()`方法。

但在将`dobby`变量赋值给`runThing1`时报错：

```

cannot use dobby (type dog) as type runner in assignment:
        dog does not implement runner (run method has pointer receiver)

```

原因是结构体`dog`在实现`run()`接口的时候使用的是引用传递，**所以此时实现此接口的类型其实是`*dog`类型,**，我们把代码改造如下：
```
var dobby dog = dog{"Dobby", 4}
	var runThing1 runner = &dobby
	runThing1.run()
```
意思是取变量`dobby`变量的地址（指针类型）赋值给`runThing1`，因为实现`run()`方法的就是结构体`dog`的指针类型，所以此时可以编译运行成功。

## 总结延申
所有go教程中关于接口总是强调一句话：牢记interface是一种变量类型。
既然是结构体，那么必然有字段和字段类型。所以在接口中，方法可以认为是一种字段，其返回值可以认为是字段类型。
那么在常规的结构体中，可以把函数作为字段吗？说干就干：
```
type mouse struct{
	name string
	run() 
}

func main(){
    var Jerry mouse = mouse{"Jerry",func(){
		fmt.Println("Mouse is running")
	}}

	Jerry.run()
}
```
运行发现报错：
```
 syntax error: unexpected ), expecting type
```
说明在常规的结构体中是不可以把函数作为字段来使用的（这不是废话吗，要不然设计者为啥要设计一个interface作为一个特殊的结构体来声明）。退一步说，go语言中本身也没有类似`this`、`self`之类的关键字来指向实例本身。所以go语言要在结构体声明之外按照`func(param struct) name (param Type) returnType`的方法来实现某种方法。
