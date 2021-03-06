# Chapter 09
데이터 조직화

## 9.1 변수 쪼개기

변수는 다양한 용도로 쓰인다. 만약 변수에 대입이 두 번 이상 이뤄진다면 여러 가지 역할을 수행한다는 신호이므로 이를 쪼개야 한다. 역할 하나당 변수 하나를 지켜야 한다.

### 절차
1. 변수를 선언한 곳과 값을 처음 대입하는 곳에서 변수 이름을 바꾼다. 만약 이후의 대입이 항상 i = i + <무언가> 형태라면 수집 변수이므로 쪼개면 안 된다. 수집 변수는 총합 계산, 문자열 연결, 스트림, 컬렉션 추가 등의 용도로 흔히 쓰인다.
2. 가능하면 이때 불변으로 선언한다.
3. 이 변수에 두 번째로 값을 대입하는 곳 앞까지의 모든 참조를 새로운 변수 이름으로 바꾼다.
4. 두 번째 대입 시 변수를 원래 이름으로 다시 선언한다.
5. 테스트한다.
6. 반복한다. 이 과정을 마지막 대입까지 반복한다.

### Before
```js
let temp = 2 * (height + width);
console.log(temp);
temp = height * width;
console.log(temp);
```

### After
```js
const perimeter = 2 * (height + width);
console.log(perimeter);
const area = height * width;
console.log(area);
```

## 9.2 필드 이름 바꾸기

이름은 중요하다. 프로그램 곳곳에서 쓰이는 레코드 구조체의 필드 이름은 특히 더 중요하다. 따라서 반드시 깔끔하게 관리해야 한다.

### 절차
1. 레코드의 유효 범위가 제한적이라면 필드에 접근하는 모든 코드를 수정한 후 테스트한다.
2. 레코드가 캡슐화되지 않았다면 우선 레코드를 캡슐화한다.
3. 캡슐화된 객체 안의 private 필드명을 변경하고, 그에 맞게 내부 메서드들을 수정한다.
4. 테스트한다.
5. 생성자의 매개변수 중 필드와 이름이 겹치는 게 있다면 함수 선언 바꾸기로 변경한다.
6. 접근자들의 이름도 바꿔준다.

### Before
```js
class Organization {
    constructor(data) {
        this._name = data.name;
        this._country = data.country;
    }
    get name() {return this._name;}
    set name(aString) {this._name = aString;}
    get country() {return this._country;}
    set country(aCountryCode) {this._country = aCountryCode;}
} 
```

### After
```js
class Organization {
    constructor(data) {
        this._title = data.title;
        this._country = data.country;
    }
    get title() {return this._title;}
    set title(aString) {this._title = aString;}
    get country() {return this._country;}
    set country(aCountryCode) {this._country = aCountryCode;}
}
```

## 9.3 파생 변수를 질의 함수로 바꾸기

가변 데이터는 유효 범위를 가능한 좁히는 것이 좋다. 서로 다른 코드가 섞여서 원인을 찾기 어려운 문제를 야기할 수 있기 때문이다. 

### 절차
1. 변수 값이 갱신되는 지점을 모두 찾는다. 필요하면 변수 쪼개기를 활용해 각 갱신 지점에서 변수를 분리한다.
2. 해당 변수의 값을 계산해주는 함수를 만든다.
3. 해당 변수가 새용되는 모든 곳에 어서션을 추가하여 함수의 계산 결과가 변수의 값과 같은지 확인한다.
4. 테스트한다.
5. 변수를 읽는 코드를 모두 함수 호출로 대체한다.
6. 테스트한다.
7. 변수를 선언하고 갱신하는 코드를 죽은 코드 제거하기로 없앤다.

### Before
```js
get production() { return this._production; }
applyAdjustment(anAudjustment) {
  this._adjustments.push(anAdujustment);
  this._production += anAdjustment.amount;
}
```

### After
```js
get production() {
  return this._adjustments
      .reduce((sum, a) => sum + a.amount, 0);
}
applyAdjustment(anAudjustment) {
  this._adjustments.push(anAdujustment);
}
```

## 9.4 참조를 값으로 바꾸기

객체를 다른 객체에 중첩하면 '내부 객체를 참조' 혹은 '값으로 취급'할 수 있다. 참조로 다루는 경우에는 내부 객체는 그대로 둔 채 그 객체의 속성만 갱신하며, 값으로 다루는 경우에는 새로운 속성을 담은 객체로 기존 내부 객체를 통째로 대체한다. 값으로 취급하면 불변 데이터로 다루기 때문에 분산 시스템과 동시성 시스템 등에서 특히 유용하다.  

### 절차
1. 후보 클래스가 불변인지, 혹은 불변이 될 수 있는지 확인한다.
2. 각각의 세터를 하나씩 제거한다.
3. 이 값 객체의 필드들을 사용하는 동치성 비교 메서드를 만든다. 동치성 비교 메서드를 오버라이드할 때는 보통 해시코드 생성 메서드도 함께 오버라이드해야 한다.

### Before
```js
class Product {
    applyDiscount(arg) {this._price.amout -= arg;}
```

### After
```js
class Product {
    applyDiscount(arg) {
        this._price = new Money(this._price.amount - arg, this._price.currency);
    }
```

## 9.5 값을 참조로 바꾸기

9.4의 반대 리팩터링. 논리적으로 같은 데이터를 복제해서 사용할 때 가장 크게 문제되는 상황은 그 데이터를 업데이트해야 할 때다. 모든 복제본을 찾아서 빠짐없이 갱신해야 하며, 하나라도 놓치면 데이터 일관성이 깨져버린다. 이런 상황이라면 값 데이터를 모두 참조로 바꿔주는 게 좋다. 값을 참조로 바꾸면 엔티티 하나당 객체도 단 하나만 존재하게 된다. 이 때 이런 객체들을 모아놓고 클라이언트의 접근을 관리해주는 저장소 객체(repository)를 사용하게 된다.   

### 절차
1. 같은 부류에 속하는 객체들을 보관할 저장소를 만든다.
2. 생성자에서 이 부류의 객체들 중 특정 객체를 정확히 찾아내는 방법이 있는지 확인한다.
3. 호스트 객체의 생성자들을 수정하여 필요한 객체를 이 저장소에서 찾도록 한다. 하나 수정할 때마다 테스트한다.

### Before
```js
let customer = new Customer(customerData);
```

### After
```js
let customer = customerRepository.get(customerData.id);
```

## 9.6 매직 리터럴 바꾸기

매직 리터럴(magic literal)이란 소스 코드의 여러 곳에서 등장하는 일반적인 리터럴 값을 말한다. 예를 들어 움직임을 계산하는 코드라면 9.80665라는 표준중력 숫자를 많이 찾아 볼 수 있다. 하지만 코드를 읽는 사람이 이 값의 의미를 모른다면 숫자 자체로는 의미를 명확히 알려주지 못하므로 매직 리터럴이라 할 수 있다. 상수를 정의하고 숫자 대신 그 상수를 사용하도록 바꾸자.

### 절차
1. 상수를 선언하고 매직 리터럴을 대입한다.
2. 해당 리터럴이 사용되는 곳을 모두 찾는다.
3. 찾은 곳 각각에서 리터럴이 새 상수와 똑같은 의미로 쓰였는지 확인하여, 같은 의미라면 상수로 대체한 후 테스트한다.

### Before
```js
function potentialEnergy(mass, height) {
    return mass * 9.81 * height;
}
```

### After
```js
const STANDARD_GRAVITY = 9.81;
function potentialEnergy(mass, height) {
    return mass * STANDARD_GRAVITY * height;
}
```
