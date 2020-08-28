# golang interface 에 대한 실험

## references
- https://medium.com/golangspec/type-assertions-in-go-e609759c42e1
- https://medium.com/rungo/interfaces-in-go-ab1601159b3a

## introduction
- 최초 궁금했던 부분에서.. 실험을 진행하다보니 온갖 실험을 해봤다.

1. golang 에서 interface 를 구현(implement)할때 receiver 를 일부는 Pointer, 일부는 Value 로 하면 어떤일이 발생할지 궁금해졌다.
  - 내용이 길어서 결과는 아래 참고.
2. `v, ok := val.(*X)` 와 같은 assertion fail (`ok==false`) 했을때, `v` 값은 `nil` 일까 `&X{}` 일까?
  - 정답: nil 이다
3. interface 선언을 inlining 해보고싶은데, `var x io.Reader` 형태밖에 없을까?
  - 조금 이상하긴 하지만, `x := interface{}(nil)` 과 같은 형태도 가능하다.
4. type assertion 할때 `i.(T)` 라고 할때 `i` 는 interface, `T` 는 type 이라 알고있는데, `T` 위치에 interface 정의한것을 본것같은데?
> 아래와 같이 가능하다, `T` 부분에 `walk()` 를 구현하는 concrete 한 type 을 정의했다고 볼 수 있나보다... (솔직히 잘 모르겠다)
```golang
bb, ok := k.(interface{
		walk()
	})
fmt.Println(bb, ok)
bb.walk() // 가능하다
```

## 1번에 대한 tl;dr;
- 해당 type 을 직접 선언해서 사용할때는 Pointer 로 선언하나 Value 로 선언하나 똑같이 호출 가능하다. 
> `X`든 `&X` 든 run(), walk() 는 모두 호출이 가능하다

- 다만, 해당 interface 로 선언한 변수에 type 을 할당한다면 문제가 생긴다. Pointer 로 전달했을때만 가능하다 
> interface 로 정의된 변수에 동적으로 할당할때 문제가 생긴다. (Pointer 로 전달하는건 괜찮다)

| | X | *X |
|---|---|---|
| Runner | X | O |
| Walker | O | O |
| RunnerWalker | X | O |



## code with comments
```golang
package main

import (
	"fmt"
)

type X struct {}

func (x X) walk() {
	fmt.Println("X walks")
}

func (x *X) run() {
	fmt.Println("X runs")
}

type Walker interface{
	walk()
}

type Runner interface{
	run()
}

type WalkerRunner interface{
	walk()
	run()
}

func main(){
	x1 := X{}
	x1.walk(); 	x1.run()	// 이건 실행 가능함

	x2 := &X{}
	x2.walk(); 	x2.run()	// 이것도 실행 가능함

	var wr WalkerRunner
	var r Runner
	var w Walker
	_, _ , _ = wr, r, w	// just to avoid unused error

	//wr = x1	// invalid, X{}  이건 안됨
	//wr = x2	// valid, &X{} 이건 됨

	//r = x1	// invalid, X{}
	//r = x2 	// valid, &X{}

	//w = x1 // valid, X{}
	//w = x2 // valid, &X{}

	// A type assertion provides access to an interface value's underlying concrete value.
	// assertion 할때 y 는 항상 interface 여야 한다. assertion 은 interface 를 구현체(concrete type)로 변경해내는 역할을 한다.
	// This statement asserts that the interface value i holds the concrete type T and assigns the underlying T value to the variable t.

	// var y interface{}
	y := interface{}(nil)	// 이런 표현식이 가능함
	fmt.Println("y==nil", y == nil)

	var y1 interface{}
	fmt.Println("y1==nil", y1 == nil)
	//zz := interface{}(struct{	// 이런것도 가능함
	//	aa string
	//}{})


	k := interface{
		run()
	}(&X{})

	//k1 := interface{		// 불가능하다
	//	run()
	//}(X{})

	k.run()
	//k.walk()  // k 는 run() 만 가능하다

	aa, ok := k.(*X)
	fmt.Println(aa, ok) //output: &{} true    &X{} 는 walk(), run() 둘다 구현했다

	bb, ok := k.(interface{
		walk()
	})
	fmt.Println(bb, ok)	//output: &{} true    &X{} 는 walk(), run() 둘다 구현했다

	cc, ok := k.(interface{
		quack()
	})
	fmt.Println(cc, ok)	//output: <nil>, false    구현한적 없어서

	//v, ok := y.(*X)	// assertion fail 했을때, v 값은 nil 이다. ( &X{} 이길 기대했는데.. )
	//fmt.Println(v, ok)
}

```
