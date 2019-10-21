# decorate vs strategy
- strategy changes the guts of the object, decorate changes the skin.

## Strategy
- Runtime에 strategy를 갈아끼운다.

- 알고리즘을 숨길수있는 패턴(캡슐화/ 혹은 갈아끼움)
- 이기기위한 목표는 하나 - 택할 수 있는 전략은 여럿

## Chain of Responsibility
- single responsibility principal 을 위한 패턴(단일책임원칙)
- 병렬적으로 실행
- 개발자에게 오전, 오후, 저녁에 출근하라는 책임이 주어졌을때, 오전,오후,저녁에 출근을 다하면된다.

## Command
- commander가 box에 정보를 넣어서 보내면, receiver가 box를 열어서 처리
- commander는 스스로 실행 x
- 어명을 들고 가는 사람은 열어볼 수 없고, 읽어보고 집행할 수 있는 사람은 따로있다.

- COR은 스스로 실행될 권한이 있음

# Template Pattern
- 일부 알고리즘을 재사용하도록 함
- 수정 불가능한 부분을 만들어놓고, 유저가 변경 가능한 일부만 노출, 변경 가능하게끔

## Memento
- 상태의 Recover를 위한 Reminder.
- memento, originator, caretaker
- care taker : originator 에 의해 생성된 memento를 실행하는 역할
- originator : memento 의 행동들을 기록하고있음

- originator 는 상태를 받고 변화를 하며, memento를 남겨준다.
- 남긴 memento를 caretaker에 저장해놓는다.
- caretaker 은 나중에 undo 할때 쓰일 수 있다.

- Idle상태에서 주먹을 지르라는 커맨드 들어옴 -> 주먹을 지르고 , pop하면 다시 Idle로 돌아감

 
## Interpreter
- Domain Specific Language를 만들때 사용
- SQL이 대표적인 예시
- 예제에서 Read func 를 만들어 놓고, 


## Visitor Pattern

- visitable의 struct 내부에 algorithm을 구현하는것이 아니라, visitor에 algorithm을 구현하여 decoupling 시키는것
- 구현하고자 하는 algorithm에 따라 visitor 들을 여러개 만들어서 param으로 전달해주는 방식으로 쉽게 적용가능하다.
- 새로운 algorithm을 적용할때, 기존의  struct type에 변경을 가하지 않았다.

- visitable.Accept(Visitor) 형태로 구현하고, Accept는 아래와 같이 구현한다.

```go
func (Visitable)Accept(Visitor){
   Visitor.Visit(Visitable)
}
```

- The key is to understand that the Visitor pattern executes an algorithm in its Visit method that deals with the Visitable object

- To separate the algorithm of some type from its implementation within some other type

- To improve the flexibility of some types by using them with little or no logic at all so all new functionality can be added without altering the object structure

- To fix a structure or behavior that would break the open/closed principle in a type

- open/closed principle states that: entities should be open for extension but closed for modification.

- To effectively use the Visitor design pattern, we must have two roles–a visitor and a visitable. The Visitor is the type that will act within a Visitable type. So a Visitable interface implementation has an algorithm detached to the Visitor type:

- The very important thing here is that we can add more functionality to both the structs, MessageA and MessageB, without altering their types.

- We can just create a new Visitor type that does everything on the Visitable, for example, we can create a Visitor to add a method that prints the contents of the Msg field:

- We have just added some functionality to both types without altering their contents! That's the power of the Visitor design pattern.

- we decoupled some algorithms outside of the types to the visitors.

```go
// visitor.go
package visitor
 
import (
   "io"
   "fmt"
   "os"
)
  
// MessageA, B struct 를 visit할 Visitor 가 있다. (interface)
// MessageVisitor, MsgFieldVisitorPrinter 는 VisitA, VisitB를 구현해서 Visitor 이다.
 
// MessageA, B는 Accept를 구현하여 Visitable 이다.
// MessageA, B (Visitable)의 Msg 를 변경, 출력하는 '알고리즘'을 Visitor 에 구현한다.
 
// Visitable 구현 영역
 
type Visitable interface {
   Accept(Visitor)    // Visitable은 Visitor를 parameter로 하는 Accept를 구현한다.
}
 
// Accept는 Visitor의 VisitA에 자신을 parameter로 전달한다.
func (m *MessageA) Accept(v Visitor) {
   v.VisitA(m)
}
 
// Accept는 Visitor의 VisitB에 자신을 parameter로 전달한다.
func (m *MessageB) Accept(v Visitor) {
   v.VisitB(m)
}
 
type MessageA struct {
   Msg string
   Output io.Writer
}
 
type MessageB struct {
   Msg string
   Output io.Writer
}
 
func (m *MessageA) Print() {
   if m.Output == nil {
      m.Output = os.Stdout
   }
 
   fmt.Fprintf(m.Output, "A: %s", m.Msg)
}
 
func (m *MessageB) Print() {
   if m.Output == nil {
      m.Output = os.Stdout
   }
 
   fmt.Fprintf(m.Output, "B: %s", m.Msg)
}
 
//Visitor 구현 영역
 
type Visitor interface {
   VisitA(*MessageA)
   VisitB(*MessageB)
}
 
type MessageVisitor struct {}
 
//VisitA 는 MessageA의 Msg를 "${Msg} (Visited A)" 라고 바꾼다.
func (mf *MessageVisitor) VisitA(m *MessageA){
   m.Msg = fmt.Sprintf("%s %s", m.Msg, "(Visited A)")
}
 
//VisitB 는 MessageB의 Msg를 "${Msg} (Visited B)" 라고 바꾼다.
func (mf *MessageVisitor) VisitB(m *MessageB){
   m.Msg = fmt.Sprintf("%s %s", m.Msg, "(Visited B)")
}
 
type MsgFieldVisitorPrinter struct {}
 
//VisitA 는 MessageA의 Msg를 콘솔에 출력한다.
func (mf *MsgFieldVisitorPrinter) VisitA(m *MessageA){
   fmt.Printf(m.Msg)
}
 
//VisitB는 MessageB의 Msg를 콘솔에 출력한다.
func (mf *MsgFieldVisitorPrinter) VisitB(m *MessageB){
   fmt.Printf(m.Msg)
}
```

```go
// visitor_test.go
package visitor
 
import "testing"
 
type TestHelper struct {
   Received string
}
 
func (t *TestHelper) Write(p []byte) (int, error) {
   t.Received = string(p)
   return len(p), nil
}
 
func Test_Overall(t *testing.T) {
   testHelper := &TestHelper{}
   visitor := &MessageVisitor{}
 
   t.Run("MessageA test", func(t *testing.T){
      msg := MessageA{
         Msg: "Hello World",
         Output: testHelper,    //Output은 io.Writer 인데 TestHelper는 Write를 구현했으니 io.Writer가 맞다.
      }
 
      msg.Accept(visitor)    // msg.Accept는 msg의 Msg의 값 뒤에 (Visited A)를 붙여준다
 
      expected := "Hello World (Visited A)"
      if msg.Msg !=  expected {
         t.Errorf("Expected result was incorrect. %s != %s",
            msg.Msg, expected)
      }
 
      msg.Print()    // msg.Print는 msg의 Output에 A: msg.Msg 값을 준다
 
      expected = "A: Hello World (Visited A)"
      if testHelper.Received !=  expected {
         t.Errorf("Expected result was incorrect. %s != %s",
            testHelper.Received, expected)
      }
   })
 
   t.Run("MessageB test", func(t *testing.T){
      msg := MessageB {
         Msg: "Hello World",
         Output: testHelper,
      }
 
      msg.Accept(visitor)
      msg.Print()
 
      expected := "B: Hello World (Visited B)"
      if testHelper.Received !=  expected {
         t.Errorf("Expected result was incorrect. %s != %s",
            testHelper.Received, expected)
      }
   })
}
```

```go
//visitor2.go
package main
 
import "fmt"
 
// INTERFACES --------------------------------------------------------------
 
type ProductInfoRetriever interface {
   GetPrice() float32
   GetName() string
}
 
type Visitor interface {
   Visit(ProductInfoRetriever)
}
 
type Visitable interface {
   Accept(Visitor)
}
 
// PRODUCT -----------------------------------------------------------------
 
type Product struct {
   Price float32
   Name  string
}
 
func (p *Product) GetPrice() float32 {
   return p.Price
}
 
func (p *Product) Accept(v Visitor) {
   v.Visit(p)
}
 
func (p *Product) GetName() string {
   return p.Name
}
 
// PRODUCTS ----------------------------------------------------------------
 
type Rice struct {
   Product
}
 
type Pasta struct {
   Product
}
 
 
type Fridge struct {
   Product
}
 
//GetPrice overrides GetPrice method of Product type
func (f *Fridge) GetPrice() float32 {
   return f.Product.Price + 20
}
 
//Accept overrides "Accept" method from Product and implements the Visitable
//interface
func (f *Fridge) Accept(v Visitor) {
   v.Visit(f)
}
 
// VISITOR -----------------------------------------------------------------
 
type PriceVisitor struct {
   Sum float32
}
 
func (pv *PriceVisitor) Visit(p ProductInfoRetriever) {
   pv.Sum += p.GetPrice()
}
 
type NamePrinter struct {
   ProductList string
}
 
func (n *NamePrinter) Visit(p ProductInfoRetriever) {
   n.ProductList = fmt.Sprintf("%s\n%s", p.GetName(), n.ProductList)
}
 
// MAIN -------------------------------------------------------------------
 
func main() {
   products := make([]Visitable, 3)
   products[0] = &Rice{
      Product: Product{
         Price: 32.0,
         Name:  "Some rice",
      },
   }
   products[1] = &Pasta{
      Product: Product{
         Price: 40.0,
         Name:  "Some pasta",
      },
   }
   products[2] = &Fridge{
      Product: Product{
         Price: 50,
         Name:  "A fridge",
      },
   }
 
   priceVisitor := &PriceVisitor{}
 
   for _, p := range products {   // p는 Visitable 이다
      p.Accept(priceVisitor) // p.Product.Accept 는 PriceVisitor.Visit을 호출한다.
      // PriceVisitor 는 Sum 을 갖고있어서 Sum 에다 Product의 Price 값을 더해줌 (Product Sum 구해서 갖고있음)
   }
 
   fmt.Printf("Total: %f\n", priceVisitor.Sum)    // priceVisitor.Sum은 결국 위에서 구한 product 들의 price의 sum
 
 
   nameVisitor := &NamePrinter{}
 
   for _, p := range products {   // sum더하는 로직을 구현한 priceVisitor 대신 nameVisitor를 인자로 넘기면,
      p.Accept(nameVisitor)  // NamePrinter의 ProductList에 rice , pasta ,  fridge 순서대로 저장해서 거꾸로 올라감
   }
 
   fmt.Printf("\nProduct list:\n-------------\n%s", nameVisitor.ProductList)
}

```

## State Pattern

- State patterns are directly related to FSMs(Finite State Machine).

- An FSM, in very simple terms, is something that has one or more states and travels between them to execute some behaviors.

- One state can transit to the other and vice versa.

- 한 State는 다른 State로 전환 가능하고 그 반대의 경우도 가능하다.

- a State interface and an implementation of each state we want to achieve.

- There is also usually a context that holds cross-information between the states

- State 변화사이에 context를 가지고 information을 주고받는다.

- To have a type that alters its own behavior when some internal things have changed

- Model complex graphs and pipelines can be upgraded easily by adding more states and rerouting their output states

- output state를 추가하여 쉽게 복잡도를 증가 시킬 수 있다.

- So, the idea is simple: we execute the state in the context, passing a pointer to the context to it. Each state returns true until the game has finished and the FinishState struct will return false

- The power of the State pattern is not only the capacity to create a complex FSM, but also the flexibility to improve it as much as you want by adding new states and modifying some old states to point to the new ones without affecting the rest of the FSM.

```go
//state.go
  
package main
 
import (
   "fmt"
   "os"
   "math/rand"
   "time"
)
 
 
type GameState interface {
   executeState(*GameContext) bool
}
// GameContext struct 는 GameState 를 갖는다.
// 이 게임은 StartState > AskState > FinishState 순으로 진행된다.
// 위의 State 는 각각 executeState 를 구현하는 GameState interface 이다.
// Context에서 Next 를 갈아끼워가면서 main의 range를 돌때 계속 진행되도록 구현되어있다.
// GameState의 executeState는 GameContext를 입력받는데, 이 GameContext를 계속 들고다닌다.
 
type GameContext struct {
   SecretNumber int
   Retries int
   Won bool
   Next GameState
}
 
type StartState struct{}
func(s *StartState) executeState(c *GameContext) bool {
   c.Next = &AskState{}
 
   rand.Seed(time.Now().UnixNano())
   c.SecretNumber = rand.Intn(10)
   fmt.Println("Introduce a number a number of retries to set the difficulty:")
   fmt.Fscanf(os.Stdin, "%d\n", &c.Retries)
 
   return true
}
 
type FinishState struct{}
func(f *FinishState) executeState(c *GameContext) bool {
   if c.Won {
      println("Congrats, you won")
   } else {
      fmt.Printf("You loose. The correct number was: %d\n", c.SecretNumber)
   }
 
   return false
}
 
type AskState struct {}
func (a *AskState) executeState(c *GameContext) bool{
   fmt.Printf("Introduce a number between 0 and 10, you have %d tries left\n", c.Retries)
 
   var n int
   fmt.Fscanf(os.Stdin, "%d", &n)
   c.Retries = c.Retries - 1
 
   if n == c.SecretNumber {
      c.Won = true
      c.Next = &FinishState{}
   }
 
   if c.Retries == 0 {
      c.Next = &FinishState{}
   }
 
   return true
}
 
func main() {
   start := StartState{}
   game := GameContext{
      Next:&start,
   }
 
   for game.Next.executeState(&game) {}
 
   for i:=0; false; { // for test
      fmt.Println(i)
      i--
   }
}
``` 

## Mediator Pattern
 

- 중재자 패턴을 사용하면 객체 간 통신은 중자재 객체 안에 함축된다. 객체들은 더 이상 다른 객체와 서로 직접 통신하지 않으며 대신 중재자를 통해 통신한다. 이를 통해 통신 객체 간 의존성을 줄일 수 있으므로 결합도를 감소시킨다.

- Wikipedia

### 출처

- https://dzone.com/articles/design-patterns-mediator  

> 완전 observer pattern랑 구분이 안감

![image](https://user-images.githubusercontent.com/34496756/67188870-2a79b800-f428-11e9-95a5-34dcbe538b36.png)

![image](https://user-images.githubusercontent.com/34496756/67188913-3e251e80-f428-11e9-9289-7aa2a32fffd3.png)

- https://sourcemaking.com/design_patterns/mediator

> 비행기 여러대가 서로 통신하기 보다는, 가운데에 관제탑(mediator)을 두고 통신을 한다

![image](https://user-images.githubusercontent.com/34496756/67188784-01f1be00-f428-11e9-95f4-30b5b0f9da45.png)

> Users and Groups are decoupled from one another, many mappings can easily be maintained and manipulated simultaneously, and the mapping abstraction can be extended in the future by defining derived classes.

- 가운데 mediator를 두고, 서로 colleague가 된다.

- 서로 mediator에게 listen을 하고있고 변화가 생기면 서로에게 notify한다.

- notify시 mediator 에 등록된 대상 중 본인을 제외한 모두에게 notify한다. broadcasting과 같이 진행.

 
- observer pattern이 한 publisher가 여러 observer에게 notify한다면  (1:N)

- mediator 패턴은 서로 broadcasting 한다. (N:M)


- It's a pattern that will be in between two types to exchange information.

- One of the key objectives of any design pattern is to avoid tight coupling between objects.

- The pattern that maintains which objects give what information is the Mediator.

- the main objectives of the Mediator pattern are about loose coupling and encapsulation. The objectives are:
 
- To provide loose coupling between two objects that must communicate between them

- To reduce the amount of dependencies of a particular type to the minimum by passing these needs to the Mediator pattern

- Just remember that the Mediator pattern is there to act as a managing type between two types that don't know about each other
so that you can take one of the types without affecting the other and replace a type in a more easy and convenient way.

## Mediator VS Observer (Need Help)
 
- Mediator는 N:M 관계, Observer 는 1:N 관계 (??)
- Observer  Pattern은 한개의 trigger가 여러개의 event를 발생시킬때 유용. (1개의 publisher가 여러개의 listener에게 notify함)
 

## Observer Pattern
 
-Observer Pattern은  publisher와 observer(listener) 가 있다.

- publisher 는 observer 들을 list로 관리하고 있다가, broadcast 하면 observer 들이 loop를 돌면서 변화를 감지하는 패턴이다.

- 아래의 예제에서 observer 는 notify를 구현, publisher는 notifyToObserver 를 구현하고 notifyToObserver는 loop를 돌면서 observer.notify를 실행했다.

- the Observer pattern, also known as publish/subscriber or publish/listener.

- subscribe to some event that will trigger some behavior on many subscribed types. Why is this so interesting? Because we uncouple an event from its possible handlers.

- The Observer pattern is especially useful to achieve many actions that are triggered on one event. It is also especially useful when you don't know how many actions are performed after an event in advance or there is a possibility that the number of actions is going to grow in the near future.

- When the Publisher struct is triggered, it must notify all its observers of the new event with the data associated.

- The Publisher structure stores the list of subscribed observers in a slice field called ObserversList. Then it has the three methods mentioned on the acceptance criteria-the AddObserver method to subscribe a new observer to the publisher, the RemoveObserver method to unsubscribe an observer, and the NotifyObservers method with a string that acts as the message we want to spread between all observers

- We have unlocked the power of event-driven architectures with the State pattern and the Observer pattern.

- The Observer pattern is commonly used in UI's. Android programming is filled with Observer patterns

- so that the Android SDK can delegate the actions to be performed by the programmers creating an app.

```go
// observer.go
package observer
 
import "fmt"
 
// Observer , Publisher 가 있다.
// Observer 는 Notify 를 구현한다.
// Publisher는 ObserverList를 갖는다.
 
// Publisher 는 Observer 를 추가, 삭제, 하는 로직을 구현하고,
// Publisher 가 Loop를 돌며 Publisher가 갖고있는 Observer 들을 Notify 가능하다.
// Notify에 로직 구현하면 된다.
 
type Observer interface {
   Notify(string)
}
 
type Publisher struct {
   ObserversList []Observer
}
 
func (s *Publisher) AddObserver(o Observer) {
   s.ObserversList = append(s.ObserversList, o)
}
 
func (s *Publisher) RemoveObserver(o Observer) {
   var indexToRemove int
 
   for i, observer := range s.ObserversList {
      if observer == o {
         indexToRemove = i
         break
      }
   }
 
   s.ObserversList = append(s.ObserversList[:indexToRemove], s.ObserversList[indexToRemove+1:]...)
}
 
func (s *Publisher) NotifyObservers(m string) {
   fmt.Printf("Publisher received message '%s' to notify observers\n", m)
   for _, observer := range s.ObserversList {
      observer.Notify(m)
   }
}
```

```go
// observer_test.go
package observer
 
import (
   "fmt"
   "testing"
)
 
type TestObserver struct {
   ID      int
   Message string
}
 
//Observer 의 Notify는 , Observer 가 Publisher 로 부터 받은 message를 출력하고,
//Observer의 Message안에 값을 넣는다.
 
func (p *TestObserver) Notify(m string) {
   fmt.Printf("Observer %d: message '%s' received \n", p.ID, m)
   p.Message = m
}
 
func TestSubject(t *testing.T) {
   testObserver1 := &TestObserver{1, "default"}
   testObserver2 := &TestObserver{2, "default"}
   testObserver3 := &TestObserver{3, "default"}
 
   publisher := Publisher{}
 
   t.Run("AddObserver", func(t *testing.T) {
      publisher.AddObserver(testObserver1)
      publisher.AddObserver(testObserver2)
      publisher.AddObserver(testObserver3)
 
      if len(publisher.ObserversList) != 3 {
         t.Fail()
      }
   })
 
   t.Run("RemoveObserver", func(t *testing.T) {
      publisher.RemoveObserver(testObserver2)
 
      if len(publisher.ObserversList) != 2 {
         t.Errorf("The size of the observer list is not the "+
            "expected. 3 != %d\n", len(publisher.ObserversList))
      }
 
      for _, observer := range publisher.ObserversList {
         testObserver, ok := observer.(*TestObserver)
         if !ok {
            t.Fail()
         }
 
         if testObserver.ID == 2 {
            t.Fail()
         }
      }
   })
 
   t.Run("Notify", func(t *testing.T) {
 
      if len(publisher.ObserversList) == 0 {
         t.Errorf("The list is empty. Nothing to test\n")
      }
 
      for _, observer := range publisher.ObserversList {
         printObserver, ok := observer.(*TestObserver)
         if !ok {
            t.Fail()
            break
         }
 
         if printObserver.Message != "default" {
            t.Errorf("The observer's Message field weren't"+
               " empty: %s\n", printObserver.Message)
         }
      }
 
      message := "Hello World!"
      publisher.NotifyObservers(message)
 
      for _, observer := range publisher.ObserversList {
         printObserver, ok := observer.(*TestObserver)
         if !ok {
            t.Fail()
            break
         }
 
         if printObserver.Message != message {
            t.Errorf("Expected message on observer %d was "+
               "not expected: '%s' != '%s'\n", printObserver.ID,
               printObserver.Message, message)
         }
      }
   })
}
```
