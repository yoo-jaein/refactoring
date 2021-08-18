# Chapter 11
API 리팩터링

## 11.1 질의 함수와 변경 함수 분리하기

'질의 함수(읽기 함수)는 모두 부수효과가 없어야 한다'는 규칙을 따르는 것이 좋다. 이를 위해 함수에서 상태를 변경하는 부분과 질의하는 부분을 분리하자. 

### 절차
1. 대상 함수를 복제하고 질의 목적에 충실한 이름을 짓는다. 
2. 새 질의 함수에서 부수효과를 모두 제거한다.
3. 정적 검사를 수행한다.
4. 원래 함수(변경 함수)를 호출하는 곳을 모두 찾아낸다. 호출하는 곳에서 반환 값을 사용한다면 질의 함수를 호출하도록 바꾸고, 원래 함수를 호출하는 코드를 바로 아래 줄에 새로 추가한다. 수정할 때마다 테스트한다.
5. 원래 함수에서 질의 관련 코드를 제거한다.
6. 테스트한다.

### Before
```js
function getTotalOutstandingAndSendBill() {
    const result = customer.invoices.reduce((total, each) => each.amount + total, 0);
    sendBill();
    return result;
}
```

### After
```js
function totalOutstanding() {
    return customer.invoices.reduce((total, each) => each.amount + total, 0);
}

function sendBill() {
    emailGateway.send(formatBill(customer));
}
```

## 11.2 함수 매개변수화하기

두 함수의 로직이 아주 비슷하고 단지 리터럴 값만 다르다면, 그 다른 값만 매개변수로 받아 처리하는 함수 하나로 합치자. 코드 중복을 없앨 수 있고 함수의 유용성이 커진다.

### 절차
1. 비슷한 함수 중 하나를 선택한다.
2. 함수 선언 바꾸기로 리터럴들을 매개변수로 추가한다.
3. 이 함수를 호출하는 곳 모두에 적절한 리터럴 값을 추가한다.
4. 테스트한다.
5. 매개변수로 받은 값을 사용하도록 함수 본문을 수정한다. 하나 수정할 때마다 테스트한다.
6. 비슷한 다른 함수를 호출하는 코드를 찾아 매개변수화된 함수를 호출하도록 하나씩 수정한다. 하나 수정할 때마다 테스트한다.

### Before
```js
function tenPercentRaise(aPerson) {
    aPerson.salary = aPerson.salary.multiply(1.1);
}
function fivePercentRaise(aPerson) {
    aPerson.salary = aPerson.salary.multiply(1.05);
}
```

### After
```js
function raise(aPerson, factor) {
    aPerson.salary = aPerson.salary.multiply(1 + factor);
} 
```

## 11.3 플래그 인수 제거하기

플래그 인수(flag argument)란 호출되는 함수가 실행할 로직을 호출하는 쪽에서 선택하기 위해 전달하는 인수다. 플래그 인수를 제거하면 코드가 깔끔해지고 코드를 읽는 사람에게 뜻을 온전히 전달할 수 있게 한다.

### 절차
1. 매개변수로 주어질 수 있는 값 각각에 대응하는 명시적 함수들을 생성한다. 
2. 원래 함수를 호출하는 코드를 모두 찾아서 각 리터럴 값에 대응되는 명시적 함수를 호출하도록 수정한다.

### Before
```js
function setDimension(name, value) {
    if (name === "height") {
        this._height = value;
        return;
    }
    if (name === "width") {
        this._width = value;
        return;
    }
}
```

### After
```js
function setHeight(value) {this._height = value;}
function setWidth(value) {this._width = value;}
```

## 11.4 객체 통째로 넘기기

레코드를 통째로 넘기면 변화에 대응하기 쉽다. 예컨대 그 함수가 더 다양한 데이터를 사용하도록 바뀌어도 매개변수 목록은 수정할 필요가 없다. 하지만 함수가 레코드 자체에 의존하기를 원치 않을 때는 이 리팩터링을 수행하지 않는다. 어떤 객체로부터 값 몇 개를 얻은 후 그 값들만으로 무언가를 하는 로직이 있다면, 매개변수 객체 만들기 후에 적용한다.

### 절차
1. 매개변수들을 원하는 형태로 받는 빈 함수를 만든다.
2. 새 함수의 본문에서는 원래 함수를 호출하도록 하며, 새 매개변수와 원래 함수의 매개변수를 매핑한다.
3. 정적 검사를 수행한다.
4. 모든 호출자가 새 함수를 사용하게 수정한다. 하나씩 수정하며 테스트한다.
5. 호출자를 모두 수정했다면 원래 함수를 인라인한다.
6. 새 함수의 이름을 적절히 수정하고 모든 호출자에 반영한다.

### Before
```js
const low = aRoom.daysTempRange.low;
const high = aRoom.daysTempRange.high;
if (aPlan.withinRange(low, high))
```

### After
```js
if (aPlan.withinRange(aRoom.daysTempRange))
```

## 11.5 매개변수를 질의 함수로 바꾸기

다른 매개변수에서 얻을 수 있는 값을 별도의 매개변수로 전달하는 것은 아무 의미가 없다. 다른 리팩터링을 수행한 뒤 특정 매개변수가 더는 필요 없어졌을 때가 있는데, 이번 리팩터링을 적용하는 가장 흔한 사례다.

### 절차
1. 필요하다면 대상 매개변수의 값을 계산하는 코드를 별도 함수로 추출해놓는다.
2. 함수 본문에서 대상 매개변수로의 참조를 모두 찾아서 그 매개변수의 값을 만들어주는 표현식을 참조하도록 바꾼다. 하나 수정할 때마다 테스트한다.
3. 함수 선언 바꾸기로 대상 매개변수를 없앤다.

### Before
```js
availableVacation(anEmployee, anEmployee.grade);

function availableVacation(anEmployee, grade) {
    ...
```

### After
```js
availableVacation(anEmployee)

function availableVacation(anEmployee) {
    const grade = anEmployee.grade;
    ...
```

## 11.6 질의 함수를 매개변수로 바꾸기

### 절차
1. 변수 추출하기로 질의 코드를 함수 본문의 나머지 코드와 분리한다.
2. 함수 본문 중 해당 질의를 호출하지 않는 코드들을 별도 함수로 추출한다.
3. 방금 만든 변수를 인라인하여 제거한다.
4. 원래 함수도 인라인한다.
5. 새 함수의 이름을 원래 함수의 이름으로 고쳐준다.

### Before
```js
targetTemperature(aPlan)

function targetTemperature(aPlan) {
    currentTemperature = thermostat.currentTemperature;
```

### After
```js
targetTemperature(aPlan, thermostat.currentTemperature)

function targetTemperature(aPlan, currentTemperature) {
```

## 11.7 세터 제거하기

세터 메서드가 있다는 것은 필드가 수정될 수 있다는 뜻이다. 세터 제거하기가 필요한 상황은 주로 두 가지다. 첫째, 무조건 접근자 메서드를 통해서만 필드를 다루고 싶을 때. 두 번째, 클라이언트에서 생성 스크립트(일련의 세터 모음)를 사용해 객체를 생성할 때.

### 절차
1. 설정해야 할 값을 생성자에서 받지 않는다면 그 값을 받을 매개변수를 생성자에 추가한다(함수 선언 바꾸기). 그런 다음 생성자 안에서 적절한 세터를 호출한다.
2. 생성자 밖에서 세터를 호출하는 곳을 찾아 제거하고, 대신 새로운 생성자를 사용하도록 한다. 하나 수정할 때마다 테스트한다.
3. 세터 메서드를 인라인한다. 가능하다면 해당 필드를 불변으로 만든다.
4. 테스트한다.

### Before
```js
class Person {
    get name() {...}
    set name(aString) {...}
```

### After
```js
get name() {...}
```

## 11.8 생성자를 팩터리 함수로 바꾸기

### 절차
1. 팩터리 함수를 만든다. 팩터리 함수의 본문에서는 원래의 생성자를 호출한다.
2. 생성자를 호출하던 코드를 팩터리 함수 호출로 바꾼다.
3. 하나씩 수정할 때마다 테스트한다.
4. 생성자의 가시 범위가 최소가 되도록 제한한다.

### Before
```js
leadEngineer = new Employee(document.leadEngineer, 'E');
```

### After
```js
leadEngineer = createEngineer(document.leadEngineer);
```

## 11.9 함수를 명령으로 바꾸기

### 절차
1. 대상 함수의 기능을 옮길 빈 클래스를 만든다. 클래스 이름은 함수 이름에 기초해 짓는다.
2. 방금 생성한 빈 클래스로 함수를 옮긴다.
3. 함수의 인수들 각각은 명령의 필드로 만들어 생성자를 통해 설정할지 고민해본다.

### Before
```js
function score(candidate, medicalExam, scoringGuide) {
    let result = 0;
    let healthLevel = 0;
```

### After
```js
class Scorer {
    constructor(candidate, medicalExam, scoringGuide) {
        this._candidate = candidate;
        this._medicalExam = medicalExam;
        this._scoringGuide = scoringGuide;
    }
    
    execute() {
        this._result = 0;
        this._healthLevel = 0;
```

## 11.10 명령을 함수로 바꾸기

### 절차
1. 명령을 생성하는 코드와 명령의 실행 메서드를 호출하는 코드를 함께 함수로 추출한다.
2. 명령의 실행 함수가 호출하는 보조 메서드들 각각을 인라인한다.
3. 함수 선언 바꾸기를 적용하여 생성자의 매개변수 모두를 명령의 실행 메서드로 옮긴다.
4. 명령의 실행 메서드에서 참조하는 필드들 대신 대응하는 매개변수를 사용하게끔 바꾼다. 하나씩 수정할 때마다 테스트한다.
5. 생성자 호출과 명령의 실행 메서드 호출을 호출자(대체 함수) 안으로 인라인한다.
6. 테스트한다.
7. 죽은 코드 제거하기로 명령 클래스를 없앤다.

### Before
```js
class ChargeCalculator {
    constructor(customer, usage) {
        this._customer = customer;
        this._usage = usage;
    }
    execute() {
        return this._customer.rate * this._usage;
    }
}
```

### After
```js
function charge(customer, usage) {
    return customer.rate * usage;
}
```

## 11.11 수정된 값 반환하기

이 리팩터링은 값 하나를 계산하는 함수들에 가장 효과적이고, 값 여러 개를 갱신하는 함수에는 효과적이지 않다.

### 절차
1. 함수가 수정된 값을 반환하게 하여 호출자가 그 값을 자신의 변수에 저장하게 한다.
2. 테스트한다.
3. 피호출 함수 안에 반환할 값을 가리키는 새로운 변수를 선언한다.
4. 테스트한다.
5. 계산이 선언과 동시에 이뤄지도록 통합한다. 즉, 선언 시점에 계산 로직을 바로 실행해 대입한다.
6. 테스트한다.
7. 피호출 함수의 변수 이름을 새 역할에 어울리도록 바꿔준다.
8. 테스트한다.

### Before
```js
let totalAscent = 0;
calculateAscent();
```

### After
```js
const totalAscent = calculateAscent();
```

## 11.12 오류 코드를 예외로 바꾸기

### 절차
1. 콜스택 상위에 해당 예외를 처리할 예외 핸들러를 작성한다.
2. 테스트한다.
3. 해당 오류 코드를 대체할 예외와 그 밖은 예외를 구분할 식별 방법을 찾는다.
4. 정적 검사를 수행한다.
5. catch절을 수정하여 직접 처리할 수 있는 예외는 적절히 대처하고 그렇지 않은 예외는 다시 던진다.
6. 테스트한다.
7. 오류 코드를 반환하는 곳 모두에서 예외를 던지도록 수정한다. 하나씩 수정할 때마다 테스트한다.
8. 모두 수정했다면 그 오류 코드를 콜스택 위로 전달하는 코드를 모두 제거한다. 하나씩 수정할 때마다 테스트한다.

### Before
```js
if (data)
    return new ShippingRules(data);
else
    return -23;
```

### After
```js
if (data)
    return new ShippingRules(data);
else
    throw new OrderProcessingError(-23);
```

## 11.13 예외를 사전확인으로 바꾸기

### 절차
1. 예외를 유발하는 상황을 검사할 수 있는 조건문을 추가한다. catch 블록의 코드를 조건문의 조건절 중 하나로 옮기고, 남은 try 블록의 코드를 다른 조건절로 옮긴다. 
2. catch 블록에 어서션을 추가하고 테스트한다.
3. try문과 catch 블록을 제거한다.
4. 테스트한다.

### Before
```js
try {
    return values[periodNumber];
} catch (ArrayIndexOutOfBoundsException e) {
    return 0;
}
```

### After
```js
return (periodNumber >= values.length) ? 0 : values[periodNumber];
```
