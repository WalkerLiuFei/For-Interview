# Golang中的反射

在讲反射之前，先来看看Golang关于类型设计的一些原则

- 变量包括（type, value）两部分
  - 理解这一点就知道为什么nil != nil了
- type 包括 static type和concrete type. 简单来说 static type是你在编码是看见的类型(如int、string)，concrete type是runtime系统看见的类型
- 类型断言能否成功，取决于变量的concrete type，而不是static type. 因此，一个 reader变量如果它的concrete type也实现了write方法的话，它也可以被类型断言为writer.

接下来要讲的反射，就是建立在类型之上的，Golang的指定类型的变量的类型是静态的（也就是指定int、string这些的变量，它的type是static type），在创建变量的时候就已经确定，反射主要与Golang的interface类型相关（它的type是concrete type），只有interface类型才有反射一说。在Golang的实现中，每个interface变量都有一个对应pair，pair中记录了实际变量的值和类型:

```
(value, type)
```

反射中的reflect.ValueOf() 和reflect.TypeOf() 其实就读取 interface{} 类型中的这两个值。

##  反射的高级用法

这算是一个高级用法了，前面我们只说到对类型、变量的几种反射的用法，包括如何获取其值、其类型、如果重新设置新值。但是在工程应用中，另外一个常用并且属于高级的用法，就是通过reflect来进行方法【函数】的调用。比如我们要做框架工程的时候，需要可以随意扩展方法，或者说用户可以自定义方法，那么我们通过什么手段来扩展让用户能够自定义呢？关键点在于用户的自定义方法是未可知的，因此我们可以通过reflect来搞定

示例如下：

```
package main

import (
	"fmt"
	"reflect"
)

type User struct {
	Id   int
	Name string
	Age  int
}

func (u User) ReflectCallFuncHasArgs(name string, age int) {
	fmt.Println("ReflectCallFuncHasArgs name: ", name, ", age:", age, "and origal User.Name:", u.Name)
}

func (u User) ReflectCallFuncNoArgs() {
	fmt.Println("ReflectCallFuncNoArgs")
}

// 如何通过反射来进行方法的调用？
// 本来可以用u.ReflectCallFuncXXX直接调用的，但是如果要通过反射，那么首先要将方法注册，也就是MethodByName，然后通过反射调动mv.Call

func main() {
	user := User{1, "Allen.Wu", 25}
	
	// 1. 要通过反射来调用起对应的方法，必须要先通过reflect.ValueOf(interface)来获取到reflect.Value，得到“反射类型对象”后才能做下一步处理
	getValue := reflect.ValueOf(user)

	// 一定要指定参数为正确的方法名
	// 2. 先看看带有参数的调用方法
	methodValue := getValue.MethodByName("ReflectCallFuncHasArgs")
	args := []reflect.Value{reflect.ValueOf("wudebao"), reflect.ValueOf(30)}
	methodValue.Call(args)

	// 一定要指定参数为正确的方法名
	// 3. 再看看无参数的调用方法
	methodValue = getValue.MethodByName("ReflectCallFuncNoArgs")
	args = make([]reflect.Value, 0)
	methodValue.Call(args)
}


运行结果：
ReflectCallFuncHasArgs name:  wudebao , age: 30 and origal User.Name: Allen.Wu
ReflectCallFuncNoArgs

复制代码
```

#### 说明

1. 要通过反射来调用起对应的方法，必须要先通过reflect.ValueOf(interface)来获取到reflect.Value，得到“反射类型对象”后才能做下一步处理
2. reflect.Value.MethodByName这.MethodByName，需要指定准确真实的方法名字，如果错误将直接panic，MethodByName返回一个函数值对应的reflect.Value方法的名字。
3. []reflect.Value，这个是最终需要调用的方法的参数，可以没有或者一个或者多个，根据实际参数来定。
4. reflect.Value的 Call 这个方法，这个方法将最终调用真实的方法，参数务必保持一致，如果reflect.Value'Kind不是一个方法，那么将直接panic。
5. 本来可以用u.ReflectCallFuncXXX直接调用的，但是如果要通过反射，那么首先要将方法注册，也就是MethodByName，然后通过反射调用methodValue.Call





反射中的`reflect.ValueOf()` 和`reflect.TypeOf()` 其实就读取 `interface{}` 类型中的这两个值。



Golang reflect慢主要有两个原因

1. 涉及到内存分配以及后续的GC；
2. reflect实现里面有大量的枚举，也就是for循环，比如类型之类的。

## 总结

上述详细说明了Golang的反射reflect的各种功能和用法，都附带有相应的示例，相信能够在工程应用中进行相应实践，总结一下就是：

- 反射可以大大提高程序的灵活性，使得interface{}有更大的发挥余地
  - 反射必须结合interface才玩得转
  - 变量的type要是concrete type的（也就是interface变量）才有反射一说
- 反射可以将“接口类型变量”转换为“反射类型对象”
  - 反射使用 TypeOf 和 ValueOf 函数从接口中获取目标对象信息
- 反射可以将“反射类型对象”转换为“接口类型变量
  - reflect.value.Interface().(已知的类型)
  - 遍历`reflect.Type`的Field获取其Field
- 反射可以修改反射类型对象，但是其值必须是“addressable”
  - 想要利用反射修改对象状态，前提是 interface.data 是 settable,即 pointer-interface
- 通过反射可以“动态”调用方法
- 因为Golang本身不支持模板，因此在以往需要使用模板的场景下往往就需要使用反射(reflect)来实现

