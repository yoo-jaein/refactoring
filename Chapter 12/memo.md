# Chapter 12
상속 다루기

## 12.1 메서드 올리기

메서드 올리기를 적용하기 가장 쉬운 상황은 메서드들의 본문 코드가 똑같을 때다. 이럴 땐 그냥 복사해 붙여넣으면 끝이다. 반면, 메서드 올리기를 적용하기에 가장 복잡한 상황은 해당 메서드의 본문에서 참조하는 필드들이 서브클래스에만 있는 경우다. 이런 경우라면 필드를 먼저 슈퍼클래스로 올린 후에 메서드를 올려야 한다.  

두 메서드의 전체 흐름은 비슷하지만 세부 내용이 다르다면 템플릿 메서드(template method) 만들기를 고려해보자.  

### 절차
1. 똑같이 동작하는 메서드인지 면밀히 살펴본다.
2. 메서드 안에서 호출하는 다른 메서드와 참조하는 필드들을 슈퍼클래스에서도 호출하고 참조할 수 있는지 확인한다.
3. 메서드 시그니처가 다르다면 함수 선언 바꾸기로 슈퍼클래스에서 사용하고 싶은 형태로 통일한다.
4. 슈퍼클래스에 새로운 메서드를 생성하고, 대상 메서드의 코드를 복사해넣는다.
5. 정적 검사를 수행한다.
6. 서브클래스 중 하나의 메서드를 제거한다.
7. 테스트한다.
8. 모든 서브클래스의 메서드가 없어질 때까지 다른 서브클래스의 메서드를 하나씩 제거한다.

### Before
```js
class Employee {...}

class Salesperson extends Employee {
    get name() {...}
}

class Engineer extends Employee {
    get name() {...}
}
```

### After
```js
class Employee {
    get name() {...}
}

class Salesperson extends Employee {...}
class Engineer extends Employee {...}
```

## 12.2 필드 올리기

### 절차
1. 후보 필드들을 사용하는 곳 모두가 그 필드들을 똑같은 방식으로 사용하는지 면밀히 살핀다.
2. 필드들의 이름이 각기 다르다면 똑같은 이름으로 바꾼다.
3. 슈퍼클래스에 새로운 필드를 생성한다. 서브클래스에서 이 필드에 접근할 수 있어야 하기 때문에 대부분의 언어에서 protected로 선언하면 된다.
4. 서브클래스의 필드들을 제거한다.
5. 테스트한다.

### Before
```java
class Employee {...}

class Salesperson extends Employee {
    private String name;
}

class Engineer extends Employee {
    private String name;
}
```

### After
```java
class Employee {
    protected String name;
}

class Salesperson extends Employee {...}
class Engineer extends Employee {...}
```

## 12.3 생성자 본문 올리기

서브클래스들에서 기능이 같은 메서드들을 발견하면 함수 추출하기와 메서드 올리기를 차례로 적용하여 슈퍼클래스로 옮길 수 있다. 그런데 그 메서드가 생성자라면 조금 다른 식으로 접근해야 한다.  

### 절차
1. 슈퍼클래스에 생성자가 없다면 하나 정의한다. 서브클래스의 생성자들에서 이 생성자가 호출되는지 확인한다.
2. 문장 슬라이드하기로 공통 문장 모두를 super() 호출 직후로 옮긴다.
3. 공통 코드를 슈퍼클래스에 추가하고 서브클래스들에서는 제거한다. 생성자 매개변수 중 공통 코드에서 참조하는 값들을 모두 super()로 건넨다.
4. 테스트한다.
5. 생성자 시작 부분으로 옮길 수 없는 공통 코드에는 함수 추출하기와 메서드 올리기를 차례로 적용한다.

### Before
```js
class Party { }

class Employee extends Party {
    constructor(name, id, monthlyCost) {
        super();
        this._id = id;
        this._name = name;
        this._monthlyCost = monthlyCost;
    }
}

class Department extends Party {
    constructor(name, staff) {
        super();
        this._name = name;
        this._staff = staff;
    }
}
```

### After
```js
class Party {
    constructor(name) {
        this._name = name;
    }
}

class Employee extends Party {
    constructor(name, id, monthlyCost) {
        super(name);
        this._id = id;
        this._monthlyCost = monthlyCost;
    }
}

class Departement extends Party {
    constructor(name, staff) {
        super(name);
        this._staff = staff;
    }
}
```

## 12.4 메서드 내리기

소수의 서브클래스와만 관련된 메서드는 슈퍼클래스에서 제거하고 해당 서브클래스에 추가하는 편이 깔끔하다.

### 절차
1. 대상 메서드를 모든 서브클래스에 복사한다.
2. 슈퍼클래스에서 그 메서드를 제거한다.
3. 테스트한다.
4. 이 메서드를 사용하지 않는 모든 서브클래스에서 제거한다.
5. 테스트한다.

### Before
```js
class Employee {
    get quota {...}
}

class Engineer extends Employee {...}
class Salesperson extends Employee {...}
```

### After
```js
class Employee {...}

class Engineer extends Employee {...}
class Salesperson extends Employee {
    get quota {...}
}
```

## 12.5 필드 내리기

소수의 서브클래스에서만 사용하는 필드는 해당 서브클래스로 옮긴다.

### 절차
1. 대상 필드를 모든 서브클래스에 정의한다.
2. 슈퍼클래스에서 그 필드를 제거한다.
3. 테스트한다.
4. 이 필드를 사용하지 않는 모든 서브클래스에서 제거한다.
5. 테스트한다.

### Before
```java
class Employee {
    private String quota;
}

class Engineer extends Employee {...}
class Salesperson extends Employee {...}
```

### After
```js
class Employee {...}

class Engineer extends Employee {...}
class Salesperson extends Employee {
    protected String quota;
}
```

## 12.6 타입 코드를 서브클래스로 바꾸기

소프트웨어 시스템에서는 비슷한 대상들을 특정 특성에 따라 구분해야 할 때가 자주 있다. 직원을 담당 업무로 구분하여 엔지니어, 관리자, 영업자 등으로 구분하는 일이 그 예시이다. 이런 일을 다루는 수단으로 타입 코드(type code) 필드가 있다. 타입 코드는 언어에 따라 열거형이나 심볼, 숫자 등으로 표현한다.  

타입 코드를 서브클래스로 바꾸는 것이 좋은 상황에는 두 가지가 있다. 첫째, 조건에 따라 다르게 동작하도록 해주는 다형성이 필요할 때. 두 번째, 특정 타입에서만 의미가 있는 필드가 메서드가 있을 때.  

객체지향 프로그래밍에서 인스턴스 변수에 저장된 값을 기반으로 메서드 내에서 타입을 명시적으로 구분하는 방식은 객체지향을 위반하는 것으로 간주되기 때문에 이 리팩터링을 적용하는 것이 좋다.  

### 절차
1. 타입 코드 필드를 자가 캡슐화한다.
2. 타입 코드 값 하나를 선택하여 그 값에 해당하는 서브클래스를 만든다. 타입 코드 게터 메서드를 오버라이드하여 해당 타입 코드의 리터럴 값을 반환하게 한다.
3. 매개변수로 받은 타입 코드와 방금 만든 서브클래스를 매핑하는 선택 로직을 만든다.
4. 테스트한다.
5. 타입 코드 값 각각에 대해 서브클래스 생성과 선택 로직 추가를 반복한다. 클래스 하나가 완성될 때마다 테스트한다.
6. 타입 코드 필드를 제거한다.
7. 테스트한다.
8. 타입 코드 접근자를 이용하는 메서드 모두에 메서드 내리기와 조건부 로직을 다형성으로 바꾸기를 적용한다.

### Before
```js
function createEmployee(name, type) {
    return new Employee(name, type);
}
```

### After
```js
function createEmployee(name, type) {
    switch (type) {
        case "engineer": return new Enginner(name);
        case "salesperson": return new Salesperson(name);
        case "manager": return new Manager(name);
    }
}
```

## 12.7 서브클래스 제거하기

### 절차
1. 서브클래스의 생성자를 팩터리 함수로 바꾼다.
2. 서브클래스의 타입을 검사하는 코드가 있다면 그 검사 코드에 함수 추출하기와 함수 옮기기를 차례로 적용하여 슈퍼클래스로 옮긴다. 하나 변경할 때마다 테스트한다.
3. 서브클래스의 타입을 나타내는 필드를 슈퍼클래스에 만든다.
4. 서브클래스를 참조하는 메서드가 방금 만든 타입 필드를 이용하도록 수정한다.
5. 서브클래스를 지운다.
6. 테스트한다.

### Before
```js
class Person {
    get genderCode() {return "X";}
}

class Male extends Person {
    get genderCode() {return "M";}
}

class Female extends Person {
    get genderCode() {return "F";}
}
```

### After
```js
class Person {
    get genderCode() {return this._genderCode;}
}
```

## 12.8 슈퍼클래스 추출하기

비슷한 일을 수행하는 두 클래스가 보이면 상속을 이용해서 비슷한 부분을 공통의 슈퍼클래스로 옮겨 담을 수 있다. 공통 부분이 데이터라면 필드 올리기를 활용하고, 동작이라면 메서드 올리기를 활용하면 된다.  

슈퍼클래스 추출하기의 대안으로는 클래스 추출하기가 있다. 어느 것을 선택하느냐는 중복 동작을 상속으로 해결하느냐 위임으로 해결하느냐에 달렸다.  

### 절차
1. 빈 슈퍼클래스를 만든다. 원래의 클래스들이 새 클래스를 상속하도록 한다.
2. 테스트한다.
3. 생성자 본문 올리기, 메서드 올리기, 필드 올리기를 차례로 적용하여 공통 원소를 슈퍼클래스로 옮긴다.
4. 서브클래스에 남은 메서드들을 검토한다. 공통되는 부분이 있다면 함수로 추출한 다음 메서드 올리기를 적용한다.
5. 원래 클래스들을 사용하는 코드를 검토하여 슈퍼클래스의 인터페이스를 사용하게 할지 고민해본다.

### Before
```js
class Department {
    get totalAnnualCost() {...}
    get name() {...}
    get headCount() {...}
}

class Employee {
    get annualCost() {...}
    get name() {...}
    get id() {...}
}
```

### After
```js
class Party {
    get name() {...}
    get annualCost() {...}
}

class Department extends Party {
    get annualCost() {...}
    get headCount() {...}
}

class Employee extends Party {
    get annualCost() {...}
    get id() {...}
}
```

## 12.9 계층 합치기

클래스 계층구조를 리팩터링하다 보면 어떤 클래스와 그 부모가 너무 비슷해져서 더는 독립적으로 존재할 필요가 없는 경우가 생긴다. 이 때 둘을 하나로 합친다.  

### 절차
1. 두 클래스 중 제거할 것을 고른다.
2. 필드 올리기와 메서드 올리기 혹은 필드 내리기와 메서드 내리기를 적용하여 모든 요소를 하나의 클래스로 옮긴다.
3. 제거할 클래스를 참조하던 모든 코드가 남겨질 클래스를 참조하도록 고친다.
4. 빈 클래스를 제거한다.
5. 테스트한다.

### Before
```js
class Employee {...}
class Salesperson extends Employee {...}
```

### After
```js
class Employee {...}
```

## 12.10 서브클래스를 위임으로 바꾸기

상속에는 단점이 있다. 첫째, 상속은 캡슐화를 위반한다. 부모의 구현이 자식들에게 노출되어 서로 강하게 결합되고, 부모를 변경할 때 자식들도 함께 변경될 확률이 높아진다. 두 번째, 상속은 설계를 유연하지 못하게 만든다. 상속은 부모와 자식 사이의 관계를 컴파일 시점에 결정한다. 런타임에 객체의 종류를 변경하는 것이 불가능하다. 세 번째, 상속은 한 번만 쓸 수 있다. 예를 들어, 사람 객체의 동작을 '나이대'와 '소득 수준'에 따라 달리 하고 싶다면 서브클래스는 젊은이와 어르신이 되거나, 혹은 부자와 서민이 되어야 한다. 둘 다는 안 된다.  

위임(delegate)은 상속의 문제를 모두 해결해준다. 위임은 객체 사이의 일반적인 관계이므로 상속보다 결합도가 훨씬 약하다. 그래서 상속(서브클래싱) 관련 문제에 직면하게 되면 이 리팩터링을 적용한다.  

### 절차
1. 생성자를 호출하는 곳이 많다면 생성자를 팩터리 함수로 바꾼다.
2. 위임으로 활용할 빈 클래스를 만든다. 이 클래스의 생성자는 서브클래스에 특화된 데이터를 전부 받아야 하며, 보통은 슈퍼클래스를 가리키는 역참조도 필요하다.
3. 위임을 저장할 필드를 슈퍼클래스에 추가한다.
4. 서브클래스 생성 코드를 수정하여 위임 인스턴스를 생성하고 위임 필드에 대입해 초기화한다. 이 작업은 팩터리 함수나 생성자에서 수행한다.
5. 서브클래스의 메서드 중 위임 클래스로 이동할 것을 고른다.
6. 함수 옮기기를 적용해 위임 클래스로 옮긴다. 원래 메서드에서 위임하는 코드는 지우지 않는다.
7. 서브클래스 외부에도 원래 메서드를 호출하는 코드가 있다면 서브클래스의 위임 코드를 슈퍼클래스로 옮긴다. 이때 위임이 존재하는지를 검사하는 보호 코드로 감싸야 한다. 호출하는 외부 코드가 없다면 원래 메서드는 죽은 코드가 되므로 제거한다.
8. 테스트한다.
9. 서브클래스의 모든 메서드가 옮겨질 때까지 5~8 과정을 반복한다.
10. 서브클래스들의 생성자를 호출하는 코드를 찾아서 슈퍼클래스의 생성자를 사용하도록 수정한다.
11. 테스트한다.
12. 서브클래스를 삭제한다.

### Before
```js
class Order {
    get daysToShip() {
        return this._warehouse.daysToShip;
    }
}

class PriorityOrder extends Order {
    get daysToShip() {
        return this._priorityPlan.daysToShip;
    }
}
```

### After
```js
class Order {
    get daysToShip() {
        return (this._priorityDelegate) ? this._priorityDelegate.daysToShip : this._warehouse.daysToShip;
    }
}

class PriorityOrderDelegate {
    get daysToShip() {
        return this._priorityPlan.daysToShip;
    }
}
```

## 12.11 슈퍼클래스를 위임으로 바꾸기

자바의 스택은 이번 리팩터링을 적용해야 하는 좋은 예다. 자바의 스택은 리스트를 상속하고 있는데, 리스트의 연산 중 스택에는 적용되지 않는 게 많음에도 그 모든 연산이 스택 인터페이스에 그대로 노출되어 있다. 이보다는 스택에서 리스트 객체를 필드에 저장해두고 필요한 기능만 위임했다면 더 멋졌을 것이다. 이렇게 슈퍼클래스의 기능들이 서브클래스에는 어울리지 않는다면 그 기능들을 상속을 통해 이용하면 안 된다는 신호다.  

### 절차
1. 슈퍼클래스 객체를 참조하는 필드를 서브클래스에 만든다. 이번 리팩터링을 끝마치면 슈퍼클래스가 위임 객체가 될 것이므로 이 필드를 '위임 참조'라 부르자. 위임 참조를 새로운 슈퍼클래스 인스턴스로 초기화한다.
2. 슈퍼클래스의 동작 각각에 대응하는 전달 함수를 서브클래스에 만든다. 물론 위임 참조로 전달한다. 서로 관련된 함수끼리 그룹으로 묶어 진행하며, 그룹을 하나씩 만들 때마다 테스트한다.
3. 슈퍼클래스의 동작 모두가 전달 함수로 오버라이드되었다면 상속 관계를 끊는다.

### Before
```js
class List {...}
class Stack extends List {...}
```

### After
```js
class Stack {
    constructor() {
        this._storage = new List();
    }
}

class List {...}
```
