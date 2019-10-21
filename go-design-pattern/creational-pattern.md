# Creational Pattern
 
## Singleton
- 생성을 질의하지 않고 바로 사용가능한 객체를 주는것

- having a unique instance of a type in the entire program

- We need a single, shared value, of some particular type.
- We need to restrict object creation of some type to a single unit along the entire program.

- Singleton pattern will give you the power to have a unique instance of some struct in your application and that no package can create any clone of this struct.

 
## Builder

- Builder 는 struct 를 형성하는 func 들이 각각 해당 struct 를 return 해줘서 func chaining이 가능하게끔 설계됨.

- Builder.construct() 했을때 완전한 객체가 생성.

- reusing an algorithm to create many implementations of an interface

- Abstract complex creations so that object creation is separated from the object user

- Create an object step by step by filling its fields and creating the embedded objects 

- Reuse the object creation algorithm between many objects

- any small change in this interface will affect all your builders and it could be awkward if you add a new method that some of your builders need and others Builders do not.

- (각 builder 는 Interface로 정의하는데, 한 builder에만 추가되고 다른 builder 에는 추가될 필요가 없는 function이 생기면 이상해질 수 있음 )


## Factory

- Factory method: delegating the creation of different types of payments

- To have a common method for every payment method called Pay
- To be able to delegate the creation of payments methods to the Factory
- To be able to add more payment methods to the library by just adding it to the factory method

## Abstract Factory

- Abstract Factory: a factory of factories
- Factory 에 추상레이어 하나를 더 얹음

```
VehicleFactory
> CarFactory
  
  >> LuxuryCar [Car, Vehicle]
    >>> GetWheels(), GetSeats(), GetDoors()
 
  >> FamiliarCar
    >>> GetWheels(), GetSeats(), GetDoors()
 
> MotorbikeFactory [Motorbike, Vehicle]
 
  >> SportMotorbike
    >>> GetWheels(), GetSeats(), GetType()
 
  >> CruiseMotorbike
    >>> GetWheels(), GetSeats(), GetType()
 
GetDoors() [Car]
GetType() [Motorbike]
GetWheels,GetSeats [Vehicle]
 ```
 

## Prototype

- The aim of the Prototype pattern is to have an object or a set of objects that is already created at compilation time, but which you can clone as many times as you want at runtime.

- var로 prototype(완전체)들을 미리 만들어놓고
- func GetClone 을 구현한 캐쉬에서 각 프로토타입들을 값 복사 이후 새 객체를 리턴해주고
- 리턴받은 객체에 값을 바꿔가며 원하는 객체로 바꿔간다

