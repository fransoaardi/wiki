# Structural Pattern
 

- We will keep using the Composite pattern in the following chapters, as it is the foundation for building relationships in the Go programming language.
 

## Composite Pattern

- The Go language, by nature, encourages use of composition almost exclusively by its lack of inheritance. Because of this, we have been using the Composite design pattern extensively until now, so let's start by defining the Composite design pattern.

- Swim도 하고 Eat도 하는 struct를 만들때

```go
package main
import (
    "fmt"
)
func main() {
    john := John{
        &pel{},
        &kim{},
    }
    john.Swim()
    john.Eat()
}
 
type Swimmer interface {
    Swim()
}
type Eatter interface {
    Eat()
}
type John struct {
    Swimmer
    Eatter
}
func (j John) Swim(){
    fmt.Println("John SWIM")
}
 
type pel struct{}
type kim struct{}
 
func (p pel) Swim() {
    fmt.Println("PEL SWIM")
}
func (k kim) Eat() {
    fmt.Println("KIM EAT")
}
 
//Output:
//John SWIM
//KIM EAT
```

## Adapter Pattern
- The Adapter pattern is very useful when, for example, an interface gets outdated and it's not possible to replace it easily or fast. Instead, you create a new interface to deal with the current needs of your application, which, under the hood, uses implementations of the old interface.

- legacy func를 만들어놓고, 이 부분을 대체할 새로운 func를 만들때
- adapter 를 놓고, 양쪽 다 호출할 수 있게끔 설계.

## Bridge Patterns
- The objective of the Bridge pattern is to bring flexibility to a struct that change often. Knowing the inputs and outputs of a method, it allows us to change code without knowing too much about it and leaving the freedom for both sides to be modified more easily.

- NormalPrinter1와  NormalPrinter2가 있다고 가정한다.

- 각각의 printer 는 PrinterAPI를 갖고있는데 PrinterAPI는 2가지 종류가 있다.
- 각각의 PrinterAPI는 PrinterAPI1, PrinterAPI2이다.
- 각각을 조합했을때 4가지 경우의 수를 만들 수 있다.
- 다양하게 만들어놓고 변경을 용이하게 하려고 한듯.

- 어떻게보면 factory패턴과 비슷할 수 있겠다.

- 같은 Weapon 를 Smith, Warrior 에게 줬을때 각각 다른 action을 한다. 
- 둘다 handle()만 했을뿐

## Proxy
- proxy를 전면에 세워놓고 뒷단 로직을 가릴수있다.
- proxy와만 소통하면 된다.
- user 가 원하는 내용은 주는데 과정은 알 필요 없다.

## Decorator
- Decorator 를 돌려가면서 무언가를 Add 할 수 있는 패턴 (계속 Decorate 한다)
- 크리스마스 트리를 만들어놓으면 계속 무언가를 붙인다.
- Server 에서 ServeHTTP 를 만들어놓고, Handler 안에 Handler 넣어가면서 기능 추가하는 Decorator Pattern 적용 가능

## Proxy vs Decorator
- proxy 는 접근제어, decorate 는 기능 확장


## Fascade
- 단면
- 뒷단에서 무슨일이 일어나는지 가려놓고, 요청하는결과를 줌
- 도시, 국가만 정해주면 날씨 위도 경도 등등 원하는 정보를 뽑아주도록.

- 쓸모없는 기능을 없애주고, 필요한것만 기능 제한


## Flyweight
-
