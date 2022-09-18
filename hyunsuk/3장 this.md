# 3장 this

## 3.1 상황에 따라 달라지는 this

- 자바스크립트에서 this는 기본적으로 실행 컨텍스트가 생성될 때 함께 결정된다.

- 실행 컨텍스트는 함수를 호출할 때 생성되므로, this는 함수를 호출할 때 결정된다.

### 3.1.1 전역 공간에서의 this

- 전역 공간에서 this는 전역 객체를 가리킨다.

- 브라우저 전역객체는 window이고 Node.js는 global이다.

### 📌 브라우저 환경에서 전역 공간에서의 this

```Js
console.log(this);	// { alert: f(), atob: f(), blur: f(), btoa: f(), ... }
console.log(window);	// { alert: f(), atob: f(), blur: f(), btoa: f(), ... }
console.log(this === window);  // true
```

- 자바스크립트의 모든 변수는 실은 실행 컨텍스트의 LexicalEnvironment로서 동작하기 때문입니다

- 전역변수를 선언하면 자바스크립트 엔진은 이를 전역객체의 프로퍼티로 할당한다

- 단순하게 (window.)이 생략된 것이라고 여겨도 무방하다.

- 전역변수를 선언하면 자바스크립트 엔진이 이를 자동으로 전역객체의 프로퍼티로 할당하면서 추가적으로 해당 프로퍼티의 configurable 속성(변경 및 삭제 가능성)을 false로 정의한다.

```Js
var a = 1;
console.log(a) // 1
console.log(window.a) // 1
console.log(this.a) // 1
```

### 3.1.2 메서드로서 호출할 때 그 메서드 내부에서의 this

### 🔍 함수 vs 메서드

- 함수와 메서드는 미리 정의한 동작을 수행하는 코드 뭉치로, 이 둘을 구분하는 유일한 차이는 독립성이다.

- 함수는 그 자체로 독립적인 기능을 수행하는 반면, 메서드는 자신을 호출한 대상 객체에 관한 동작을 수행한다.

- 어떤 함수를 객체의 프로퍼티에 할당한다고 해서 그 자체로서 무조건 메서드가 되는 것이 아니라 객체의 메서드로서 호출할 경우에만 메서드로 동작하고, 그렇지 않으면 함수로 동작한다.

```Js
let func = function (x){
    console.log(this,x);
};
func(1) // window {...} 1

let obj = {
    method : func
};
obj.method(2) // { method : f } 2

```

- 점 표기법이든 대괄호 표기법이든, 어떤 함수를 호출할 때 그 함수 이름(프로퍼티명) 앞에 객체가 명시돼 있는 경우에는 메서드로 호출한 것이고, 그렇지 않은 모든 경우에는 함수로 호출한 것이다.

```Js
let obj = {
    method : function (x) { console.log(this,x); }
};
obj.method(1); // { method : f } 1
obj['method'](2); // { method : f } 2

```

### 🔍 메서드 내부에서의 this

- this에는 호출한 주체에 대한 정보가 담긴다.

- 어떤 함수를 메서드로서 호출하는 경우 호출 주체는 바로 함수명(프로퍼티명) 앞의 객체이다.

```Js
let obj = {
    methodA : function () { console.log(this); },
    inner : {
        methodB : function () { console.log(this); }
    }
};

obj.methodA(); // { methodA: f, inner: {...} }  ( === obj)
obj['methodA'](); // { methodA: f, inner: {...} }  ( === obj)

obj.inner.methodB(); // { methodB: f }   ( === obj.inner)
obj.inner['methodB'](); // { methodB: f }   ( === obj.inner)
obj['inner'].methodB(); // { methodB: f }   ( === obj.inner)
obj['inner']['methodB'](); // { methodB: f }   ( === obj.inner)
```

### 3.1.3 함수로서 호출할 때 그 함수 내부에서의 this

### 🔍 함수 내부에서의 this

- 함수에서의 this는 전역 객체이다.

### 🔍 메서드의 내부함수에서의 this

- this 바인딩에 관해서는 함수를 실행하는 당시의 주변 환경(메서드 내부인지, 함수 내부인지 등)은 중요하지 않고, 오직 해당 함수를 호출하는 구문 앞에 `점 또는 대괄호 표기`가 있는지 없는지가 중요하다.

```Js
var obj1 = {
    outer : function () {
        console.log(this); // (1) obj1
        var innerFunc = function () {
            console.log(this) // (2) window
        }
        innerFunc(); // (2) 앞에 점이 없으므로 this는 window

        var obj2 = {
            innerMethod : innerFunc  // (3) obj2
        };
        ob2.innerMethod(); // (3) 앞에 점이 있으니 obj2
    }
};
obj1.outer();
```

### 🔍 메서드의 내부 함수에서의 this를 우회하는 방법

- 변수를 활용하는 방법

- outer 스코프에서 self라는 변수에 this를 저장한 상태에서 호출한 innerFunc2의 경우 self에는 객체 obj가 출력된다.

```Js
var obj1 = {
    outer : function () {
        console.log(this); // (1) obj1
        var innerFunc = function () {
            console.log(this) // (2) window
        };
        innerFunc();

        var self = this;
        var innerFunc2 = function () {
            console.log(self);  // (3) obj 1
        };
        innerFunc2();
    }
};
obj1.outer();
```

### 🔍 this를 바인딩하지 않는 함수

- 화살표 함수는 실행 컨텍스트를 생성할 때 this 바인딩 과정 자체가 빠지게 되어, 상위 스코프의 this를 그대로 활용할 수 있다.

- call, apply 등의 메서드를 활용해 함수를 호출할 때 명시적으로 this를 지정하는 방법이 있다.

```Js
let obj1 = {
    outer : function () {
        console.log(this); // (1) obj 1
        let innerFunc = () => {
            console.log(this) // (2) obj 1
        };
        innerFunc(); // (2) 앞에 점이 없으므로 this는 window
    }
};
obj1.outer();
```

### 3.1.4 콜백 함수 호출 시 그 함수 내부에서의 this

- 콜백 함수의 제어권을 가지는 함수(메서드)가 콜백 함수에서의 this를 무엇으로 할지를 결정하며, 특별히 정의하지 않은 경우에는 기본적으로 함수와 마찬가지로 전역객체이다.

```Js
setTimeout(function () { console.log(this); }, 300); // (1) 전역객체

[1, 2, 3, 4, 5].forEach(function (x) { // (2) 전역객체
    console.log(this);
});

document.body.innerHTML += '<button id="a">클릭</button>';
document.body.querySelector('#a')
    .addEventListener('click', function (e) { // (3) 앞서 지정한 엘리먼트
        console.log(this);
});
```

### 3.1.5 생성자 함수 내부에서의 this

- new 명령어와 함께 함수를 호출하면 해당 함수가 생성자로서 동작한다.

- 어떤 함수가 생성자 함수로서 호출된 경우 내부에서의 this는 곧 새로 만들 구체적인 인스턴스 자신이 된다.

- 예시에서 실행한 생성자 함수 내부에서의 this는 choco 인스턴스를 가리킨다.

```Js
let Cat = function(name,age) {
    this.bark = "야옹";
    this.name = name;
    this.age = age;
};

let choco = new Cat("초코",7);
let nabi = new Cat("나비",5);
console.log(choco,nabi)

/* 결과
Cat { bark: '야옹', name: '초코', age: 7 }
Cat { bark: '야옹', name: '나비', age: 5 }
*/
```

## 3.2 명시적으로 this를 바인딩하는 방법

### 3.2.1 call 메서드

```Js
Function.prototype.call(thisArg[, arg1[, arg2[, ...]]])
```

- call 메서드는 메서드의 호출 주체인 함수를 즉시 실행하도록 하는 명령

- call 메서드의 첫 번째 인자를 this로 바인딩하고, 이후의 인자들을 호출할 함수의 매개변수로 한다.

- 함수를 그냥 실행하면 this는 전역객체를 참조하지만 call 메서드를 이용하면 임의의 객체를 this로 지정할 수 있다.

### 예시 1번

```Js
let func = function (a,b,c) {
    console.log(this,a,b,c);
}

func(1,2,3) // window 1 2 3
func.call({x : 1}, 4, 5, 6) // {x : 1 } 4 5 6
```

### 예시 2번

```Js
let obj = {
    a : 1,
    method : function (x,y) {
        console.log(this.a, x, y)
    }
};

obj.method(2,3); // 1 2 3
obj.method.call({a : 4}, 5, 6); // 4 5 6
```

### 3.2.2 apply 메서드

```Js
Function.prototype.apply(thisArg[, argsArray])
```

- apply 메서드는 call 메서드와 기능적으로 완전히 동일하다.

- call 메서드는 첫 번째 인자를 제외한 나머지 모든 인자들을 호출할 함수의 매개변수로 지정하는 반면, apply 메서드는 `두 번째 인자를 배열로 받아 그 배열의 요소들을 호출할 함수의 매개변수로 지정한다는 점`에서만 차이가 있다.

```Js
let func = function (a, b, c) {
    console.log(this, a, b, c);
};
func.apply({ x: 1 }, [4, 5, 6]); // {x: 1} 4 5 6

let obj = {
    a : 1,
    method : function (x,y) {
        console.log(this.a, x, y)
    }
};

obj.method.apply({a : 4}, [5, 6]); // 4 5 6
```

### 3.2.3 call / apply 메서드의 활용

### 🔍 유사배열객체에 배열 메서드를 적용

- 배열의 구조와 유사한 객체의 경우(유사배열객체) call 또는 apply 메서드를 이용해 배열 메서드를 차용할 수 있다.

- slice 메서드는 원래 시작 인덱스값과 마지막 인덱스값을 받아 시작값부터 마지막값의 앞부분까지의 배열 요소를 추출하는 메서드인데, 매개변수를 아무것도 넘기지 않을 경우에는 그냥 원본 배열의 얕은 복사본을 반환한다.

- 아래 예제에서 call 메서드를 이용해 원본인 유사배열객체의 얕은 복사를 수행한 것인데, slice 메서드가 배열 메서드이기 때문에 복사본은 배열로 반환하였다.

```Js
let obj = {
    0 : 'a',
    1 : 'b',
    2 : 'c',
    length : 3
};

Array.prototype.push.call(obj, 'd');
console.log(obj) // { 0: 'a', 1: 'b', 2: 'c', 3: 'd', length: 4 }

let arr = Array.prototype.slice.call(obj);
console.log(arr) // [ 'a', 'b', 'c', 'd' ]
```

- 함수 내부에서 접근할 수 있는 arguments 객체도 유사배열객체이므로 위의 방법으로 배열로 전환해서 활용할 수 있다.

- querySelectorAll, getElementsByClassName 등의 Node 선택자로 선택한 결과인 NodeList도 마찬가지이다.

- 그 밖에도 유사배열객체에는 call/apply 메서드를 이용해 모든 배열 메서드를 적용할 수 있다.

- 배열처럼 인덱스와 length 프로퍼티를 지니는 문자열에 대해서도 마찬가지이다.

- 단, 문자열의 경우 length 프로퍼티가 읽기 전용이기 때문에 원본 문자열에 변경을 가하는 메서드(push, pop, shift, unshift, splice 등)는 에러를 던지며, concat처럼 대상이 반드시 배열이어야 하는 경우에는 에러는 나지 않지만 제대로 된 결과를 얻을 수 없다.

- ES6에서는 유사배열객체 또는 순회 가능한 `모든 종류의 데이터 타입을 배열로 전환하는 Array.from 메서드를 새로 도입`하였다.

```Js
let obj = {
    0 : 'a',
    1 : 'b',
    2 : 'c',
    length : 3
};

let arr = Array.from(obj);
console.log(arr) // ['a', 'b', 'c']
```

### 🔍 생성자 내부에서 다른 생성자를 호출

- 생성자 내부에 다른 생성자와 공통된 내용이 있을 경우 call 또는 apply를 이용해 다른 생성자를 호출하면 간단하게 반복을 줄일 수 있다.

```Js
function Person(name,gender) {
    this.name = name;
    this.gender = gender;
}

function Student(name, gender, age){
    Person.call(this, name, gender);
    this.age = age;
}
function Employee(name, gender, company){
    Person.apply(this, [name,gender]);
    this.company = company
}
let chris = new Student("Chris","male",20)
let tom = new Employee("Tom","male","google")
```

### 🔍 여러 인수를 묶어 하나의 배열로 전달하고 싶을 때 - apply 활용

- 여러 개의 인수를 받는 메서드에게 하나의 배열로 인수들을 전달하고 싶을 때 apply 메서드를 사용하면 좋다.

```Js
let numbers = [10, 20, 3, 16, 45];
let max = Math.max.apply(null, numbers);
let min = Math.min.apply(null, numbers);
console.log(max, min);      // 45 3
```

- ES6에서는 펼치기 연산자spread operator를 이용하면 apply를 적용하는 것보다 더욱 간편하게 작성할 수 있다.

```Js
let numbers = [10, 20, 3, 16, 45];
let max = Math.max(...numbers);
let min = Math.min(...numbers);
console.log(max, min);      // 45 3
```

### 3.2.4 bind 메서드

```Js
Function.prototype.bind(thisArg[, arg1[, arg2[, ...]]])
```

- bind 메서드는 ES5에서 추가된 기능으로, call과 비슷하지만 즉시 호출하지는 않고 넘겨받은 this 및 인수들을 바탕으로 새로운 함수를 반환하기만 하는 메서드이다.

- 다시 새로운 함수를 호출할 때 인수를 넘기면 그 인수들은 기존 bind 메서드를 호출할 때 전달했던 인수들의 뒤에 이어서 등록된다.

- 즉 bind 메서드는 함수에 this를 미리 적용하는 것과 부분 적용 함수를 구현하는 두 가지 목적을 모두 지닌다

```Js
let func = function (a, b, c, d) {
    console.log(this, a, b, c, d);
};
func(1, 2, 3, 4); // window 1 2 3 4

let bindFunc1 = func.bind({ x : 1});
bindFunc1(5,6,7,8); // {x : 1 } 5 6 7 8

let bindFunc2 = func.bind({ x : 1},4, 5);
bindFunc2(6,7); // { x : 1 } 4 5 6 7
bindFunc2(8,9); // { x : 1 } 4 5 8 9
```

### 🔍 name 프로퍼티

- bind 메서드를 적용해서 새로 만든 함수는 name 프로퍼티에 동사 bind의 수동태인 'bound'라는 접두어가 붙는다.

```Js
let func = function (a, b, c, d) {
    console.log(this, a, b, c, d);
};
let bindFunc = func.bind({ x : 1}, 4, 5);
console.log(func.name); // func
console.log(bindFunc.name); // bound func
```

### 🔍 상위 컨텍스트의 this를 내부함수나 콜백 함수에 전달하기

- call, apply 또는 bind 메서드를 이용하면 this의 우회를 깔끔하게 처리 가능하다.

```Js
let obj = {
    outer: function () {
        console.log(this);
        // call을 이용한 방법
        let innerFunc = function () {
            console.log(this);
        };
        innerFunc.call(this);

        // bind를 이용한 방법
        let innerFunc = function () {
            console.log(this);
        }.bind(this);
        innerFunc();
    }
};
obj.outer();
```

- 콜백 함수를 인자로 받는 함수나 메서드 중에서 기본적으로 콜백 함수 내에서의 this에 관여하는 함수 또는 메서드에 대해서도 bind 메서드를 이용하면 this 값을 사용자의 입맛에 맞게 바꿀 수 있다.

### 3.2.5 화살표 함수의 예외사항

- 화살표 함수 내부에는 this가 아예 없으며, 접근하고자 하면 스코프체인상 가장 가까운 this에 접근하게 된다.

- 별도의 변수로 this를 우회하거나 call/apply/bind를 적용할 필요가 없어 더욱 간결하고 편리하다.

```js
let obj = {
  outer: function () {
    console.log(this);
    // 화살표 함수를 이용한 방법
    let innerFunc = () => {
      console.log(this);
    };
    innerFunc();
  },
};
obj.outer();
```

### 3.2.6 별도의 인자로 this를 받는 경우(콜백 함수 내에서의 this)

- 콜백 함수를 인자로 받는 메서드 중 일부는 추가로 this로 지정할 객체(thisArg)를 인자로 지정할 수 있는 경우가 있다.

- 이러한 메서드의 thisArg 값을 지정하면 콜백 함수 내부에서 this 값을 원하는 대로 변경할 수 있다.

- 이러한 형태는 배열 메서드에 많이 있고 Set, Map 등의 메서드에도 일부 존재한다.

## 3.3 정리

### 명시적 this 바인딩이 없을 시 규칙

- 전역공간에서의 this는 전역객체(브라우저에서는 window, Node.js에서는 global)를 참조합니다.

- 어떤 함수를 메서드로서 호출한 경우 this는 메서드 호출 주체(메서드명 앞의 객체)를 참조합니다.

- 어떤 함수를 함수로서 호출한 경우 this는 전역객체를 참조합니다. 메서드의 내부함수에서도 같습니다.

- 콜백 함수 내부에서의 this는 해당 콜백 함수의 제어권을 넘겨받은 함수가 정의한 바에 따르며, 정의하지 않은 경우에는 전역객체를 참조합니다.

- 생성자 함수에서의 this는 생성될 인스턴스를 참조합니다.

### 명시적 this 바인딩

- call, apply 메서드는 this를 명시적으로 지정하면서 함수 또는 메서드를 호출합니다.

- bind 메서드는 this 및 함수에 넘길 인수를 일부 지정해서 새로운 함수를 만듭니다.

- 요소를 순회하면서 콜백 함수를 반복 호출하는 내용의 일부 메서드는 별도의 인자로 this를 받기도 합니다.
