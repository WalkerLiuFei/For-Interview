[参考](https://github.com/teh-cmc/go-internals/blob/master/chapter2_interfaces/README.md)

# Golang method calls

..4 different kinds of functions..:

> - top-level func
> - method with value receiver
> - method with pointer receiver
> - func literal

..and 5 different kinds of calls:

> - direct call of top-level func (`func TopLevel(x int) {}`)
> - direct call of method with value receiver (`func (Value) M(int) {}`)
> - direct call of method with pointer receiver (`func (*Pointer) M(int) {}`)
> - indirect call of method on interface (`type Interface interface { M(int) }`)
> - indirect call of func value (`var literal = func(x int) {}`)

Mixed together, these make up for 10 possible combinations of function and call types:

> - direct call of top-level func /
> - direct call of method with value receiver /
> - direct call of method with pointer receiver /
> - indirect call of method on interface / containing value with value method
> - indirect call of method on interface / containing pointer with value method
> - indirect call of method on interface / containing pointer with pointer method
> - indirect call of func value / set to top-level func
> - indirect call of func value / set to value method
> - indirect call of func value / set to pointer method
> - indirect call of func value / set to func literal

```go

//go:noinline
func Add(a, b int32) int32 { return a + b }

type Adder struct{ id int32 }
//go:noinline
func (adder *Adder) AddPtr(a, b int32) int32 { return a + b }
//go:noinline
func (adder Adder) AddVal(a, b int32) int32 { return a + b }

func main() {
	Add(10, 32) // direct call of top-level function

	adder := Adder{id: 6754}
	adder.AddPtr(10, 32) // direct call of method with pointer receiver
	adder.AddVal(10, 32) // direct call of method with value receiver

	(&adder).AddVal(10, 32) // implicit dereferencing
}
```



```assembly
"".Add STEXT nosplit size=15 args=0x10 locals=0x0
        0x0000 00000 (./main.go:5)      TEXT    "".Add(SB), NOSPLIT|ABIInternal, $0-16
  ;; ...omitted everything but the actual function call...
        0x0000 00000 (./main.go:5)      PCDATA  $0, $0
        0x0000 00000 (./main.go:5)      PCDATA  $1, $0
        0x0000 00000 (./main.go:5)      MOVL    "".b+12(SP), AX
        0x0004 00004 (./main.go:5)      MOVL    "".a+8(SP), CX
        0x0008 00008 (./main.go:5)      ADDL    CX, AX
        0x000a 00010 (./main.go:5)      MOVL    AX, "".~r2+16(SP)
        0x000e 00014 (./main.go:5)      RET
         ;; ...omitted everything but the actual function call...
"".(*Adder).AddPtr STEXT nosplit size=15 args=0x18 locals=0x0
        0x0000 00000 (./main.go:9)      TEXT    "".(*Adder).AddPtr(SB), NOSPLIT|ABIInternal, $0-24
 ;; ...omitted everything but the actual function call...
        0x0000 00000 (./main.go:9)      MOVL    "".b+20(SP), AX
        0x0004 00004 (./main.go:9)      MOVL    "".a+16(SP), CX
        0x0008 00008 (./main.go:9)      ADDL    CX, AX
        0x000a 00010 (./main.go:9)      MOVL    AX, "".~r2+24(SP)
        0x000e 00014 (./main.go:9)      RET
        0x0000 8b 44 24 14 8b 4c 24 10 01 c8 89 44 24 18 c3     .D$..L$....D$..
"".Adder.AddVal STEXT nosplit size=15 args=0x18 locals=0x0
        0x0000 00000 (./main.go:11)     TEXT    "".Adder.AddVal(SB), NOSPLIT|ABIInternal, $0-24
 ;; ...omitted everything but the actual function call...
        0x0000 00000 (./main.go:11)     PCDATA  $1, $0
        0x0000 00000 (./main.go:11)     MOVL    "".b+16(SP), AX
        0x0004 00004 (./main.go:11)     MOVL    "".a+12(SP), CX
        0x0008 00008 (./main.go:11)     ADDL    CX, AX
        0x000a 00010 (./main.go:11)     MOVL    AX, "".~r2+24(SP)
        0x000e 00014 (./main.go:11)     RET
        0x0000 8b 44 24 10 8b 4c 24 0c 01 c8 89 44 24 18 c3     .D$..L$....D$..
"".main STEXT size=171 args=0x0 locals=0x28

;; main.main start here 
		;; 分配40 字节的stack frame
        0x0000 00000 (./main.go:13)     TEXT    "".main(SB), ABIInternal, $40-0
        ;; 下面这三个指令完成 stack 扩展
        0x0000 00000 (./main.go:13)     MOVQ    (TLS), CX
        0x0009 00009 (./main.go:13)     CMPQ    SP, 16(CX)
        ;; ...omitted everything but the actual function call...
        0x000d 00013 (./main.go:13)     JLS     161
        ;; ...omitted everything but the actual function call...
        ;; 下探 40 byte，因为已经分配足够了，不必担心不够用
        0x0013 00019 (./main.go:13)     SUBQ    $40, SP
        ;; 前8字节用来保存 "".main 函数地址，所以函数执行应该在 32(SP)偏移处开始
        0x0017 00023 (./main.go:13)     MOVQ    BP, 32(SP)
        0x001c 00028 (./main.go:13)     LEAQ    32(SP), BP
 		;; ...omitted everything but the actual function call...
 		
 		;; 这四行是用来调用global Add 方法的
        0x0021 00033 (./main.go:14)     MOVQ    $137438953482, AX
        0x002b 00043 (./main.go:14)     MOVQ    AX, (SP)
        0x002f 00047 (./main.go:14)     CALL    "".Add(SB)
        
        ;; 初始化Adder 结构体, "".adder+ 本身没意思，这里的意思是取 SP偏移28个字节的位置
        ;; 因为 第29 - 32着四个字节已经被global add method 的result 占用了
        0x0034 00052 (./main.go:16)     MOVL    $0, "".adder+28(SP)
        0x003c 00060 (./main.go:16)     MOVL    $6754, "".adder+28(SP)
        ;; ...omitted everything but the actual function call...
        ;; 取出 adder 的指针，放入AX 寄存器中
        0x0044 00068 (./main.go:17)     LEAQ    "".adder+28(SP), AX
         ;; ...omitted everything but the actual function call...
        ;; 将上一步拿到的指针放入SP寄存器中，等待执行，注意
        0x0049 00073 (./main.go:17)     MOVQ    AX, (SP)
        ;; 参数初始化其实是  ， 2 和 10 两个 int32 的参数组成的8 byte参数
        0x004d 00077 (./main.go:17)     MOVQ    $137438953482, AX
        ;; 将上面的值放入SP寄存器中，
        0x0057 00087 (./main.go:17)     MOVQ    AX, 8(SP)
        ;; 调用函数, SB 寄存器存储了 AddPtr这个函数的位置
        0x005c 00092 (./main.go:17)     CALL    "".(*Adder).AddPtr(SB)
        ;; "".adder+28(SP) 这个位置存放的是 adder 这个结构体对象，第一个参数是
        ;; adder这个 4个字节的结构体
        0x0061 00097 (./main.go:18)     MOVL    "".adder+28(SP), AX
        0x0065 00101 (./main.go:18)     MOVL    AX, (SP)
        0x0068 00104 (./main.go:18)     MOVQ    $137438953482, AX
        0x0072 00114 (./main.go:18)     MOVQ    AX, 4(SP)
        0x0077 00119 (./main.go:18)     CALL    "".Adder.AddVal(SB)
        ;; "".adder+28(SP) 这个位置存放的是 adder 这个结构体对象，第一个参数是
        ;; adder这个 4个字节的结构体
        0x007c 00124 (./main.go:20)     MOVL    "".adder+28(SP), AX
        0x0080 00128 (./main.go:20)     MOVL    AX, (SP)
        0x0083 00131 (./main.go:20)     MOVQ    $137438953482, AX
        0x008d 00141 (./main.go:20)     MOVQ    AX, 4(SP)
        0x0092 00146 (./main.go:20)     CALL    "".Adder.AddVal(SB)
        0x0097 00151 (./main.go:21)     MOVQ    32(SP), BP
        0x009c 00156 (./main.go:21)     ADDQ    $40, SP
        0x00a0 00160 (./main.go:21)     RET
        0x00a1 00161 (./main.go:21)     NOP
        
      ;; ...omitted everything but the actual function call...
        0x00a1 00161 (./main.go:13)     CALL    runtime.morestack_noctxt(SB)
      ;; ...omitted everything but the actual function call...
        0x00a6 00166 (./main.go:13)     JMP     0

```

### 隐式取消引用

#### case A The receiver is still on the stack 

如果接收器仍在堆栈上，并且其大小足够小，可以按几条指令进行复制（如此处的情况），则编译器只需将其值复制到堆栈的顶部，然后对

if the receiver is still on the stack and its size is sufficient small that can be copied with few instructions as is case here, the compiler just coiped its value to the top of the stack then does straightforward call to.

`(&adder).AddVal(10, 32)` thus looks like this in this situation:

```
0x0074 MOVL	"".adder+28(SP), AX	;; move (i.e. copy) adder (note the MOV instead of a LEA) to..
0x0078 MOVL	AX, (SP)		;; ..the top of the stack (argument #1)
0x007b MOVQ	$137438953482, AX	;; move (32,10) to..
0x0085 MOVQ	AX, 4(SP)		;; ..the top of the stack (arguments #3 & #2)
0x008a CALL	"".Adder.AddVal(SB)
```

#### case B The receiver is allocated on the heap

```assembly
  0x0000 TEXT	"".(*Adder).AddVal(SB), DUPOK|WRAPPER, $32-24
  ;; ...omitted preambles...

  0x0026 MOVQ	""..this+40(SP), AX ;; check whether the receiver..
  0x002b TESTQ	AX, AX		    ;; ..is nil
  0x002e JEQ	92		    ;; if it is, jump to 0x005c (panic)

  0x0030 MOVL	(AX), AX            ;; dereference pointer receiver..
  0x0032 MOVL	AX, (SP)            ;; ..and move (i.e. copy) the resulting value to argument #1

  ;; forward (copy) arguments #2 & #3 then call the wrappee
  0x0035 MOVL	"".a+48(SP), AX
  0x0039 MOVL	AX, 4(SP)
  0x003d MOVL	"".b+52(SP), AX
  0x0041 MOVL	AX, 8(SP)
  0x0045 CALL	"".Adder.AddVal(SB) ;; call the wrapped method

  ;; copy return value from wrapped method then return
  0x004a MOVL	16(SP), AX
  0x004e MOVL	AX, "".~r2+56(SP)
  ;; ...omitted frame-pointer stuff...
  0x005b RET

  ;; throw a panic with a detailed error
  0x005c CALL	runtime.panicwrap(SB)

  ;; ...omitted epilogues...
```



1. receiver func 和 global func 的 调用方式没有明显的不同，唯一的区别是在调用receiver func 时会将 receiver作为第一个参数。

2. 当 object 被转换成 interface 后，object 就发生了escape 到了 heap上面。成了global object 。
3. interface 的类型转换和数据填充分别是在 compile 和 link 时完成的，runtime 没有i处理这些东西，所以对于性能来说是很友好的。
4. 每个Go函数在函数入口处都有一个很小的序言。它检查我们是否已经用完分配的堆栈空间，如果有，则调用morestack函数。具体的实现方式 就是在生成 中间汇编时进行的。（编译器进行）
5. 