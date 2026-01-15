
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

## ✅ 파라미터 2개 함수형 인터페이스

| 함수형 인터페이스 | 사용처 | 실행함수 |
| --- | --- | --- |
| BiFunction<T1, T2, R> | 입력 2개, 반환 1개 | R apply(T1 t1, T2 t2) |
| BiConsumer<T1, T2> | 입력 2개, 반환 X | void accept(T1 t1, T2 t2) |
| BiPredicate<T1, T2> | 입력 2개 검증 | boolean test(T1 t1, T2 t2) |

> Tri 로 매개변수 3개는 없다. 필요하면 만들어서 사용해도 되지만, 함수형 인터페이스에서 권장되지는 않는다.

### 예시코드
```java
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
add.apply(5, 10); // 15

BiConsumer<String, Integer> repeat = (c, n) -> {
    for (int i = 0; i < n; i++) {
        System.out.print(c);
    }
};
repeat.accept("*", 5); // *****

BiPredicate<Integer, Integer> isGreater = (a, b) -> a > b;
isGreater.test(10, 5); // true
```

## ✅ 기본형 반환 함수형 인터페이스

```java
//기본형 매개변수, IntFunction, LongFunction, DoubleFunction
IntFunction<String> function = x -> "숫자: " + x;
function.apply(100); // 100

//기본형 반환, ToIntFunction, ToLongFunction, ToDoubleFunction
ToIntFunction<String> toIntFunction = s -> s.length();
toIntFunction.applyAsInt("hello"); // 5

// 기본형 매개변수, 기본형 반환
IntToLongFunction intToLongFunction = x -> x * 100L;
intToLongFunction.applyAsLong(10); // 1000

// IntUnaryOperator: int -> int
IntUnaryOperator intUnaryOperator = x -> x * 100;
intUnaryOperator.applyAsInt(10); // 1000

// 기타, IntConsumer, IntSupplier, IntPredicate
```


## ✅ Stream API

### Stream 생성

#### 1. 컬렉션 생성
```java
List<String> list = List.of("a", "b", "c");
Stream<String> stream1 = list.stream();
```

#### 2. 배열 생성
```java
String[] arr = {"a", "b", "c"};
Stream<String> stream2 = Arrays.stream(arr);
```

#### 3. Stream.of() 사용
```java
Stream<String> stream3 = Stream.of("a", "b", "c");
```

#### 4. 무한 스트림 생성 - iterate()
```java
Stream<Integer> infiniteStream = Stream.iterate(0, n -> n + 2);
infiniteStream.limit(3).forEach(System.out::println);
```

#### 5. 무한 스트림 생성 - generate()
```java
Stream<Double> randomStream = Stream.generate(Math::random);
randomStream.limit(3).forEach(System.out::println);
```


### Stream 중간 연산

#### 1. filter
```java
numbers.stream()
        .filter(n -> n % 2 == 0)
        .forEach(n -> System.out.print(n + " "));
```

#### 2. map
```java
numbers.stream()
        .map(n -> n * n)
        .forEach(n -> System.out.print(n + " "));
```

#### 3. distinct (중복제거)
```java
numbers.stream()
        .distinct()
        .forEach(n -> System.out.print(n + " "));
```

#### 4. sorted (기본정렬)
```java
Stream.of(3, 1, 4, 1, 5, 9, 2, 6, 5)
        .sorted()
        .forEach(n -> System.out.print(n + " "));
```

#### 5. sorted (커스텀 정렬)

```java
Stream.of(3, 1, 4, 1, 5, 9, 2, 6, 5)
        .sorted(Comparator.reverseOrder())
        .forEach(n -> System.out.print(n + " "));
```


#### 6. peek (동작 확인용)
```java
numbers.stream()
        .peek(n -> System.out.print("before: " + n + ", "))
        .map(n -> n * n)
        .peek(n -> System.out.print("after: " + n + ", "))
        .limit(5)
        .forEach(n -> System.out.println("최종값: " + n));
```

#### 7. limit (개수 제한)
```java
numbers.stream()
        .limit(5)
        .forEach(n -> System.out.println(n + " "));
```

#### 8. skip (처음 n개 요소 건너뛰기)
```java
numbers.stream()
        .skip(5)
        .forEach(n -> System.out.println(n + " "));
```

#### 9. takeWhile (조건을 만족하는 동안만 선택)
```java
//1, 2, 3, 4, 5, 1, 2, 3
numbers.stream()
        .takeWhile(n -> n < 5)
        .forEach(n -> System.out.println(n + " "));
//1, 2, 3, 4
```

#### 10. dropWhile (조건을 만족하는 동안 건너뛰기)
```java
//1, 2, 3, 4, 5, 1, 2, 3
numbers.stream()
        .dropWhile(n -> n < 5)
        .forEach(n -> System.out.println(n + " "));
System.out.println("\n");
//5, 1, 2, 3
```

#### 11. flatMap (리스트 펼치기)
```java
// [[1, 2], [3, 4], [5, 6]]
List<Integer> flatMapResult = outerList.stream()
        .flatMap(list -> list.stream())
        .toList();
// [1, 2, 3, 4, 5, 6]
```

### Stream 최종연산


#### 1. collect - List 수집
```java
List<Integer> evenNumbers1 = numbers.stream()
        .filter(n -> n % 2 == 0)
        .collect(Collectors.toList());
```

#### 2. toList()
```java
List<Integer> evenNumbers2 = numbers.stream()
        .filter(n -> n % 2 == 0)
        .toList();
```

#### 3. toArray - 배열로 변환
```java
Integer[] arr = numbers.stream()
        .filter(n -> n % 2 == 0)
        .toArray(Integer[]::new);
```


#### 4. forEach - 각 요소 처리
```java
numbers.stream()
        .limit(5)
        .forEach(n -> System.out.print(n + " "));
```

#### 5. count - 요소 개수
```java
long count = numbers.stream()
        .filter(n -> n > 5)
        .count();
```

#### 6. reduce - 요소들의 합
```java
// 초기값이 없는 reduce
Optional<Integer> sum1 = numbers.stream()
        .reduce((a, b) -> a + b);

// 초기값이 있는 reduce
int sum2 = numbers.stream()
        .reduce(100, (a, b) -> a + b);
```

#### 7. min - 최솟값
```java
Optional<Integer> min = numbers.stream()
        .min(Integer::compareTo);
```

#### 8. max - 최댓값
```java
Optional<Integer> max = numbers.stream()
        .max(Integer::compareTo);
```

#### 9. findFirst - 첫 번째 요소
```java
Optional<Integer> first = numbers.stream()
        .filter(n -> n > 5)
        .findFirst();
```

#### 10. findAny - 아무 요소나 하나 찾기
```java
Optional<Integer> any = numbers.stream()
        .filter(n -> n > 5)
        .findAny();
```

#### 11. anyMatch - 조건을 만족하는 요소 존재 여부
```java
boolean hasEven = numbers.stream()
        .anyMatch(n -> n % 2 == 0);
```

#### 12. allMatch - 모든 요소가 조건을 만족하는지
```java
boolean allPositive = numbers.stream()
        .allMatch(n -> n > 0);
```

#### 13. noneMatch - 조건을 만족하는 요소가 없는지
```java
boolean noNegative = numbers.stream()
        .noneMatch(n -> n < 0);
```

### 기본형 특화 스트림

#### 1. 기본형 특화 스트림 생성 (IntStream, LongStream, DoubleStream)

```java
IntStream stream = IntStream.of(1, 2, 3, 4, 5);
IntStream range1 = IntStream.range(1, 6); // [1,2,3,4,5]
IntStream range2 = IntStream.rangeClosed(1, 5); // [1,2,3,4,5]

// 통계 관련 메서드 (sum, average, max, min, count)
int sum = IntStream.range(1, 6).sum();

// avewrage(): 평균값 계산
double avg = IntStream.range(1, 6)
        .average()
        .getAsDouble();

// summaryStatistics(): 모든 통계 정보
IntSummaryStatistics stats = IntStream.range(1, 6).summaryStatistics();
System.out.println("합계: " + stats.getSum());
System.out.println("평균: " + stats.getAverage());
System.out.println("최댓값: " + stats.getMax());
System.out.println("최솟값: " + stats.getMin());
System.out.println("개수: " + stats.getCount());
```

#### 2. 타입 변환 메서드
```java
LongStream longStream = IntStream.range(1, 5).asLongStream();
DoubleStream doubleStream = IntStream.range(1, 5).asDoubleStream();
Stream<Integer> boxedStream = IntStream.range(1, 5).boxed();
```

#### 3. 기본형 특화 매핑
```java
LongStream mappedLong = IntStream.range(1, 5)
        .mapToLong(i -> i * 10L);

DoubleStream mappedDouble = IntStream.range(1, 5)
        .mapToDouble(i -> i * 1.5);

Stream<String> mappedObj = IntStream.range(1, 5)
        .mapToObj(i -> "Number: " + i);
```

#### 4. 객체 스트림 -> 기본형 특화 스트림으로 매핑
```java
Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5);
IntStream intStream = stream.mapToInt(i -> i);
```
