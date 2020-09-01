# functions from the golang strings package

## codes

### Fields

```golang
    // scanner.Text() 가 " abc def abk " 라면, ["abc", "def", "abk"] 로 나뉨 
    // 아래 두 줄은 의도한 바 동일하게 동작함
    // x := strings.Split(strings.TrimSpace(scanner.Text()), " ")
		x := strings.Fields(scanner.Text())
```

> reference: golang 1.15 `strings/strings.go`
```golang
func Fields(s string) []string {
	// First count the fields.
	// This is an exact count if s is ASCII, otherwise it is an approximation.
	n := 0
	wasSpace := 1
	// setBits is used to track which bits are set in the bytes of s.
	setBits := uint8(0)
	for i := 0; i < len(s); i++ {
		r := s[i]
		setBits |= r
		isSpace := int(asciiSpace[r])
		n += wasSpace & ^isSpace
		wasSpace = isSpace
	}

	if setBits >= utf8.RuneSelf {
		// Some runes in the input string are not ASCII.
		return FieldsFunc(s, unicode.IsSpace)
	}
  
  (...)
```
> golang 1.15 `unicode/utf8/utf8.go`
- `RuneSelf  = 0x80         // characters below RuneSelf are represented as themselves in a single byte.`
- `Fields` 는 rune 으로 표현될 수 있는 문자열에 한해서, FieldsFunc(s,unicode.IsSpace) 와 동일하게 동작함
- 아래 생략된 부분은 byte 문자열이라 가정하고 asciiSpace 로 정의된 공백 문자열들 제거하고, 새로운 문자열이 시작할때 그 이전까지의 문자열을 담는 형태를 취하고있음 
- `asciiSpace` 를 아래와 같이 정의한게 흥미롭다. ascii 는 7bit 문자열(혹은 앞에 0을 붙여서 8bit 로 쓰기도 한다)이고, ascii[x] 이런식으로 코드에서 조회하기때문에 혹시 index outofbound 생길까봐 256 length 로 넉넉하게 잡은 것 같다.
- `var asciiSpace = [256]uint8{'\t': 1, '\n': 1, '\v': 1, '\f': 1, '\r': 1, ' ': 1}`
- 참고: 아래 코드는 적합한 코드이다.
```golang
	var x = [10]int{0:1, 2:3, 3:5}
	fmt.Println(x)
  // Output: [1 0 3 5 0 0 0 0 0 0]
```

### FieldsFunc()
> reference: https://golang.org/pkg/strings/#FieldsFunc
```golang
  f := func(c rune) bool {
		return !unicode.IsLetter(c) && !unicode.IsNumber(c)
	}
	fmt.Printf("Fields are: %q", strings.FieldsFunc("  foo1b ar2,baz3...", f))
```

