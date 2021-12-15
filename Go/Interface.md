# Interface

Go에서 Interface는 Method의 집합이다...!!!

뭐 맞는 말이지만 틀린 말이기도 하다. Interface에는 Method만 들어가는 것이 아니라 Interface도 들어 갈수도 있다. 그리고 Method가 없는 빈 Interface도 존재한다.

`Interface는 Object의 Method를 연결해주는 역활을 한다...!!!`

Go에서는 OOPL처럼 Object라는 개념이 없는데 뭔 개소리(?)냐고 하는 분들이 있을수 있다. 이런 분들에게는 Struct를 Object처럼 봐 달라고 부탁하고 싶다. 실제로 Struct를 사용하다 보면 '어... 이거 Object와 똑같네...!!!' 라고 느낄 것이다.

1. [일반적인 Interface 사용 방법](#일반적인-interface-사용-방법)

1. [Interface를 Parameter로 전달해야 하는 경우](#interface를-parameter로-전달해야-하는-경우)

1. [여러개의 Interface에 Struce를 전달하는 경우](#여러개의-interface에-struce를-전달하는-경우)

1. [Interface의 Cascading...???](#interface의-cascading)

1. [Interface에서 또 다른 Intercase로 Cascading하는 경우](#interface에서-또-다른-intercase로-cascading하는-경우)

1. [Interface의 타입 구분](#interface의-타입-구분)

1. [Interface내 Interface](#interface내-interface)

1. [Interface에게 Method전달](#Interface에게-Method전달)

1. [Multy Interface Parameter](#multy-interface-parameter)

## 일반적인 Interface 사용 방법

```go
package main

import (
	"fmt"Interface를 Parameter로 전달해야 하는 경우
	"math"
)

type Shape interface {
	Area() float64
	Perimeter() float64
}

type Rect struct {
	width  float64
	height float64
}

func (r Rect) Area() float64 {
	return r.width * r.height
}

func (r Rect) Perimeter() float64 {
	return 2 * (r.width + r.height)
}

type Circle struct {
	radius float64
}

func (c Circle) Area() float64 {
	return math.Pi * c.radius * c.radius
}

func (c Circle) Perimeter() float64 {
	return 2 * math.Pi * c.radius
}

func main() {
	// interface 동적 타입, 동적 값
	var s Shape = Rect{10.0, 3.0}

	fmt.Printf("type of s is %T\n", s)
	fmt.Printf("value of s is %v\n", s)
	fmt.Printf("Area of s is %0.2f\n", s.Area())
	fmt.Printf("Perimeter of s is %0.2f\n\n", s.Perimeter())

	s = Circle{10.0}
	fmt.Printf("type of s is %T\n", s)
	fmt.Printf("value of s is %v\n", s)
	fmt.Printf("Area of s is %0.2f\n", s.Area())
	fmt.Printf("Perimeter of s is %0.2f\n\n", s.Perimeter())

	// 포인터를 전달하지 않아도 메소드를 전달 받을수 있다.
	a := s.Area()
	p := s.Perimeter()
	fmt.Printf("Area of s is %0.2f\n", a)
	fmt.Printf("Perimeter of s is %0.2f\n", p)

	// type of s is main.Rect
	// value of s is {10 3}
	// value of s is 30.00
	// Perimeter of s is 26.00
	//
	// type of s is main.Circle
	// value of s is {10}
	// value of s is 314.16
	// Perimeter of s is 62.83
}
```
Rect과 Circle를 OOPL의 Class로 인식해 주었으면 한다. 그리고 Class에서 사용할 Method를 정의하게 되는데 이 부분을 Go에서는 Receiver를 사용해서 누구의 Method인지 지정을 해준다.

```go
func (r Rect) Area() float64 {
	// [...]
}
```
`(r Rect)`가 Receiver라고 불리는 부분이다.

참고로 위 Sample이 Go의 일반적인 Sample인 것처럼 보이지만 보통 Receiver는 Point Receiver를 주로 사용한다.

```go
func (r *Rect) Area() float64 {
	// [...]
}
```
`(r *Rect)`처럼 `*`를 사용해서 Value Receiver가 아닌 Point Receiver라는 것을 지정해 준다.

Sample에서는 Receiver가 변경될 일이 없기 때문에 Pointer를 사용하지 않았지만 OOPL의 Get, Set기능을 구현하고자 한다면 Pointer를 사용해야 되고 아래 Sample처럼 변경하면 된다.

```go
package main

import (
	"fmt"
	"math"
)

type Shape interface {
	Area() float64
	Perimeter() float64
}

type Rect struct {
	width  float64
	height float64
}

func (r *Rect) Area() float64 {
	return r.width * r.height
}

func (r *Rect) Perimeter() float64 {
	return 2 * (r.width + r.height)
}

type Circle struct {
	radius float64
}

func (c *Circle) Area() float64 {
	return math.Pi * c.radius * c.radius
}

func (c *Circle) Perimeter() float64 {
	return 2 * math.Pi * c.radius
}

func main() {
	// &를 사용해서 포인터를 전달
	var s Shape = &Rect{10.0, 3.0}

	fmt.Printf("type of s is %T\n", s)
	fmt.Printf("value of s is %v\n", s)
	fmt.Printf("Area of s is %0.2f\n", s.Area())
	fmt.Printf("Perimeter of s is %0.2f\n\n", s.Perimeter())

	// &를 사용해서 포인터를 전달
	s = &Circle{10.0}
	fmt.Printf("type of s is %T\n", s)
	fmt.Printf("value of s is %v\n", s)
	fmt.Printf("Area of s is %0.2f\n", s.Area())
	fmt.Printf("Perimeter of s is %0.2f\n\n", s.Perimeter())

	// 포인터를 전달하지 않아도 메소드를 전달 받을수 있다.
	a := s.Area()
	p := s.Perimeter()
	fmt.Printf("Area of s is %0.2f\n", a)
	fmt.Printf("Perimeter of s is %0.2f\n", p)

	// type of s is main.Rect
	// value of s is {10 3}
	// value of s is 30.00
	// Perimeter of s is 26.00
	//
	// type of s is main.Circle
	// value of s is {10}
	// value of s is 314.16
	// Perimeter of s is 62.83
}
```
처음 Go를 접하는 분들에게는 어려울 수 있는 Sample이지만 최대한 OOPL적으로 생각하면 뭐 그리 어렵지 않은 Source가 될 것이다.

`&Rect{10.0, 3.0}` 이 부분은 Instance생성이라 이해하면 된다. `&`는 Pointer Receiver를 사용하였기 때문에 Address를 전달해주기 위해 사용이 되었다.

이후 부터는 Interface에 기술된 Method를 사용하기만 하면 된다. Source에서 처럼 어떤 Struct가 넘어 왔는지는 무시해도 관련 Method를 호출해 준다.

닭이 울때는 '꼬끼오'라고 울고 오리가 울때는 '꽥꽥'하고 운다. 대상에 따라 우는 방법이 달라지지만 Interface를 사용하면 대상이 무엇이든 상관없이 운다는 Method만 집중하면 되는 것이다.

## Interface를 Parameter로 전달해야 하는 경우

```go
package main

import "fmt"

type MyString string

type Rect struct {
	width  float64
	height float64
}

func (r *Rect) Area() float64 {
	return r.width * r.height
}

// 빈 인터페이스에 파라미터를 받아서 처리.
func explain(i interface{}) {
	// 빈 인터페이스를 전달 받아서 그런지 할수 있는 것이 없다.
	fmt.Printf("type '%T', value %v\n", i, i)
}

func main() {
	// 새로운 타입을 만든 후 사용 방법...
	ms := MyString("Hello World...!!!")
	r := Rect{5.5, 4.5}

	explain(ms)
	explain(r)

	explain(r.Area())

	// type 'main.MyString', value Hello World...!!!
	// type 'main.Rect', value {5.5 4.5}
	// type 'float64', value 24.75
}
```
빈 Interface를 활용하는 방식으로 Sample은 단순해도 활용도가 무척 높은 코드중 하나이다. 완전히 숙지학길 바란다.

Interface를 따로 선언하지 않고 Parameter로 사용되었다. 위에서 Sample에서 전달했듯이 Interface를 사용하면 전달된 대상은 무시하고 Method에 집중을 할수 있다고 했다.

마치 블랙홀처럼 Struct뿐만 아니라 Method도 바로 바로 전달해서 사용할수 있다는 점이 Interface의 매력인 것 같다.

## 여러개의 Interface에 Struce를 전달하는 경우

```go
package main

import "fmt"

type Shape interface {
	Area() float64
}

type Object interface {
	Volume() float64
}

type Cude struct {
	side float64
}

func (c *Cude) Area() float64 {
	return 6 * (c.side * c.side)
}

func (c *Cude) Volume() float64 {
	return c.side * c.side * c.side
}

func main() {
	c := Cude{3}

	var s Shape = &c
	var o Object = &c
	fmt.Println("Interface Area is", s.Area())
	fmt.Println("Interface Volume is", o.Volume())

	// Shape에는 Volume이 없고... Object에는 Area가 없어서 에러...
	//fmt.Println("Interface Area is", s.Volume())
	//fmt.Println("Interface Volume is", o.Area())

	// 단순하게 sturct를 사용하면 호출이 가능하다.
	fmt.Println("Struct Area is", c.Area())
	fmt.Println("Struct Volume is", c.Volume())

	// Interface Area is 54
	// Interface Volume is 27

	// Struct Area is 54
	// Struct Volume is 27
}
```
한개의 Struct를 여러개의 Interface에 전달하는 Sample로 중요하게 봐야 하는 부분은 Interface별로 각각 다른 한개의 Method만 정의 되었다는 것이다.

Interface에 정의된 Method는 제대로 작동이 되지만 정의되어 있지 않은 Method는 호출이 불가능하다.

Sample에서는 Interface를 사용하지 않고 Struct에서 바로 호출도 가능하지만 우리가 원한 Interface를 활용하기 위한 방법과는 거리가 있다.

## Interface의 Cascading...???

```go
package main

import "fmt"

type Shape interface {
}

type Cube struct {
	side float64
}

func (c *Cube) Area() float64 {
	return 6 * (c.side * c.side)
}

func (c *Cube) Volume() float64 {
	return c.side * c.side * c.side
}

func main() {
	var s Shape = &Cube{3}
	// fmt.Println("Area is", s.Area()) // 에러 발생
	// fmt.Println("Volume is", s.Volume()) // 에러 발생

	c, ok := s.(*Cube)
	if !ok {
		fmt.Println("Error...!!!")
	}
	fmt.Println("Area is", c.Area())
	fmt.Println("Volume is", c.Volume())

	fmt.Printf("%T", *c)

	// Area is 54
	// Volume is 27
	// main.Cube
}
```
Sample에선 빈 Interface를 만들고 이곳에 'Cube' Struct를 활당했다. 그리고 Comment처럼 Interface에 정의되지 않은 Method를 호출하는 경우는 에러가 발생된다.

하지만 Interface를 통해 Struct를 다시 전달 받아 사용이 가능하다. 

실무에서 활용도가 높은 Sample이다. 꼭 숙지를 해주길 바란다.

## Interface에서 또 다른 Intercase로 Cascading하는 경우 

```go
package main

import "fmt"

type Shape interface {
	Area() float64
}

type Object interface {
	Volume() float64
}

type Skin interface {
	Color() float64
}

type Cude struct {
	side float64
}

func (c *Cude) Area() float64 {
	return 6 * (c.side * c.side)
}

func (c *Cude) Volume() float64 {
	return c.side * c.side * c.side
}

func main() {
	var s Shape = &Cude{3}

	// Interface를 통해 인터페이스를 리턴 받는데 전달받은
	// Interface에 메소드[Area() 또는 Volume()]가 선언되어 있으면 문제가 없지만.
	// value1, ok1 := s.(*Object) // 에러 발생...
	value1, ok1 := s.(Object)
	fmt.Printf("value %v, %v\n", value1, ok1)

	// Color() 메소드는 없기 때문에 인터페이스를 전달받지 못한다.
	value2, ok2 := s.(Skin)
	fmt.Printf("value %v, %v\n", value2, ok2)

	// value {3}, true
	// value <nil>, false
}
```
바로 전 Sample은 Interface => Struct로 Cascading되는 것이고 이번에는 Interface => Interface로 Cascading되는 Sample이다.

혼돈이 생길지 모르겠지만 Interface에 전달된 Struct을 다른 Interface에 간접적으로 활당한다고 생각하면 그리 어렵지 않을 것이다.

이전 Sample에서 확인했듯이 전달 받은 Interface 관련 Method를 지원하냐 안하냐의 문제처럼 Skin으로 Cascading된 경우에는 아무것도 전달을 받지 못한다는 것을 확인할 수 있다.

참고로 Struct를 전달 받지 않기 때문에 *를 사용하지 않는다는 것과 Pointer형식으로 전달받게 된다는 것이다.

## Interface의 타입 구분

```go
package main

import (
	"fmt"
	"strings"
)

func explain(i interface{}) {
	switch i.(type) {
	case string:
		fmt.Println("String", strings.ToUpper(i.(string)), i)
	case int:
		fmt.Println("Int", i.(int), i)
	default:
		fmt.Println("Something", i.(bool), i)
	}
}

func main() {
	// 빈 인터페이스 파라미터를 전달하고 타입을 분석해서 출력하는 부분이다.
	// 향후 전달해야 되는 객체에 따라 다른게 행동해야 하는 경우가 생기면 활용할수 있는 코드다.
	explain("Hello World...!!!")
	explain(53)
	explain(true)

	// String HELLO WORLD...!!! Hello World...!!!
	// Int 53 53
	// Something true true
}
```

단순한 Sample이지만 실무에서 많이 사용되는 방식의 Interface활용 방법이다. 너무 단순해서 설명은 생략한다.

### Interface내 Interface
```go
package main

import (
	"fmt"
)

type Shape interface {
	Area() float64
}

type Object interface {
	Volume() float64
}

type Material interface {
	Shape
	Object
}

type Cude struct {
	side float64
}

func (c *Cude) Area() float64 {
	return 6 * (c.side * c.side)
}

func (c *Cude) Volume() float64 {
	return c.side * c.side * c.side
}

func main() {
	c := &Cude{3}
	// 인터페이스의 인터페이스...
	var m Material = c
	var s Shape = c
	var o Object = c

	fmt.Printf("type: %T, value: %v\n", m, m)
	fmt.Printf("type: %T, value: %v\n", s, s)
	fmt.Printf("type: %T, value: %v\n", o, o)

	fmt.Printf("type: %T, value: %v m.Area: %0.2f\n", m, m, m.Area())
	fmt.Printf("type: %T, value: %v m.Volume: %0.2f\n", m, m, m.Area())
	fmt.Printf("type: %T, value: %v s.Area: %0.2f\n", s, s, s.Area())
	fmt.Printf("type: %T, value: %v o.Volume: %0.2f\n", o, o, o.Volume())

	// 역시 인터페이스내에 메소드가 정의되어 있지 않기 때문에 에러...
	//fmt.Printf("type: %T, value: %v s.Area: %0.2f\n", s, s, s.Volume())
	//fmt.Printf("type: %T, value: %v o.Volume: %0.2f\n", o, o, o.Area())

	// type: main.Cude, value: {3}
	// type: main.Cude, value: {3}
	// type: main.Cude, value: {3}
	// type: main.Cude, value: {3} m.Area: 54.00
	// type: main.Cude, value: {3} m.Volume: 54.00
	// type: main.Cude, value: {3} s.Area: 54.00
	// type: main.Cude, value: {3} o.Volume: 27.00
}
```
Interface내에 Interface가 있다고 너무 고민하지 않아도 된다. 'Material'처럼 선언해도 결국은...

```go
type Material interface {
	Area() float64
	Volume() float64
}
```

이렇게 선언해준 것과 별 차이가 없다.

### Interface에게 Method전달
```go
package main

import (
	"fmt"
)

type Shape interface {
	Area() float64
	Perimeter() float64
}

type Rect struct {
	width  float64
	height float64
}

func (r *Rect) Area() float64 {
	return r.width * r.height
}

func (r *Rect) Perimeter() float64 {
	return 2 * (r.width + r.height)
}

func main() {
	r := Rect{5.0, 4.0}
	var s Shape = &r
	area := s.Area()
	perimeter := s.Perimeter()
	fmt.Println("Area is", area)
	fmt.Println("Perimeter is", perimeter)

	// Area is 20
	// Perimeter is 18
}
```
Sample 단순해서 설명할 내용이 없다. Interface를 통해 Method를 호출하듯이 Method를 변수로 전달 받아 사용할 수 있다는 것을 보여주는 Sample이다.

## Multy Interface Parameter

```go
package main

import (
	"fmt"
	"math"
)

type geometry interface {
	area() float64
	perim() float64
}

type rect struct {
	width, height float64
}

func (r *rect) area() float64 {
	return r.width * r.height
}

func (r *rect) perim() float64 {
	return 2*r.width + 2*r.height
}

type circle struct {
	radius float64
}

func (c *circle) area() float64 {
	return math.Pi * c.radius * c.radius
}

func (c *circle) perim() float64 {
	return 2 * math.Pi * c.radius
}

// 포인터를 다중으로 전달하더라도 '*'는 사용하지 않는다.
// 즉 인터페이스는 자체가 포인터 역활을 하고 있다는 것인가...???
func measure(g geometry) {
	fmt.Println(g)
	fmt.Println(g.area())
	fmt.Println(g.perim())
	fmt.Println()
}

func showArea(geo ...geometry) {
	for _, s := range geo {
		a := s.area()
		fmt.Println(a)
	}
	fmt.Println()
}

func main() {
	r := rect{width: 3, height: 4}
	c := circle{radius: 5}

	measure(&r) // {3 4}, 12, 14 출력
	measure(&c) // {5}, 78.53981633974483, 31.41592653589793 출력

	showArea(&r, &c) // 12, 78.53981633974483 출력

	// &{3 4}
	// 12
	// 14
	//
	// &{5}
	// 78.53981633974483
	// 31.41592653589793
	//
	// 12
	// 78.53981633974483
}
```
이 Sample의 핵심은 이 부분일 것이다.

```go
func showArea(geo ...geometry) {
	for _, s := range geo {
		a := s.area()
		fmt.Println(a)
	}
}
```
여러개의 Struct를 Interface로 한번에 전달 받아 처리를 하는 방법으로 실무에서 많이 활용되는 방식이다.

참고로 Go의 fmt.Println은 아래와 같이 선언되어 있다.

```go
func Println(a ...interface{}) (n int, err error) {
	return Fprintln(os.Stdout, a...)
}
```