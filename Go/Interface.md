Interface 연구
-----
Go에서 Interface는 Method의 집합이다...???

뭐 맞는 말이지만 틀린 말이기도 하다. Interface에는 Method만 들어가는 것이 아니라 Interface도 들어 갈수도 있다. 그리고 Method가 없는 빈 Interface도 존재한다.

Interface는 Object의 Method를 연결해주는 역활을 한다. Go에서는 OOPL처럼 Object라는 개념이 없는 뭔 개소리(?)냐고 하는 분들이 있을수 있다. 이런 분들에게는 Struct를 Object처럼 봐 달라고 부탁하고 싶다. 실제로 Struct를 사용하다 보면 '어... 이거 Object와 똑같네...!!!' 라고 느낄 것이다.

### 일반적인 Interface 사용 방법
<pre>
<code>
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

</code>
</pre>

Rect과 Circle를 OOPL의 Class로 인식해 주었으면 한다. 그리고 Class에서 사용할 Method를 정의하게 되는데 이 부분을 Go에서는 Receiver를 사용해서 누구의 Method인지 지정을 해준다.

<pre>
<code>
func <u>(r Rect)</u> Area() float64 {
	// [...]
}
</code>
</pre>

(r Rect)가 Receiver라고 불리는 부분이다.

참고로 위 Sample이 일반적인 Go Sample인 것처럼 보이지만 보통 Receiver를 사용하는 경우에는 Pointer Receiver를 주로 사용한다.

<pre>
<code>
func <u>(r *Rect)</u> Area() float64 {
	// [...]
}
</code>
</pre>

(r *Rect)처럼 *를 사용해서 Receiver가 Call by Value가 아닌 Call by Reference라는 것을 지정해 준다.

Sample에서는 Receiver가 변경될 일이 없기 때문에 Pointer를 사용하지 않았지만 OOPL의 Get, Set기능을 구현하고자 한다면 Pointer를 사용해야 되고 아래 Sample처럼 변경하면 된다.

<pre>
<code>
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
</code>
</pre>
처음 Go를 접하는 분들에게는 어려울 수 있는 Sample이지만 최대한 OOPL적으로 생각하면 뭐 그리 어렵지 않은 Source가 될 것이다.

&Rect{10.0, 3.0} 이 부분은 Instance생성이라 이해하면 된다. &는 Pointer Receiver를 사용하였기 때문에 Address를 전달해주기 위해 사용이 되었다.

이후 부터는 Interface에 기술된 Method를 사용하기만 하면 된다. Source에서 처럼 어떤 Struct가 넘어 왔는지는 무시해도 관련 Method를 호출해 준다.

닭이 울때는 '꼬끼오'라고 하고 오리가 울때는 '꽥꽥'이라고 하지만 Interface를 사용하면 대상이 무엇이든 상관없이 운다는 Method만 집중하면 되는 것이다.

### Interface를 Parameter로 전달해야 하는 경우
<pre>
<code>
package main

import "fmt"

type MyString string

type Rect struct {
	width  float64
	height float64
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

	// type 'main.MyString', value Hello World...!!!
	// type 'main.Rect', value {5.5 4.5}
}
</code>
</pre>
Sample은 단순해도 활용도가 무척 높은 코드중 하나이다. 완전히 숙지학길 바란다.

### Multi Interface를 사용하는 경우
<pre>
<code>
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

func (c Cude) Area() float64 {
	return 6 * (c.side * c.side)
}

func (c Cude) Volume() float64 {
	return c.side * c.side * c.side
}

func main() {
	// 멀티 interface
	c := Cude{3}

	var s Shape = c
	var o Object = c
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
</code>
</pre>
Multi Interface을 사용하는 Sample이다. 여기서 중요하게 봐야 할 부분이 에러가 발생하는 부분으로 Interface에 정의가 되어 있지 않은 Function은 호출을 할 수 없다.

### Multi Interface에서 발생된 문제점을 해결해 보자.
<pre>
<code>
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
	// Interface에서 Struct를 리턴받아(캐스케이딩) 사용하는 방법.
	var s Shape = &Cube{3}
	c, ok := s.(*Cube)
	if !ok {
		fmt.Println("Error...!!!")
	}
	fmt.Println("Area is", c.Area())
	fmt.Println("Volume is", c.Volume())

	// Area is 54
	// Volume is 27
}
</code>
</pre>
Pointer와 친해지기 위해 Sample에 Pointer를 사용했다. 실무에서는 Pointer를 사용해야 하는 경우가 많이 발생되기 때문에 미리 친해지는 것도 나쁘지 않을 것이다.

### Interface에서 또 다른 Intercase로 Cascading하는 경우 
<pre>
<code>
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
</code>
</pre>
Interface에서 다른 형태의 Interface로 Cascading해서 전달을 반드는 경우이다. 여기서 중요한 것은 Struct를 전달 받지 않기 때문에 *를 사용하지 않는다는 것과 Pointer로 전달받게 된다는 것이다.
*를 사용하면 에러가 발생된다.

### Interface의 타입 구분
<pre>
<code>
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
</code>
</pre>
Interface를 Parameter로 전달 받은 다음 Type을 구분해서 처리할 방식을 다르게 할 수 있다.