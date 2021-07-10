# Chapter 06 memo
기본적인 리팩터링

## 6.1 함수 추출하기

코드를 보고 무슨 일을 하는지 파악하는데 한참이 걸린다면? 그 부분을 함수로 추출한 뒤 '무슨 일'에 걸맞는 이름을 짓자. <함수 추출하기> 원칙을 적용하면 함수가 많아지고, 함수 호출이 많아진다. 함수 호출이 많아지면 성능이 느려질까? 그렇지 않다. 옛날이면 모를까 요즘은 함수 호출로 인해 성능이 느려지는 문제는 거의 발생하지 않는다.  

### 절차
1. 함수를 새로 만들고 목적을 잘 드러내는 이름을 붙인다.
2. 추출할 코드를 원본 함수에서 복사하여 새 함수에 붙여넣는다.
3. 추출한 코드 중 원본 함수의 지역 변수를 참조하거나 추출한 함수의 유효범위를 벗어나는 변수는 없는지 검사한다. 있다면 매개변수로 전달한다.
4. 변수를 다 처리했다면 컴파일한다.
5. 원본 함수에서 추출한 코드 부분을 새로 만든 함수를 호출하는 문장으로 바꾼다.
6. 테스트한다.
7. 방금 추출한 것과 똑같거나 비슷한 코드가 없는지 살핀다. 있다면 방금 추출한 함수를 사용할지 검토한다.

### Before
```js
function prinOwing(invoice) {
    printBanner();
    let outstanding = calculateOutstanding();

    console.log('고객명: ${invoice.customer}');
    console.log('채무액: ${outstanding}');
}
```

### After
```js
function printOwing(invoice) {
    printBanner();
    let outstanding = calculateOutstanding();
    printDetails(outstanding);
    
    function printDetails(outstanding) {
        console.log('고객명: ${invoice.customer}');
        console.log('채무액: ${outstanding}');
    }
}
```

## 6.2 함수 인라인하기 

리팩터링 과정에서 잘못 추출된 함수, 간접 호출을 너무 과하게 쓰는 코드 등에 적용해보자. 유용한 것만 남기고 나머지는 제거한다. 

### 절차
1. 다형 메서드인지 확인한다. 서브클래스에서 오버라이드하는 메서드는 인라인하면 안된다!
2. 인라인할 함수를 호출하는 곳을 모두 찾는다.
3. 각 호출문을 함수 본문으로 교체한다.
4. 하나씩 교체할 때마다 테스트한다.
5. 원래 있던 함수를 제거한다.

### Before
```js
function getRating(driver) {
    return moreThanFiveLateDeliveries(driver) ? 2 : 1;
}

function moreThanFiveLateDeliveries(driver) {
    return driver.numberOfLateDeliveries > 5;
}
```

### After
```js
function getRating(driver) {
    return (driver.numberOfLateDeliveries > 5) ? 2 : 1;
}
```

## 6.3 변수 추출하기

표현식이 너무 복잡해서 이해하기 어려울 때가 있다. 이럴 때 로컬 변수를 활용하면 표현식을 쪼개 코드의 목적을 명확하게 드러낼 수 있다. 이 과정에서 추가한 변수는 디버깅에도 도움이 된다. 디버거에 중단점을 지정할 수 있기 때문이다. 

### 절차
1. 추출하려는 표현식에 부작용은 없는지 확인한다.
2. 불변 변수(final, const)를 하나 선언하고 이름을 붙일 표현식의 복제본을 대입한다.
3. 원본 표현식을 새로 만든 변수로 교체한다.
4. 테스트한다.
5. 표현식을 여러 곳에서 사용한다면 각각을 새로 만든 변수로 교체한다. 하나 교체할 때마다 테스트한다.

### Before
```js
return order.quantity * order.itemPrice -
    Math.max(0, order.quantity - 500) * order.itemPrice * 0.05 +
    Math.min(order.quantity * order.itemPrice * 0.1, 100);
```

### After
```js
const basePrice = order.quantity * order.itemPrice;
const quantityDiscount = Math.max(0, order.quantity - 500) * order.itemPrice * 0.05;
const shipping = Math.min(basePrice * 0.1, 100);
return basePrice - quantityDiscount + shipping;
```

## 6.4 변수 인라인하기

추출한 변수를 다시 인라인 할 수도 있다.  

### 절차
1. 대입문의 표현식에 부작용이 생기지는 않는지 확인한다.
2. 변수가 불변으로 선언되지 않았다면 불변으로 만든 후 테스트한다.
3. 이 변수를 가장 처음 사용하는 코드를 찾아서 바꾼다.
4. 테스트한다.
5. 변수를 사용하는 모든 부분을 교체할 때까지 반복한다.
6. 변수 선언문과 대입문을 지운다.
7. 테스트한다.

### Before
```js
let basePrice = anOrder.basePrice;
return (basePrice > 1000);
```

### After
```js
return anOrder.basePrice > 1000;
```

## 6.5 함수 선언 바꾸기

함수는 프로그램을 작은 부분으로 나누는 주된 수단이다. 함수를 잘 정의하면 시스템에 새로운 부분을 추가하기가 쉬워지는 반면, 잘못 정의하면 지속적인 방해 요인이 된다. 함수의 이름과 매개변수를 바꾸면서 함수의 선언부를 적절하게 개선해보자. 

### 절차
함수 선언 바꾸기에는 정답이 없다. 먼저 간단한 절차를 거친다음 더 세분화된 마이그레이션 절차를 거치자. 

#### 간단한 절차
1. 매개변수를 제거하려거든 먼저 함수 본문에서 제거 대상 매개변수를 참조하는 곳은 없는지 확인한다.
2. 메서드 선언을 원하는 형태로 바꾼다.
3. 기존 메서드 선언을 참조하는 부분을 모두 찾아서 바뀐 형태로 수정한다.
4. 테스트한다.

#### 마이그레이션 절차
1. 이어지는 추출 단계를 수월하게 만들어야 한다면 함수의 본문을 적절히 리팩터링한다.
2. 함수 본문을 새로운 함수로 추출한다.
3. 추출한 함수에 매개변수를 추가해야 한다면 위의 간단한 절차를 따라 추가한다.
4. 테스트한다.
5. 기존 함수를 인라인한다.
6. 테스트한다.

이 리팩터링을 할 때는 먼저 변경 사항을 살펴보고 함수 선언과 호출문을 단번에 고칠 수 있는지 가늠해본다. 변경할게 여러 개면 나눠서 각각을 독립적으로 처리하자. 

### Before
```js
function circum(radius) {...}
```

### After
```js
function circumference(radius) {...}
```

## 6.6 변수 캡슐화하기

데이터는 함수보다 다루기가 까다롭다. 데이터는 참조하는 모든 부분을 한 번에 바꿔야 코드가 제대로 작동하기 때문이다. 따라서 변수의 유효범위가 넓어질수록 다루기 어려워진다. 전역 데이터가 골칫거리인 이유도 바로 여기에 있다.  

그래서 접근 범위가 넓은 데이터를 옮길 때는 먼저 그 데이터로의 접근을 독점하는 함수를 만드는 식으로 캡슐화하는 것이 가장 좋은 방법일 때가 많다. 데이터 캡슐화는 다른 경우에도 도움을 준다. 데이터를 변경하고 사용하는 코드를 감시할 수 있는 확실한 통로가 되어주기 때문에 검증이나 추가 로직을 쉽게 끼워 넣을 수 있다. 데이터의 유효범위가 넓을수록 캡슐화해야 한다. 그래야 자주 사용하는 데이터에 대한 결합도가 높아지는 일을 막을 수 있다. 객체 지향에서 객체의 데이터를 항상 private으로 유지해야 한다고 강조하는 이유도 여기에 있다.   

### 절차

1. 변수로의 접근과 갱신을 전담하는 캡슐화 함수들을 만든다.
2. 정적 검사를 수행한다.
3. 변수를 직접 참조하던 부분을 모두 적절한 캡슐화 함수 호출로 바꾼다. 하나씩 바꿀 때마다 테스트ㅏㄴ다.
4. 변수의 접근 범위를 제한한다.
5. 테스트한다.
6. 변수 값이 레코드라면 레코드 캡슐화하기를 적용할지 고려해본다.

### Before
```js
let defaultOwner = {firstName: "마틴", lastName: "파울러"};
```

### After
```js
let defaultOwnerData = {firstName: "마틴", lastName: "파울러"};
export function defaultOwner() { return defaultOwnerData; }
export function setDefaultOwner(ars) { defaultOwnerData = arg; }
```

## 6.7 변수 이름 바꾸기

명확한 프로그래밍의 핵심은 이름짓기다. 

### 절차

1. 폭넓게 쓰이는 변수라면 변수 캡슐화하기를 고려한다.
2. 이름을 바꿀 변수를 참조하는 곳을 모두 찾아서 하나씩 변경한다.
3. 테스트한다.

### Before
```js
let a = height * width;
```

### After
```js
let area = height * width;
```

## 6.8 매개변수 객체 만들기

데이터 항목 여러 개가 이 함수에서 저 함수로 함께 몰려다니는 경우를 자주 본다. 이런 데이터 무리를 발견하면 하나의 데이터 구조로 모아주자. 이렇게 하면 데이터 사이의 관계가 명확해지며, 함수가 이 데이터 구조를 받게 하면 매개변수의 수가 줄어든다. 같은 데이터 구조를 사용하는 모든 함수가 원소를 참조할 때 항상 같은 이름을 사용하기 때문에 코드 일관성도 높여준다.  

하지만 이 리팩터링의 진정한 힘은 코드를 더 근본적으로 바꿔준다는 데 있다. 새로 만든 데이터 구조가 문제 영역을 훨씬 간결하게 표현하는 새로운 추상 개념이 되면서, 코드의 개념적인 그림을 다시 그릴 수도 있다.

### 절차

1. 적당한 데이터 구조가 없다면 새로 만든다. 클래스로 만들고 값 객체(VO)로 만드는 것이 좋다.
2. 테스트한다.
3. 함수 선언 바꾸기로 새 데이터 구조를 매개변수로 추가한다.
4. 테스트한다.
5. 함수 호출 시 새로운 데이터 구조 인스턴스를 넘기도록 수정한다. 하나씩 수정할 때마다 테스트한다.
6. 기존 매개변수를 사용하던 코드를 새 데이터 구조의 원소를 사용하도록 바꾼다.
7. 기존 매개변수를 제거하고 테스트한다.

### Before
```js
function amountInvoiced(startDate, endDate) {...}
function amountReceived(startDate, endDate) {...}
function amountOverdue(startDate, endDate) {...}
```

### After
```js
function amountInvoiced(aDateRange) {...}
function amountReceived(aDateRange) {...}
function amountOverdue(aDateRange) {...}
```

## 6.9 여러 함수를 클래스로 묶기

클래스는 대다수의 최신 프로그래밍 언어가 제공하는 기본적인 빌딩 블록이다. 클래스는 데이터와 함수를 하나의 공유 환경으로 묶은 후, 다른 프로그램 요소와 어우러질 수 있도록 그중 일부를 외부에 제공한다.  

이 리팩터링은 이미 만들어진 함수들을 재구성할 때는 물론, 새로 만든 클래스와 관련하여 놓친 연산을 찾아서 새 클래스의 메서드로 뽑아내는 데도 좋다. 

### 절차

1. 함수들이 공유하는 공통 데이터 레코드를 캡슐화한다.
2. 공통 레코드를 사용하는 함수 각각을 새 클래스로 옮긴다.
3. 데이터를 조작하는 로직들은 함수로 추출해서 새 클래스로 옮긴다.

### Before
```js
function base(aReading) {...}
function taxableCharge(aReading) {...}
function calculateBaseCharge(aReading) {...}
```

### After
```js
class Reading {
    base() {...}
    taxableCharge() {...}
    calculateBaseCharge() {...}
}
```

## 6.10

### 절차

### Before
```js

```

### After
```js

```

## 6.11
### 절차

### Before
```js

```

### After
```js

```
