# Chapter 02 memo
챕터 2에서는 리팩터링 전반에 적용되는 원칙에 대해 이야기한다.

## 리팩터링의 정의
리팩터링은 코드베이스를 정리하거나 구조를 바꾸는 모든 작업인 재구성 중 특수한 형태.

리팩터링은 겉보기 동작은 그대로 유지한 채, __코드를 이해하고 수정하기 쉽도록__ 내부 구조를 변경하는 작업이다.

## 두 개의 모자

기능 추가 모자와 리팩터링 모자

전체 작업 시간이 짧다 해도, 항상 내가 쓰고 있는 모자가 무엇인지와 그에 따른 미묘한 작업 방식의 차이를 분명하게 인식해야 한다.

## 리팩터링하는 이유

1. 규칙적인 리팩터링은 코드의 구조를 지탱해준다.
2. 코드의 목적이 더 잘 드러나게, 내 의도를 명확하게 전달해준다.
3. 코드가 하는 일을 깊이 파악하게 되면서 버그를 쉽게 찾을 수 있다.
4. 한 마디로 정리하면 다음과 같다. 리팩터링하면 코드 개발 속도를 높일 수 있다.

## 구조를 튼튼히 하는 것과 신규 기능을 추가하는 것?

내부 설계가 잘 된 소프트웨어는 새로운 기능을 추가할 지점과 어떻게 고칠지를 쉽게 찾을 수 있다. 모듈화가 잘 되어 있으면 전체 코드베이스 중 작은 일부만 이해하면 된다.

## 리팩터링하는 시점

1. 기본적으로는 숨쉬는 것 처럼 리팩토링 해라!  
   - 삼진 리팩터링 - 비슷한 일을 세 번째 하게 되면 리팩터링한다.
   - 쓰레기 줍기 리팩터링
   - 이해를 위한 리팩터링  

2. 신규 기능 추가 직전에 해라!  
   - 준비를 위한 리팩터링
   
3. 오래 걸리는 리팩터링  
   
4. 코드 리뷰  

## 리팩토링과 함께 하자
CI/CD(지속적 통합/배포) → 머지 충돌 방지

테스팅 → Tdd(테스트코드+리팩터링)

Agile, XP, Pair

## 리팩터링과 성능

하드 리얼타임 시스템을 제외하곤, 튜닝하기 쉽게 리팩터링한 후 속도를 위한 튜닝을 한다. 코드베이스 중 극히 일부에서 대부분의 시간을 소비한다. 따라서 리팩터링-프로파일러 분석-성능에 큰 영향을 주는 부분만 집중 최적화를 적용한다.

Java의 성능 최적화 툴 IntelliJ에 내장가능한 VisualVM

## 자동화된 리팩터링
IDE, 인텔리제이, 이클립스, 이멕스
