# Chapter 08 memo
기능 이동

## 8.1 함수 옮기기

### 절차
1. 선택한 함수가 현재 컨텍스트에서 사용중인 모든 프로그램 요소를 살펴본다. 이 요소들 중에도 함께 옮겨야 할 게 있는지 고민해본다.
2. 선택한 함수가 다형 메서드인지 확인한다.
3. 선택한 함수를 타깃 컨텍스트로 복사한다. 복사해서 새로 만든 타깃 함수를 다듬는다.
4. 정적 분석을 수행한다.
5. 소스 컨텍스트에서 타깃 함수를 참조할 방법을 찾아 반영한다.
6. 소스 함수를 타깃 함수의 위임 함수가 되도록 수정한다.
7. 테스트한다.
8. 소스 함수를 인라인할지 고민해본다.

### Before
```js
class Account {
    get overdraftCharge() {
    ...
    }
```

### After
```js
class AccountType {
    get overdraftCharge() {
    ...
    }
```

## 8.2 필드 옮기기

### 절차
1. 소스 필드가 캡슐화되어 있지 않다면 캡슐화한다.
2. 테스트한다.
3. 타깃 객체에 필드와 접근자 메서드를 생성한다.
4. 정적 검사를 수행한다.
5. 소스 객체에서 타깃 객체를 참조할 수 있는지 확인한다.
6. 접근자들이 타깃 필드를 사용하도록 수정한다.
7. 테스트한다.
8. 소스 필드를 제거한다.
9. 테스트한다.

### Before
```js
class Customer {
    get plan() {return this._plan;}
    get discountRate() {return this._discountRate;}
```

### After
```js
class Customer {
    get plan() {return this._plan;}
    get discountRate() {return this.plan.discountRate;}
```

## 8.3 문장을 함수로 옮기기

### 절차
1. 반복 코드가 함수 호출 부분과 멀리 떨어져 있다면 문장 슬라이스하기를 적용해 근처로 옮긴다.
2. 타깃 함수를 호출하는 곳이 한 곳뿐이면, 단순히 소스 위치에서 해당 코드를 잘라내어 피호출 함수로 복사하고 테스트한다. 이 경우라면 나머지 단계는 무시한다.
3. 호출자가 둘 이상이면 호출자 중 하나에서 '타깃 함수 호출 부분과 그 함수로 옮기려는 문장들을 함께' 다른 함수로 추출한다. 추출한 함수에 기억하기 쉬운 임시 이름을 지어준다.
4. 다른 호출자 모두가 방금 추출한 함수를 사용하도록 수정한다. 하나씩 수정할 때마다 테스트한다.
5. 모든 호출자가 새로운 함수를 사용하게 되면 원래 함수를 새로운 함수 안으로 인라인한 후 원래 함수를 제거한다.
6. 새로운 함수의 이름을 원래 함수의 이름으로 바꿔준다.

### Before
```js
result.push('<p>제목: ${person.photo.title}</p>');
result.concat(photoData(person.photo));

function photoData(aPhoto) {
    return [
        '<p>위치: ${aPhoto.location}</p>',
        '<p>날짜: ${aPhoto.date.toDateString()}</p>',
    ];
}
```

### After
```js
result.concat(photoData(person.photo));

function photoData(aPhoto) {
    return [
        '<p>제목: ${aPhoto.title}</p>',
        '<p>위치: ${aPhoto.location}</p>',
        '<p>날짜: ${aPhoto.date.toDateString()}</p>',
    ];
}
```

## 8.4 문장을 호출한 곳으로 옮기기

### 절차
1. 호출자가 한두 개뿐이고 피호출 함수도 간단한 상황이면, 피호출 함수의 처음 혹은 마지막 줄들을 잘라내어 호출자로 복사해 넣는다. 테스트한다.
2. 더 복잡한 상황에서는, 이동하지 않길 원하는 모든 문장을 함수로 추출한 다음 검색하기 쉬운 임시 이름을 지어준다.
3. 원래 함수를 인라인한다.
4. 추출된 함수의 이름을 원래 함수의 이름으로 변경한다.

### Before
```js
emitPhotoData(outStream, person.photo);

function emitPhotoData(outStream, photo) {
    outStream.write('<p>제목: ${photo.title}</p>\n');
    outStream.write('<p>위치: ${photo.location}</p>\n');
}
```

### After
```js
emitPhotoData(outStream, person.photo);
outStream.write('<p>위치: ${photo.location}</p>\n');

function emitPhotoData(outStream, photo) {
    outStream.write('<p>제목: ${photo.title}</p>\n');
}
```

## 8.5 인라인 코드를 함수 호출로 바꾸기

### 절차
1. 인라인 코드를 함수 호출로 대체한다.
2. 테스트한다.

### Before
```js
let appliesToMass = false;
for(const s of states) {
    if (s === "MA") appliesToMass = true;
}
```

### After
```js
appliesToMass = states.includes("MA");
```

## 8.6 문장 슬라이드하기

### 절차
1. 코드 조각을 이동할 목표 위치를 찾는다. 코드 조각의 원래 위치와 타겟 위치 사이의 코드를 훑어보면서, 조각을 모으고 나면 동작인 달라지는 코드가 있는지 살핀다. 경우에 따라 이 리팩터링은 포기한다.
2. 코드 조각을 원래 위치에서 잘라내어 타겟 위치에 붙여 넣는다.
3. 테스트한다.

### Before
```js
const pricingPlan = retrievePricingPlan();
const order = retreiveOrder();
let charge;
const chargePerUnit = pricingPlan.unit;
```

### After
```js
const pricingPlan = retrievePricingPlan();
const chargePerUnit = pricingPlan.unit;
const order = retreiveOrder();
let charge;
```

## 8.7 반복문 쪼개기

### 절차
1. 반복문을 복제해 두 개로 만든다.
2. 반복문이 중복되어 생기는 부수효과를 파악해서 제거한다.
3. 테스트한다.
4. 완료됐으면, 각 반복문을 함수로 추출할지 고민해본다.

### Before
```js
let averageAge = 0;
let totalSalary = 0;
for (const p of people) {
    averageAge += p.age;
    totalSalary += p.salary;
}
averageAge = averageAge / people.length;
```

### After
```js
let totalSalary = 0;
for (const p of people) {
    totalSalary += p.salary;
}

let averageAge = 0;
for (const p of people) {
    averageAge += p.age;
}
averageAge = averageAge / people.length;
```

## 8.8 반복문을 파이프라인으로 바꾸기

### 절차
1. 반복문에서 사용하는 컬렉션을 가리키는 변수를 하나 만든다.
2. 반복문의 첫 줄부터 시작해서, 각각의 단위 행위를 적절한 컬렉션 파이프라인 연산으로 대체한다. 
3. 반복문의 모든 동작을 대체했다면 반복문 자체를 지운다.

### Before
```js
const names = [];
for (const i of input) {
    if (i.job === "programmer")
        names.push(i.name);
}
```

### After
```js
const names = input
    .filter(i => i.job === "programmer")
    .map(i => i.name)
;
```

## 8.9 죽은 코드 제거하기

### 절차
1. 죽은 코드를 외부에서 참조할 수 있는 경우(함수) 혹시라도 호출하는 곳이 있는지 확인한다.
2. 없다면 죽은 코드를 제거한다.
3. 테스트한다.

### Before
```js
if(false) {
    doSomethingThatUsedToMatter();
}
```

### After
```js

```
