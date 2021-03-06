#### Closure

Closure(클로저) 란 __두 개의 함수로 만들어진 환경__ 으로 이루어진 특별한 객체의 한 종류이다. 여기서 __환경__ 이라 함은 클로저가 생성될 때 그 __범위__에 있던 여러 지역 변수들이 포함된 __context__를 말한다. 이 클로저를 통해서 자바스크립트에는 없는 비공개(private) 속성/메소스, 공개 속성/메소드를 구현할 수 있는 방안을 마련할 수 있다.

다음은 클로저가 생성되는 조건이다.

1. 내부 함수가 익명 함수로 되어 외부 함수의 반환값으로 사용된다.
2. 내부 함수는 외부 함수의 실행 환경(execution environment)에서 실행된다.
3. 내부 함수에서 사용되는 변수 x는 외부 함수의 변수 스코프에 있다.

```javascript
function outer() {
    var name = `closure`;
    function inner() {
        console.log(name);
    }
    inner();
}
outer();
// console > closure
```

`outer`함수를 실행시키는 `context`에는 `name`이라는 변수가 존재하지 않는다는 것을 확인할 수 있다. 비슷한 맥락으로 다음과 같이 변경해 볼 수 있다.

```javascript
var name = `Warning`;
function outer() {
    var name = `closure`;
    return function inner() {
        console.log(name);
    };
}

car callFunc = outer();
callFunc();
// console > closure
```

위 코드에서 `callFunc`를 클로저라고 한다. `callFunc` 호출에 의해 `name` 이라는 값이 console에 찍히는데, 찍히는 값은 `Warning`이 아니라 `closure` 라는 값이다. 즉, `outer` 함수의 context에 속해있는 변수를 참조하는 것이다. 여기서 `outer` 함수의 지역변수로 존재하는 `name`변수를 `free variable(자유변수)`라고 한다.







# this에 대해서

자바스크립트에서 모든 함수는 실행될 때마다 함수 내부에 `this`라는 객체가 추가된다. `arguments`라는 유사 배열 객체와 함게 함수 내부로 암묵적으로 전달되는 것읻. 그렇기 때문에 자바스크립트에서의 `this`는 함수가 호출된 상황에 따라 그 모습을 달리한다.

##### case 1. 객체의 메서드를 호출할 때

객체의 프로퍼티가 함수일 경우 메서드라고 부른다. `this`는 함수를 실행할 때 함수를 소유하고 있는 객체(메소드를 포함하고 있는 인스턴스)를 참조한다. 즉 해당 메서드를 호출한 객체로 바인딩 된다. `A.B`일 때 `B`함수 냅부에서의 `this`는 `A`를 가리키는 것이다.

```javascript
var myObject = {
    name = "foo",
    sayName : function() {
        console.log(this);
    }
};
myObject.sayName();
// console > Object {name: "foo" , sayName: sayName()}
```

##### case 2. 함수를 호출할 때

특정 객체의 메서드가 아니라 함수를 호출하면, 해당 함수 내부 코드에서 사용된 this는 전역 객체에 바인딩 된다. `A.B`일 때 `A`가 전역 객체가 되므로 `B`함수 내부에서의 `this`는 당연히 전역 객체에 바인딩 되는 것이다.

```javascript
var value = 100;
var myObj = {
  value: 1,
  func1: function() {
    console.log(`func1's this.value: ${this.value}`);

    var func2 = function() {
      console.log(`func2's this.value ${this.value}`);
    };
    func2();
  }
};

myObj.func1();
// console> func1's this.value: 1
// console> func2's this.value: 100
```

`fucn1`에서의 `this`는 __상황1__과 같다. 그렇기 때문에 `myObj`가 `this`로 바인딩되고 `myObj`의 `value`인 1이 console에 찍히게 된다. 하지만 `func2`는 __상황2__로 해석해야 한다. `A.B`구조에서 `A`가 없기 때문에 함수 내부에서 `this`가 전역 객체를 참조하게 되고 `value`는 100이 되는 것이다.



##### case 3. 생성자 함수를 통해 객체를 생성할 때

그냥 함수를 호출하는 것이 아니라 `new`키워드를 통해 생성자 함수를 호출할 때는 또 `this`가 다르게 바인딩 된다. `new`키워드를 통해서 호출된 함수 내부에서의 `this`는 객체 자신이 된다. 생성자 함수를 호출할 때의 `this` 바인딩은 생성자 함수가 동작하는 방식을 통해 이해할 수 있다.

`new`연산자를 통해 함수를 생성자로 호출하게 되면, 일단 빈 객체가 생성되고 this 가 바인딩 된다. 이 객체는 함수를 통해 생성된 객체이며, 자신의 부모인 프로토타입 객체와 연결되어 있다. 그리고 return 문이 명시되어 있지 않은 경우에는 `this`로 바인딩 된 새로 생성한 객체가 리턴된다.

```javascript
var Person = function(name) {
  console.log(this);
  this.name = name;
};

var foo = new Person("foo"); // Person
console.log(foo.name); // foo
```



##### case 4. apply, call, bind 를 통한 호출

상황1, 상황2, 상황3 에 의존하지 않고 `this`를 자바스크립트 코드로 주입 또는 설정할 수 있는 방법이다. 상황 2 에서 사용했던 예제 코드를 다시 한 번 보고 오자. `func2`를 호출할 때, `func1`에서의 this를 주입하기 위해서 위 세가지 메소드를 사용할 수 있다. 그리고 세 메소드의 차이점을 파악하기 위해 `func2`에 파라미터를 받을 수 있도록 수정한다.

- `bind`메소드 사용

```javascript
var value = 100;
var myObj = {
  value: 1,
  func1: function() {
    console.log(`func1's this.value: ${this.value}`);

    var func2 = function(val1, val2) {
      console.log(`func2's this.value ${this.value} and ${val1} and ${val2}`);
    }.bind(this, `param1`, `param2`);
    func2();
  }
};

myObj.func1();
// console> func1's this.value: 1
// console> func2's this.value: 1 and param1 and param2
```

- `call` 메소드 사용

```javascript
var value = 100;
var myObj = {
  value: 1,
  func1: function() {
    console.log(`func1's this.value: ${this.value}`);

    var func2 = function(val1, val2) {
      console.log(`func2's this.value ${this.value} and ${val1} and ${val2}`);
    };
    func2.call(this, `param1`, `param2`);
  }
};

myObj.func1();
// console> func1's this.value: 1
// console> func2's this.value: 1 and param1 and param2
```

- `apply` 메소드 사용

```javascript
var value = 100;
var myObj = {
  value: 1,
  func1: function() {
    console.log(`func1's this.value: ${this.value}`);

    var func2 = function(val1, val2) {
      console.log(`func2's this.value ${this.value} and ${val1} and ${val2}`);
    };
    func2.apply(this, [`param1`, `param2`]);
  }
};

myObj.func1();
// console> func1's this.value: 1
// console> func2's this.value: 1 and param1 and param2
```

- `bind` vs `apply`, `call` : 우선 `bind`는 함수를 선언할 때, `this`와 파라미터를 지정해줄 수 있으며, `call`과 `apply`는 함수를 호출할 때, `this`와 파라미터를 지정해준다.
- `apply` vs `bind`, `call` : `apply` 메소드에는 첫번째 인자로 `this`를 넘겨주고 두번째 인자로 넘겨줘야 하는 파라미터를 배열의 형태로 전달한다. `bind` 메소드와 `call` 메소드는 각각의 파라미터를 하나씩 넘겨주는 형태이다.



# Arrow Function

화살표 함수 표현식은 기존의 function 표현방식보다 간결하게 함수를 표현 가능. 화살표 함수는 항상 익명이며, 자신의 this, arguments, super 그리고 new.target을 바인딩하지 않는다. 따라서 생성자로는 사용할 수 없다.

- 화살표 함수 도입 영향 : 짧은 함수, 상위 스코프 this

- 짧은 함수

``` javascript
var materials = [
  'Hydrogen',
  'Helium',
  'Lithium',
  'Beryllium'
];

materials.map(function(material) { 
  return material.length; 
}); // [8, 6, 7, 9]

materials.map((material) => {
  return material.length;
}); // [8, 6, 7, 9]

materials.map(({length}) => length); // [8, 6, 7, 9]
```

기존의 function을 생략 후 => 로 대체 표현

- 상위 스코프 this

```javascript
function Person(){
  this.age = 0;

  setInterval(() => {
    this.age++; // |this|는 person 객체를 참조
  }, 1000);
}

var p = new Person();
```

일반 함수에서 this는 자기 자신을 this로 정의한다. 하지만 화살표 함수 this는 Person의 this와 동일한 값을 갖는다. setInterval로 전달된 this 는 Person 의 this를 가리키며, Person 객체의 age에 접근한다.
