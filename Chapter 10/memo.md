# Chapter 10
조건부 로직 간소화

## 10.1 조건문 분해하기

복잡한 조건부 로직은 프로그램을 복잡하게 만드는 가장 흔한 원흉에 속한다. 거대한 코드 블록이 주어지면 코드를 부위별로 분해한 다음 해체된 코드 덩어리들을 각 덩어리의 의도를 살린 이름의 함수 호출로 바꿔주자. 이 리팩터링은 함수 추출하기를 적용하는 한 사례로 볼 수도 있다.  

### 절차
1. 조건식과 그 조건식에 딸린 조건절 각각을 함수로 추출한다.

### Before
```js
if (!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd))
    charge = quantity * plan.summerRate;
else
    charge = quantity * plan.regularRate + plan.regularServiceCharge;
```

### After
```js
if (summer())
    charge = summerCharge();
else
    charge = regularCharge();
```

## 10.2 조건식 통합하기

비교 조건은 다르지만 그 결과로 수행하는 동작은 똑같은 코드가 있다. 같은 일을 할 거라면 조건 검사도 하나로 통합하는 게 낫다. 이럴 때 'and' 연산자와 'or' 연산자를 사용하면 여러 개의 비교 로직을 하나로 합칠 수 있다.  

### 절차
1. 해당 조건식들 모두에 부수효과가 없는지 확인한다.
2. 조건문 두 개를 선택하여 두 조건문의 조건식들을 논리 연산자로 결합한다.
3. 테스트한다.
4. 조건이 하나만 남을 때까지 2~3 과정을 반복한다.
5. 하나로 합쳐진 조건식을 함수로 추출할지 고려해본다.

### Before
```js
if (anEmployee.seniority < 2) return 0;
if (anEmployee.monthsDisabled > 12) return 0;
if (anEmployee.isPartTime) return 0;
```

### After
```js
if (isNotEligibleForDisability()) return 0;

function isNotEligibleForDisability() {
    return ((anEmployee.seniority < 2)
        || (anEmployee.monthsDisabled > 12)
        || (anEmployee.isPartTime));
}
```

## 10.3 중첩 조건문을 보호 구문으로 바꾸기

조건문에서 참인 경로와 거짓인 경로가 있다. 두 경로 모두 정상적인 코드 동작이라면 if-else절을 사용한다. 그러나 한쪽만 정상 동작이라면 비정상인 조건을 if에서 검사한 다음, 비정상인 경우 함수에서 빠져나오게 만들 수 있다. 이런 형태를 흔히 보호 구문(guard clause)이라고 한다.  

이 리팩터링은 코드의 의도를 부각하는 데 있다. 보호 구문은 "이건 이 함수의 핵심이 아니다. 이 일이 일어나면 무언가 조치를 취한 후 함수에서 빠져나온다"라고 이야기한다. 함수의 반환점이 하나여야 한다는 규칙은 그렇게 유용하지 않다. 반환점이 하나일 때 함수의 로직이 더 명백하다면 그렇게 하자. 그렇지 않다면 하지 말자.  

### 절차
1. 교체해야 할 조건 중 가장 바깥 것을 선택하여 보호 구문으로 바꾼다.
2. 테스트한다.
3. 1~2 과정을 필요한 만큼 반복한다.
4. 모든 보호 구문이 같은 결과를 반환한다면 보호 구문들의 조건식을 통합한다.

### Before
```js
function getPayAmount() {
    let result;
    if (isDead)
        result = deadAmount();
    else {
        if (isSeparated)
            result = separatedAmount();
        else {
            if (isRetired)
                result = retiredAmount();
            else
                result = normalPayAmount();
        }
    }
    return result;
}
```

### After
```js
function getPayAmount() {
    if (isDead) return deadAmount();
    if (isSeparated) return separatedAmount();
    if (isRetired) return retiredAmount();
    return normalPayAmount();
}
```

## 10.4 조건부 로직을 다형성으로 바꾸기

### 절차
1. 다형적 동작을 표현하는 클래스들이 아직 없다면 만들어준다. 이왕이면 적합한 인스턴스를 알아서 만들어 반환하는 팩터리 함수도 함께 만든다.
2. 호출하는 코드에서 팩터리 함수를 사용하게 한다.
3. 조건부 로직 함수를 슈퍼클래스로 옮긴다.
4. 서브클래스 중 하나를 선택한다. 서브클래스에서 슈퍼클래스의 조건부 로직 메서드를 오버라이드한다. 조건부 문장 중 선택된 서브클래스에 해당하는 조건절을 서브클래스 메서드로 복사한 다음 적절히 수정한다.
5. 같은 방식으로 각 조건절을 해당 서브클래스에서 메서드로 구현한다.
6. 슈퍼클래스 메서드에는 기본 동작 부분만 남긴다. 혹은 슈퍼클래스가 추상 클래스여야 한다면, 이 메서드를 추상으로 선언하거나 서브클래스에서 처리해야 함을 알리는 에러를 던진다.

### Before
```js
switch (bird.type) {
    case '유럽 제비':
        return "보통이다";
    case '아프리카 제비':
        return (bird.numberOfCoconuts > 2) ? "지쳤다" : "보통이다";
    case '노르웨이 파랑 앵무':
        return (bird.voltage > 100) ? "그을렸다" : "예쁘다";
    default:
        return "알 수 없다";
}
```

### After
```js
class EuropeanSwallow {
    get plumage() {
        return "보통이다";
    }
    ...
class AfricanSwallow {
    get plumage() {
        return (this.numberOfCoconuts > 2) ? "지쳤다" : "보통이다";
    }
    ...
class NorwegianBlueParrot {
    get plumage() {
        return (this.voltage > 100) ? "그을렸다" : "예쁘다";
    }
    ...
```

## 10.5 특이 케이스 추가하기

### 절차
1.

### Before
```js
```

### After
```js
```

## 10.6 어서션 추가하기

### 절차
1.

### Before
```js
```

### After
```js
```

## 10.7 제어 플래그를 탈출문으로 바꾸기

### 절차
1.

### Before
```js
```

### After
```js
```