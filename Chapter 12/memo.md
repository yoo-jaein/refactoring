# Chapter 12
상속 다루기

## 12.1 메서드 올리기

### 절차

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

### 절차

### Before
```js
class Party {...}

class Employee extends Party {
    constructor(name, id, monthlyCost) {
        super();
        this._id = id;
        this._name = name;
        this._monthlyCost = monthlyCost;
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
```

## 12.4 메서드 내리기

### 절차

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

### 절차

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

### 절차

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

### 절차

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

### 절차

### Before
```js
class Employee {...}
class Salesperson extends Employee {...}
```

### After
```js
class Employee {...}
```

## 12.

### 절차

### Before
```js

```

### After
```js

```

## 12.

### 절차

### Before
```js

```

### After
```js

```
