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

## 11.

### 절차
1.

### Before
```js
```

### After
```js
```

## 11.

### 절차
1.

### Before
```js
```

### After
```js
```

## 11.

### 절차
1.

### Before
```js
```

### After
```js
```

## 11.

### 절차
1.

### Before
```js
```

### After
```js
```

## 11.

### 절차
1.

### Before
```js
```

### After
```js
```

## 11.

### 절차
1.

### Before
```js
```

### After
```js
```

## 11.

### 절차
1.

### Before
```js
```

### After
```js
```

## 11.

### 절차
1.

### Before
```js
```

### After
```js
```
