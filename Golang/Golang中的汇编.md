

[参考](https://github.com/teh-cmc/go-internals/blob/master/chapter1_assembly_primer/README.md)



## Go’s assembler

1.  The most important thing to know about Go’s assembler is that it’s not not a direct representation of the underlying machine. Some of the details map precisely to the machine, but some do not.



## An example

```go
//go:noinline
func add(a, b int32) (int32, bool) { return a + b, true }

func main() { add(10, 32) }

// compile with this command :
$ GOOS=linux GOARCH=amd64 go tool compile -S direct_topfunc_call.go
```



```assembly
0x0000 TEXT		"".add(SB), NOSPLIT, $0-16
  0x0000 FUNCDATA	$0, gclocals·f207267fbf96a0178e8758c6e3e0ce28(SB)
  0x0000 FUNCDATA	$1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
  0x0000 MOVL		"".b+12(SP), AX
  0x0004 MOVL		"".a+8(SP), CX
  0x0008 ADDL		CX, AX
  0x000a MOVL		AX, "".~r2+16(SP)
  0x000e MOVB		$1, "".~r3+20(SP)
  0x0013 RET

0x0000 TEXT		"".main(SB), $24-0
  ;; ...omitted stack-split prologue...
  0x000f SUBQ		$24, SP
  0x0013 MOVQ		BP, 16(SP)
  0x0018 LEAQ		16(SP), BP
  0x001d FUNCDATA	$0, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
  0x001d FUNCDATA	$1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
  0x001d MOVQ		$137438953482, AX
  0x0027 MOVQ		AX, (SP)
  0x002b PCDATA		$0, $0
  0x002b CALL		"".add(SB)
  0x0030 MOVQ		16(SP), BP
  0x0035 ADDQ		$24, SP
  0x0039 RET
  ;; ...omitted stack-split epilogue...
```



#### the insider

```assembly
0x0000 TEXT "".add(SB), NOSPLIT, $0-16
```

+ 0x0000 : Offset of current instruction, relative to the start of the function

+ `TEXT “”.add`  The `Text` directive declarS the “”.add symbol as part of the `.text` section,(ie. runnable code) and indicates that  the instructions that follow are the body of the function.

  the empty string “” will be replaced by the name of the current package at link-time: i.e “”.add will be replaced to `“main.add”` once linked into the final binary

+ `(SB)` `SB` 是一个虚拟寄存器，其维护一个固定的指针，可以说是程序入口的开始地址。 比如说`“”.add(SB)` 是函数`add` 相对于`SP` 的偏移量。 最终这个偏移量的地址由链接器计算。

所有的user-defined 的标识都需要以类似上面的形式进行标识，

1. All user-defined symbols are written in prseudo-register `FP` (arguments and local varibles), and SB(global). The `SB ` register can be thought of as an original of memory, so the symbol `foo(SB)` indicates the name `foo` as an address in memory.

2. Finally, there are two important things to note here:
   1. The first argument `a` is not located at `0(SP)`, but rather at `8(SP)`; that's because the caller stores its return-address in `0(SP)` via the `CALL` pseudo-instruction.
   2. Arguments are passed in reverse-order; i.e. the first argument is the closest to the top of the stack.
   
3. `NOSPLIT` indicates to the compiler that it shoud not insert the `stack-split` preamble, which checks whether the current stack needs to be grown. 

   In the case of this `add` function, the compiler has set the flag by itself, it smart enough to figure out that: since the function doesn’t have local variables and not stack-frame of its own, it simply cannt outgrown the current stack; thus it’s complete waste of cpu resources to run stack-size check at each call site.

4. \$0-\$16 : `$0` denotes the size in  bytes of the stack-frame that will be allocated.  while the 16 indicates specifies the size of the arguments passed by the caller. ”Same : $24-8 states that the function has a 24 byte frame and is called with 8 bytes of argument,  which live in on the caller’s frame. if the `NOSPLIT` is not specified for the TEXT,the argument size must be provided. “

**`PCDATA` and `FUNCDATA` directives contain information for use by the garbage collector;  introduced by the compiler**

6. `”“.b+12(SP)` and `“".a+8(SP)` respectively refer to the addresses 12 bytes and 8 bytes below the top of the stack(it grow downwards.)
7. `FP` register is preudo-register used to `refer to` the function arguments. The compiler maintain a **virtual frame pointer** and refer to the arguments on the stack as offsets from that `pseudo-register` .**Thus 0(FP) is the first argument to the function, 8(FP) is the second (on a 64-bit machine), and so on.  However, when referring to a function argument this way, it is necessary to place a name at the beginning, as in first_arg+0(FP) and second_arg+8(FP). **

Finally, there are two important things to note here:

1. The first argument `a` is not located at `0(SP)`, but rather at `8(SP)`; that's because **the caller stores its return-address in `0(SP)` via the `CALL` pseudo-instruction**.
2. **Arguments are passed in reverse-order; i.e. the first argument is the closest to the top of the stack.**

```assembly
0x0008 ADDL CX, AX
# THE RESULT IS MOVED OVER TO "".~r2+16(SP)
0x000a MOVL AX, "".~r2+16(SP)
# THE SECOND RETURN VALUE, THE BOOL TYPE VALUE
0x000e MOVB $1, "".~r3+20(SP)
```

**the symbol `”".~r20` has no semantic meaning**





That's a lot of syntax and semantics to ingest all at once. Here's a quick inlined summary of what we've just covered:

```
;; Declare global function symbol "".add (actually main.add once linked)
;; Do not insert stack-split preamble
;; 0 bytes of stack-frame, 16 bytes of arguments passed in
;; func add(a, b int32) (int32, bool)
0x0000 TEXT	"".add(SB), NOSPLIT, $0-16
  ;; ...omitted FUNCDATA stuff...
  0x0000 MOVL	"".b+12(SP), AX	    ;; move second Long-word (4B) argument from caller's stack-frame into AX
  0x0004 MOVL	"".a+8(SP), CX	    ;; move first Long-word (4B) argument from caller's stack-frame into CX
  0x0008 ADDL	CX, AX		    ;; compute AX=CX+AX
  0x000a MOVL	AX, "".~r2+16(SP)   ;; move addition result (AX) into caller's stack-frame
  0x000e MOVB	$1, "".~r3+20(SP)   ;; move `true` boolean (constant) into caller's stack-frame
  0x0013 RET			    ;; jump to return address stored at 0(SP)
```

All in all, here's a visual representation of what the stack looks like when `main.add` has finished executing:

```
   |    +-------------------------+ <-- 32(SP)              
   |    |                         |                         
 G |    |                         |                         
 R |    |                         |                         
 O |    | main.main's saved       |                         
 W |    |     frame-pointer (BP)  |                         
 S |    |-------------------------| <-- 24(SP)              
   |    |      [alignment]        |                         
 D |    | "".~r3 (bool) = 1/true  | <-- 21(SP)              
 O |    |-------------------------| <-- 20(SP)              
 W |    |                         |                         
 N |    | "".~r2 (int32) = 42     |                         
 W |    |-------------------------| <-- 16(SP)              
 A |    |                         |                         
 R |    | "".b (int32) = 32       |                         
 D |    |-------------------------| <-- 12(SP)              
 S |    |                         |                         
   |    | "".a (int32) = 10       |                         
   |    |-------------------------| <-- 8(SP)               
   |    |                         |                         
   |    |                         |                         
   |    |                         |                         
 \ | /  | return address to       |                         
  \|/   |     main.main + 0x30    |                         
   -    +-------------------------+ <-- 0(SP) (TOP OF STACK)
```

## The Main

```assembly
# 24 bytes stack-frame and does't received any argumnents 
0x0000 TEXT		"".main(SB), $24-0
  ;; ...omitted stack-split prologue...
  0x000f SUBQ		$24, SP # GROWN STACK FRAME by 24 bytes(ALLOCATED,as mentioned)
  0x0013 MOVQ		BP, 16(SP)  # store the current frame-pointer 
  0x0018 LEAQ		16(SP), BP # computes the new address of the frame ponter
  ;; ...omitted FUNCDATA stuff...
  # 137438953482 actually corresponds to the 10 and 32 4-byte values concatenated into #one 8-byte value, as we pass into `add` function's arguments 
  0x001d MOVQ		$137438953482, AX   
  0x0027 MOVQ		AX, (SP)
  ;; ...omitted PCDATA stuff...
  # call the add functioon 
  0x002b CALL		"".add(SB) 
  0x0030 MOVQ		16(SP), BP
  0x0035 ADDQ		$24, SP
  0x0039 RET
  ;; ...omitted stack-split epilogue...
```



- 8 bytes (`16(SP)`-`24(SP)`) are used to store the **current value of the frame-pointer `BP`  (*the real one!)** to allow for stack-unwinding and facilitate debugging

- 1+3 bytes (`12(SP)`-`16(SP)`) are reserved for the second return value (`bool`) plus 3 bytes of necessary alignment on `amd64`

- 4 bytes (`8(SP)`-`12(SP)`) are reserved for the first return value (`int32`)

- 4 bytes (`4(SP)`-`8(SP)`) are reserved for the value of argument `b (int32)`

- 4 bytes (`0(SP)`-`4(SP)`) are reserved for the value of argument `a (int32)`

  

1. `TLS` is a virtual register maintained by the runtime that holds a pointer to the current `g`, i.e. the data-structure that keeps track of all the state of a goroutine.

Finally, we:

1. Unwind the frame-pointer by one stack-frame (i.e. we "go down" one level)
2. Shrink the stack by 24 bytes to reclaim the stack space we had previously allocated
3. Ask the Go assembler to insert subroutine-return related stuff

## A word about goroutines, stacks and splits

### Splits

For stack-splitting to work, the compiler inserts a few instructions at the beginning and end of every function that could potentially overflow its stack.
As we've seen earlier in this chapter, and to avoid unnecessary overhead, functions that cannot possibly outgrow their stack are marked as `NOSPLIT` as a hint for the compiler not to insert these checks.

Let's look at our main function from earlier, this time without omitting the stack-split preamble:

```assembly
0x0000 TEXT	"".main(SB), $24-0
  ;; stack-split prologue
  0x0000 MOVQ	(TLS), CX
  0x0009 CMPQ	SP, 16(CX)
  0x000d JLS	58

  0x000f SUBQ	$24, SP
  0x0013 MOVQ	BP, 16(SP)
  0x0018 LEAQ	16(SP), BP
  ;; ...omitted FUNCDATA stuff...
  0x001d MOVQ	$137438953482, AX
  0x0027 MOVQ	AX, (SP)
  ;; ...omitted PCDATA stuff...
  0x002b CALL	"".add(SB)
  0x0030 MOVQ	16(SP), BP
  0x0035 ADDQ	$24, SP
  0x0039 RET

  ;; stack-split epilogue
  0x003a NOP
  ;; ...omitted PCDATA stuff...
  0x003a CALL	runtime.morestack_noctxt(SB)
  0x003f JMP	0
```

As you can see, the stack-split preamble is divided into a prologue and an epilogue:

- The prologue checks whether the goroutine is running out of space and, if it's the case, jumps to the epilogue.
- The epilogue, on the other hand, triggers the stack-growth machinery and then jumps back to the prologue.

This creates a feedback loop that goes on for as long as a large enough stack hasn't been allocated for our starved goroutine.

**Prologue**

```
0x0000 MOVQ	(TLS), CX   ;; store current *g in CX
0x0009 CMPQ	SP, 16(CX)  ;; compare SP and g.stackguard0
0x000d JLS	58	    ;; jumps to 0x3a if SP <= g.stackguard0
```

`TLS` is a virtual register maintained by the runtime that holds a pointer to the current `g`, i.e. the data-structure that keeps track of all the state of a goroutine.

Looking at the definition of `g` from the source code of the runtime:

```go
type g struct {
	stack       stack   // 16 bytes
	// stackguard0 is the stack pointer compared in the Go stack growth prologue.
	// It is stack.lo+StackGuard normally, but can be StackPreempt to trigger a preemption.
	stackguard0 uintptr
	stackguard1 uintptr

	// ...omitted dozens of fields...
}
```

We can see that `16(CX)` corresponds to `g.stackguard0`, which is the threshold value maintained by the runtime that, when compared to the stack-pointer, indicates whether or not a goroutine is about to run out of space.
The prologue thus checks if the current `SP` value is less than or equal to the `stackguard0` threshold (that is, it's bigger), then jumps to the epilogue if it happens to be the case.

**Epilogue**

```
0x003a NOP
0x003a CALL	runtime.morestack_noctxt(SB)
0x003f JMP	0
```

The body of the epilogue is pretty straightforward: it calls into the runtime, which will do the actual work of growing the stack, then jumps back to the first instruction of the function (i.e. to the prologue).

The `NOP` instruction just before the `CALL` exists so that the prologue doesn't jump directly onto a `CALL` instruction. On some platforms, doing so can lead to very dark places; it's a common pratice to set-up a noop instruction right before the actual call and land on this `NOP` instead.
*[UPDATE: We've discussed about this matter in [issue #4: Clarify "nop before call" paragraph](https://github.com/teh-cmc/go-internals/issues/4).]*