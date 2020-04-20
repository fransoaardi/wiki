# intro 
- 동적 element 를 갖는 list 를 확장성있게 unmarshal 하고, 이후에 data handling 도 용이한 방법(type assertion 을 일일이 하지 않고)이 있을까 라는 질문에 시도해본 prototyping

- 아래와 같이 `Valuer` interface 에 동적인 element 각각을 static typing 할 수 있다. 아래의 예에서 `Value()` 가 상당히 무의미하지만, 이후에 data handling 할 때 유의미하게 구현할 수 있을것같다
```golang
// Parameter 는 Target.Parameters 의 동적 element 이다. Value 부분은 Value() 를 구현 하는 Valuer interface 로 정의 하여,
// 동적으로 unmarshal 할 수 있다.
type Parameter struct {
	Key   string `json:"key"`
	Value Valuer `json:"value"`
}
```

# progress

- `tJson` 는 동적인 list element 를 갖는다.(`key` 값에 따라 `value` 의 type 이 `string`, `int`, `object list` 등으로 바뀐다)
```golang
func main() {
	p := new(Target)
	err := json.Unmarshal([]byte(tJson), p)
	if err != nil {
		fmt.Println(err)
	}

	x, err := json.MarshalIndent(p, "", "  ")
	if err != nil {
		fmt.Println(err)
	}

	fmt.Println(string(x))
}

const tJson = `
{
	"parameters": [{
		"key": "poi.query",
		"value": "강남역"
	}, {
		"key": "poi.table",
		"value": [{
			"addr": "서울특별시 강남구 역삼동(역삼1동) 858",
			"tele": "1544-7788"
		}]
	}, {
		"key": "poi.table.len",
		"value": 5
	}]
}
`
```

- `UnmarshalJSON([]byte) error` 를 구현하면 `json.Unmarshaler` 가 된다. 
> `json.Unmarshal` 호출시, type check 를 해서, `json.Unmarshaler` 인 경우 custom fn 인 `UnmarshalJSON` 을 호출하도록 내부적으로 구현되어있다. 

```go
// code is from `json` package `decode.go` line 867:878
func (d *decodeState) literalStore(item []byte, v reflect.Value, fromQuoted bool) error {
	// Check for unmarshaler.
	if len(item) == 0 {
		//Empty string given
		d.saveError(fmt.Errorf("json: invalid use of ,string struct tag, trying to unmarshal %q into %v", item, v.Type()))
		return nil
	}
	isNull := item[0] == 'n' // null
	u, ut, pv := indirect(v, isNull)
	if u != nil {
		return u.UnmarshalJSON(item)
	}
```

- 따라서 아래와 같이, `UnmarshalJSON` 을 구현하면 `json.Unmarshal` 호출시 아래 fn 이 호출된다. 
```golang
// UnmarshalJSON 은 KeyChecker 로 key 를 판단하고, 나머지 부분을 적절한 struct 로 unmarshal 한다
func (p *Parameter) UnmarshalJSON(data []byte) error {
	var kc KeyChecker
	err := json.Unmarshal(data, &kc)
	if err != nil {
		return err
	}

	switch kc.Key {
	case "poi.query":
		x := PaAlias{Key: kc.Key, Value: new(poiQuery)}
		if err := json.Unmarshal(data, &x); err != nil {
			return err
		}
		*p = Parameter(x)
	case "poi.table":
		x := PaAlias{Key: kc.Key, Value: new(poiTable)}
		if err := json.Unmarshal(data, &x); err != nil {
			return err
		}
		*p = Parameter(x)
	case "poi.table.len":
		x := PaAlias{Key: kc.Key, Value: new(poiTableLen)}
		if err := json.Unmarshal(data, &x); err != nil {
			return err
		}
		*p = Parameter(x)
	default:
		return errors.New("undefined key")
	}
	return nil
}
```

# references

- https://blog.gopheracademy.com/advent-2016/advanced-encoding-decoding/

- https://developers.google.com/protocol-buffers/docs/reference/go-generated#oneof
> `golang gRPC oneof` 구현 방식을 보고 비슷하게 구현해봤다.
