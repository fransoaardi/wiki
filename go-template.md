## 참고 Links
 
- GoDoc text/template 패키지
https://godoc.org/text/template

- GoDoc html/template 패키지
https://godoc.org/html/template

- Template 사용 예제
https://www.joinc.co.kr/w/man/12/golang/networkProgramming/template

- template 에 변수 여러개 넘기기
https://stackoverflow.com/questions/18276173/calling-a-template-with-several-pipeline-parameters

## template 파일 작성법

```
// test.gohtml
This is "test.gohtml" template
It embeds "embed.gohtml"
 
This is VarMap : {{ . }}
This is IntSlice : {{ .IntSlice }}
This is Val1 : {{ .Val1 }}
This is Val2 : {{ .Val2 }}
This is Map1 : {{ .Map1 }}
 
{{ if .Val3 }}
  {{ .Val3 }} is not blank
{{ end }}
 
{{ range $it := .Map1 }}
  {{ template "embed.gohtml" PassValues "Global" $ "Local" . }}
{{ end }}
`
 
` // embed.gohtml
This is "embed.gohtml" template
This is "embed.gohtml" Dot : {{ . }}
This is "embed.gohtml" Dollar : {{ $ }}
 
{{ if .Global }}
  This is Global {{ .Global }}
{{ end }}
 
{{ if .Local }}
  This is Local {{ .Local }}
{{ end }}
`
// 사용할 funcMap ( key : function 호출 명칭, value : 소스에 구현한 function )
var funcMap = template.FuncMap{
    "ConcatStrings": ConcatStrings,
    "SliceString": SliceString,
    "ContainsString": ContainsString,
    "AddInt": AddInt,
    "MinusInt": MinusInt,
    "ModInt": ModInt,
    "Loop": Loop,
    "PassValues": PassValues,
}
// 넘길 data struct
data := struct {
    IntSlice []int
    Val1 string
    Val2 string
    Val3 string
    Map1 map[string]string
}{
    []int{10, 20, 30},
    "Val1",
    "Val2",
    "",
    map[string]string {
        "key1": "val1",
        "key2": "val2",
    },
}
var a bytes.Buffer
// embed.gohtml, test.gohtml 을 파싱하고, 파싱된 결과를 "test.gohtml" 라고 정의함
// test.gohtml 이 embed.gohtml 을 포함하기때문에 embed.gohtml을 먼저 파싱해야됨
// (*Template).Funcs는 Parse하기 전에 정의해야됨.
 
tmpl, err := template.New("test.gohtml").Funcs(funcMap).ParseFiles(경로+"embed.gohtml", 경로+"test.gohtml")
if err != nil {
    t.Error(err)
}
err = tmpl.Execute(&a, data)
//err = tmpl.ExecuteTemplate(&a, "test.gohtml", data) 위와 동일한 구문
if err != nil {
 
    t.Error(err)
}
``` 

## how-to

- 주석
```
{{/* 성공적인 주석입니다. */}}
{{ /* {{와 / 사이에 공백이 있어 주석 실패 */ }}
```

- if
```
{{ if }}, {{ end }} 를 이용합니다.
{{ if eq .value "AAA" }}
  {{ .value }}는 "AAA" 와 같습니다.
{{ end }}

{{ if .value }}
  {{ .value }} 값이 빈값이 아니라면 (nil은 안됨) 값이 출력됨
{{ end }}
``` 

- else , else if
```
{{ if }}
  ...
{{ else if }}
  ...
{{ else if }}
  ...
{{ else }}
  ...
{{ end }}
``` 

- and, or
대체적으로 연산자를 순차적으로 사용한다.

if .A != "D" && .B == "B" || .C >= "C" 는 아래와 같다.

```
{{ if or (and (ne .A "D") (eq .B "B")) (ge .C "C") }}
```

- 변수 할당

local 변수 선언시 항상 변수명 앞에 $를 붙인다.
변수에는 := 을 이용해서 값을 할당한다.

```
{{ $var := "newVariable" }} {{/* 1차 할당 */}}
{{ $var := "valueChanged" }} {{/* 새 변수가 아니더라도 := 를 이용한다 */}}
``` 

- 수학 연산

수학연산은 불가능함. 따라서 별도 function을 작성해서 사용해야됨

```
{{ $var := $var + 1 }} {{/* 에러 */}}
{{ $var := AddInt $var 1 }} {{/* $var에 1이 증가됨 */}}
```

- range

```
{{ range $idx, $col := .collections }}
  {{ $idx }} 번째 {{ $col.value }} 값이 있다.
{{ end }}
 
{{ range .collections }}
  {{ . }}에는 .collections 를 순차적으로 꺼낸 값이 할당된다.
  range 안에서 global {{ . }}은 {{ $ }}로 할당되고,
  {{ .dispCode }}는 range 안에서 {{ $.dispCode }}로 참조해야된다.
{{ end }}
``` 

- template

```
{{ template "embed.gohtml" . }}
```

"embed.gohtml"에 . 을 dataVariable로 넘기고,

해당 위치에 "embed.gohtml" 의 내용을 포함시킨다.
 

- block

```
{{ block "embed.gohtml" . }} embed.gohtml 이 없으면 이 값이 대신 포함된다. {{ end }}
``` 

- function 사용

| (pipeline) 을 이용하는 방법과 앞에 명시해주는 방법이 있다.

아래 두가지 표현식의 결과는 같다.
html function에 urlQuery를 인자로 던져주고 return을 결과로 한다.

```
{{ .urlQuery | html }}
{{ html .urlQuery }}
``` 

- HTML unescape

HTML embed를 위한 string 을 템플릿에 출력시, html escape되어 html이 아닌 string 형으로 붙게된다 ( <div> </div>  ..   등 )

템플릿에 넘길 data에 아래와 같이 HTML func 를 사용하여 unescape를 명시할 수 있다.

```
htmlString :=  "<div> aaa </div>"
moTemplate.VarMap["collections"] = []map[string]interface{}{
   {
     "value":   "AAA",
     "html":    template.HTML(htmlString),
     "value2":  "BBB",
   },
}
```
