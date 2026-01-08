
## ✅ 기본 함수형 인터페이스

| 함수형 인터페이스 | 사용처 | 실행함수 |
| --- | --- | --- |
| Function<T, R> | 입력 O, 반환 O | R apply(T t) |
| Consumer<T, R> | 입력 O, 반환 X | void accept(T t) |
| Supplier<T, R> | 입력 X, 반환 O | R get() |
| Runnable<T, R> | 입력 X, 반환 X | void run() |

### 예시코드
```java
Function<String, Integer> function = s -> s.length();
Integer length = function.apply("hello")

Consumer<String> consumer = s -> System.out.println(s);
consumer.accept("hello consumer");

Supplier<Integer> supplier = () -> new Random().nextInt(10);
Integer random = supplier.get();

Runnable runnable = () -> System.out.println("Hello Runnable");
runnable2.run();
```

## ✅ 특화 함수형 인터페이스

| 함수형 인터페이스 | 사용처 | 실행함수 |
| --- | --- | --- |
| Predicate<T> | 검증 | boolean test(T t) |
| UnaryOperator<T> | 입력과 출력 타입이 같은 단항 연산 | T apply(T t) |
| BinaryOperator<T> | 입력과 출력 타입이 같은 다항 연산 | T apply(T t1, T t2) |

> 물론 `Function`, `BiFunction` 으로 `Predicate`, `UnaryOperator`, `BinaryOperator` 를 대체할 수 있지만, 의도를 명시적으로 드러내기 위해 특화 함수형 인터페이스를 사용한다 

### 예시코드
```java
Predicate<Integer> predicate = value -> value % 2 == 0;
boolean result1 = predicate.test(10); //true

UnaryOperator<Integer> square = x -> x * x;
Integer result2 = square.apply(5); //25

BinaryOperator<Integer> addition = (a, b) -> a + b;
Integer result3 = addition.apply(1, 2); //3
```
