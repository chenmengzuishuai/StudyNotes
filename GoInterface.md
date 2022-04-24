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
这段代码中，既有值实现接口类型，又有引用实现接口类型，在实际运行的时候，将`kitty`变量赋值给`runThing`是可以的，并且可以调用其本身的`run()`方法。

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
