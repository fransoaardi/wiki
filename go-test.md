# intro

- How to run test cases in a specified file?
- https://stackoverflow.com/questions/16935965/how-to-run-test-cases-in-a-specified-file

## go test 

```bash
$ go test -v
```
하면 해당 디렉토리에 있는 모든 테스트가 실행되었다.

특정 테스트만 진행해보고 싶어서 찾아보았다.

아래의 디렉토리에 있는 `/Users/kakao/go/src/github.com/Go-Design-Patterns/Chapter02`

```bash
~/g/s/g/G/Chapter02 ❯❯❯ ls master ✱
abstract_factory factory.go prototype.go
builder.go factory_test.go prototype_test.go
builder_test.go main.go singleton_test.go
 ```

`prototype_test.go` 를 `go test` 하려고 하는데

```go
$ go test -v prototype_test.go
```

했을때 에러가 발생하였다.

```bash
~/g/s/g/G/Chapter02 ❯❯❯ go test -v prototype_test.go
# command-line-arguments
./prototype_test.go:6:16: undefined: GetShirtsCloner
./prototype_test.go:11:36: undefined: White
./prototype_test.go:16:17: undefined: whitePrototype
./prototype_test.go:17:14: undefined: whitePrototype
./prototype_test.go:27:36: undefined: White
FAIL command-line-arguments [build failed]
``` 

패키지 경로에 `prototype.go` 을 포함하였다.

```bash
~/g/s/g/G/Chapter02 ❯❯❯ go test -v prototype_test.go prototype.go
=== RUN TestClone
--- PASS: TestClone (0.00s)
prototype_test.go:16: 0xc42004c3f0 0x11dbf90
prototype_test.go:37: abbcc empty
prototype_test.go:46: LOG: Shirt with SKU 'abbcc' and Color id 1 that costs 15.000000
prototype_test.go:47: LOG: Shirt with SKU 'empty' and Color id 1 that costs 15.000000
prototype_test.go:49: LOG: The memory positions of the shirts are different 0xc42000c030 != 0xc42000c038
PASS
ok command-line-arguments 0.007s
```

`abstract_factory` 디렉토리에 있는 테스트를 실행하려고 할때는
패키지 경로에 `$GOPATH/src` 이후의 경로를 입력하였다.

```bash
~/g/s/g/G/Chapter02 ❯❯❯ go test -v github.com/Go-Design-Patterns/Chapter02/abstract_factory
=== RUN TestMotorbikeFactory
--- PASS: TestMotorbikeFactory (0.00s)
vehicle_factory_test.go:16: Motorbike vehicle has 2 wheels and 1 seats
vehicle_factory_test.go:22: Sport motorbike has type 1
vehicle_factory_test.go:29: Motorbike vehicle has 2 wheels
vehicle_factory_test.go:35: Cruise motorbike has type 2
=== RUN TestCarFactory
--- PASS: TestCarFactory (0.00s)
vehicle_factory_test.go:59: Car vehicle has 5 seats and 4 wheels
vehicle_factory_test.go:65: Luxury car has 4 doors.
vehicle_factory_test.go:72: Car vehicle has 4 seats
vehicle_factory_test.go:78: Familiar car has 5 doors.
PASS
ok github.com/Go-Design-Patterns/Chapter02/abstract_factory 0.006s
``` 

추가로, 아래와같이 `TestFoo`가 세팅되어있고,
`A=1`, `A=2`, `B=1` 과 같은 `SubTest`들이 정의되어있는 경우

```go
func TestFoo(t *testing.T) {
    // <setup code>
    t.Run("A=1", func(t *testing.T) { ... })
    t.Run("A=2", func(t *testing.T) { ... })
    t.Run("B=1", func(t *testing.T) { ... })
    // <tear-down code>
}
```
 
```bash 
$ go test -run '' # Run all tests.
$ go test -run Foo # Run top-level tests matching "Foo", such as "TestFooBar".
$ go test -run Foo/A= # For top-level tests matching "Foo", run subtests matching "A=".
$ go test -run /A=1 # For all top-level tests, run subtests matching "A=1".
```

## benchmark

benchmark 사용은

``` go
func BenchmarkXxxxx(*testing.B)
```

형태의 func들이
`go test` 시 `-bench` 를 주면 벤치마크가 실행된다.
  

```bash
$ go test -bench .
```
 

## 참고Link
 

- GoDoc 중 command 관련 (flag 정보도 포함되어있음)
https://golang.org/cmd/go/

- GoDoc에 있는 test 관련 가이드문서
https://golang.org/pkg/testing/#hdr-Subtests_and_Sub_benchmarks

 
