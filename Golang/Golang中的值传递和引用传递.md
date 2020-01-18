# Golang中的值传递和引用传递

## 值传递

Golang中默认的参数传递方式即为值传递

值传递的例子

像slice,

```go


type Values struct {
	mu            sync.Mutex
	dataArray     []byte
	dataInterface interface{}
	dataAtomic    atomic.Value
	dataMap       map[int]int
}


func TestGetPointerValue(t *testing.T) {
	values := Values{
		dataArray:     make([]byte, 0),
		dataMap:       map[int]int{},
	}
	values.dataAtomic.Store("这是个指针引用")
	fmt.Printf("&value=%p, dataArray=%p ,mutex=%p,dataInterface=%p,dataAtomic=%p dataMap=%p \n",
		&values, values.dataArray, &values.mu, &values.dataInterface, &values.dataAtomic,values.dataMap)
	passedFunc(values)
	fmt.Printf("")
	println(len(values.dataArray))
	println(values.dataMap[1])

	passFunc2(&values)
	fmt.Printf("")
	println(len(values.dataArray))
	println(values.dataMap[1])

	passFunc3(&values)
	fmt.Printf("")
	println(len(values.dataArray))
	println(values.dataMap[1])
}
func passedFunc(values Values) {
	// copy !

	// 写slice值没有影响
	values.dataArray = append(values.dataArray, 0)
	// 写map有影响
	values.dataMap[1] = 1
	fmt.Printf("&value=%p, dataArray=%p ,mutex=%p,dataInterface=%p,dataAtomic=%p dataMap=%p \n",
		&values, values.dataArray, &values.mu, &values.dataInterface, &values.dataAtomic,values.dataMap)
}

func passFunc2(values *Values) {
	// 写slice值有影响
	values.dataArray = append(values.dataArray, 0)
	// 写slice值有影响
	values.dataMap[1] =2
	fmt.Printf("&value=%p, dataArray=%p ,mutex=%p,dataInterface=%p,dataAtomic=%p dataMap=%p \n",
		values, values.dataArray, &values.mu, &values.dataInterface, &values.dataAtomic,values.dataMap)
}

func passFunc3(v *Values) {
	// 这里相当于重新值拷贝了一次
	values := *v
	// 写slice值有影响
	values.dataArray = append(values.dataArray, 0)
	// 写slice值有影响
	values.dataMap[1] =2
    
	fmt.Printf("&value=%p, dataArray=%p ,mutex=%p,dataInterface=%p,dataAtomic=%p dataMap=%p \n",
		&values, values.dataArray, &values.mu, &values.dataInterface, &values.dataAtomic,values.dataMap)
}

```

