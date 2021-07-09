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

### 절차

### Before
```js

```

### After
```js

```

## 6.7

### 절차

### Before
```js

```

### After
```js

```

## 6.8

### 절차

### Before
```js

```

### After
```js

```

## 6.9

### 절차

### Before
```js

```

### After
```js

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
