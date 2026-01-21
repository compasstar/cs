# Stream API

## ✅ Stream 생성

### 1. 컬렉션 생성
```java
List<String> list = List.of("a", "b", "c");
Stream<String> stream1 = list.stream();
```

### 2. 배열 생성
```java
String[] arr = {"a", "b", "c"};
Stream<String> stream2 = Arrays.stream(arr);
```

### 3. Stream.of() 사용
```java
Stream<String> stream3 = Stream.of("a", "b", "c");
```

### 4. 무한 스트림 생성 - iterate()
```java
Stream<Integer> infiniteStream = Stream.iterate(0, n -> n + 2);
infiniteStream.limit(3).forEach(System.out::println);
```

### 5. 무한 스트림 생성 - generate()
```java
Stream<Double> randomStream = Stream.generate(Math::random);
randomStream.limit(3).forEach(System.out::println);
```


## ✅ Stream 중간 연산

### 1. filter
```java
numbers.stream()
        .filter(n -> n % 2 == 0)
        .forEach(n -> System.out.print(n + " "));
```

### 2. map
```java
numbers.stream()
        .map(n -> n * n)
        .forEach(n -> System.out.print(n + " "));
```

### 3. distinct (중복제거)
```java
numbers.stream()
        .distinct()
        .forEach(n -> System.out.print(n + " "));
```

### 4. sorted (기본정렬)
```java
Stream.of(3, 1, 4, 1, 5, 9, 2, 6, 5)
        .sorted()
        .forEach(n -> System.out.print(n + " "));
```

### 5. sorted (커스텀 정렬)

```java
Stream.of(3, 1, 4, 1, 5, 9, 2, 6, 5)
        .sorted(Comparator.reverseOrder())
        .forEach(n -> System.out.print(n + " "));
```


### 6. peek (동작 확인용)
```java
numbers.stream()
        .peek(n -> System.out.print("before: " + n + ", "))
        .map(n -> n * n)
        .peek(n -> System.out.print("after: " + n + ", "))
        .limit(5)
        .forEach(n -> System.out.println("최종값: " + n));
```

### 7. limit (개수 제한)
```java
numbers.stream()
        .limit(5)
        .forEach(n -> System.out.println(n + " "));
```

### 8. skip (처음 n개 요소 건너뛰기)
```java
numbers.stream()
        .skip(5)
        .forEach(n -> System.out.println(n + " "));
```

### 9. takeWhile (조건을 만족하는 동안만 선택)
```java
//1, 2, 3, 4, 5, 1, 2, 3
numbers.stream()
        .takeWhile(n -> n < 5)
        .forEach(n -> System.out.println(n + " "));
//1, 2, 3, 4
```

### 10. dropWhile (조건을 만족하는 동안 건너뛰기)
```java
//1, 2, 3, 4, 5, 1, 2, 3
numbers.stream()
        .dropWhile(n -> n < 5)
        .forEach(n -> System.out.println(n + " "));
System.out.println("\n");
//5, 1, 2, 3
```

### 11. flatMap (리스트 펼치기)
```java
// [[1, 2], [3, 4], [5, 6]]
List<Integer> flatMapResult = outerList.stream()
        .flatMap(list -> list.stream())
        .toList();
// [1, 2, 3, 4, 5, 6]
```

## ✅ Stream 최종연산

### 1. collect - List 수집
```java
List<Integer> evenNumbers1 = numbers.stream()
        .filter(n -> n % 2 == 0)
        .collect(Collectors.toList());
```

### 2. toList()
```java
List<Integer> evenNumbers2 = numbers.stream()
        .filter(n -> n % 2 == 0)
        .toList();
```

### 3. toArray - 배열로 변환
```java
Integer[] arr = numbers.stream()
        .filter(n -> n % 2 == 0)
        .toArray(Integer[]::new);
```


### 4. forEach - 각 요소 처리
```java
numbers.stream()
        .limit(5)
        .forEach(n -> System.out.print(n + " "));
```

### 5. count - 요소 개수
```java
long count = numbers.stream()
        .filter(n -> n > 5)
        .count();
```

### 6. reduce - 요소들의 합
```java
// 초기값이 없는 reduce
Optional<Integer> sum1 = numbers.stream()
        .reduce((a, b) -> a + b);

// 초기값이 있는 reduce
int sum2 = numbers.stream()
        .reduce(100, (a, b) -> a + b);
```

### 7. min - 최솟값
```java
Optional<Integer> min = numbers.stream()
        .min(Integer::compareTo);
```

### 8. max - 최댓값
```java
Optional<Integer> max = numbers.stream()
        .max(Integer::compareTo);
```

### 9. findFirst - 첫 번째 요소
```java
Optional<Integer> first = numbers.stream()
        .filter(n -> n > 5)
        .findFirst();
```

### 10. findAny - 아무 요소나 하나 찾기
```java
Optional<Integer> any = numbers.stream()
        .filter(n -> n > 5)
        .findAny();
```

### 11. anyMatch - 조건을 만족하는 요소 존재 여부
```java
boolean hasEven = numbers.stream()
        .anyMatch(n -> n % 2 == 0);
```

### 12. allMatch - 모든 요소가 조건을 만족하는지
```java
boolean allPositive = numbers.stream()
        .allMatch(n -> n > 0);
```

### 13. noneMatch - 조건을 만족하는 요소가 없는지
```java
boolean noNegative = numbers.stream()
        .noneMatch(n -> n < 0);
```

## ✅ 기본형 특화 스트림

### 1. 기본형 특화 스트림 생성 (IntStream, LongStream, DoubleStream)

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

### 2. 타입 변환 메서드
```java
LongStream longStream = IntStream.range(1, 5).asLongStream();
DoubleStream doubleStream = IntStream.range(1, 5).asDoubleStream();
Stream<Integer> boxedStream = IntStream.range(1, 5).boxed();
```

### 3. 기본형 특화 매핑
```java
LongStream mappedLong = IntStream.range(1, 5)
        .mapToLong(i -> i * 10L);

DoubleStream mappedDouble = IntStream.range(1, 5)
        .mapToDouble(i -> i * 1.5);

Stream<String> mappedObj = IntStream.range(1, 5)
        .mapToObj(i -> "Number: " + i);
```

### 4. 객체 스트림 -> 기본형 특화 스트림으로 매핑
```java
Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5);
IntStream intStream = stream.mapToInt(i -> i);
```

## ✅ 다운스트림

### 0. 학생 리스트

```java
List<Student> students = List.of(
        new Student("Kim", 1, 85),
        new Student("Park", 1, 70),
        new Student("Lee", 2, 70),
        new Student("Han", 2, 90),
        new Student("Hoon", 3, 90),
        new Student("Ha", 3, 89)
);
```

### 1. 학년별로 학생들을 그룹화 해라
```java
Map<Integer, List<Student>> collect1 = students.stream()
        .collect(Collectors.groupingBy(Student::getGrade));

//collect1 = {1=[Student{name='Kim', grade=1, score=85}, Student{name='Park', grade=1, score=70}],
//2=[Student{name='Lee', grade=2, score=70}, Student{name='Han', grade=2, score=90}],
//3=[Student{name='Hoon', grade=3, score=90}, Student{name='Ha', grade=3, score=89}]}
```

### 2. 학년별로 가장 점수가 높은 학생을 구해라. reducing 사용
```java
Map<Integer, Optional<Student>> collect2 = students.stream()
        .collect(Collectors.groupingBy(
                Student::getGrade,
                Collectors.reducing((s1, s2) -> s1.getScore() > s2.getScore() ? s1 : s2)
        ));

//collect2 = {1=Optional[Student{name='Kim', grade=1, score=85}], 2=Optional[Student{name='Han', grade=2, score=90}], 3=Optional[Student{name='Hoon', grade=3, score=90}]}
```

### 3. 학년별로 가장 점수가 높은 학생을 구해라. maxBy 사용
```java
Map<Integer, Optional<Student>> collect3 = students.stream()
        .collect(Collectors.groupingBy(
                Student::getGrade,
                Collectors.maxBy(Comparator.comparingInt(Student::getScore))
        ));

//collect3 = {1=Optional[Student{name='Kim', grade=1, score=85}], 2=Optional[Student{name='Han', grade=2, score=90}], 3=Optional[Student{name='Hoon', grade=3, score=90}]}
```

### 4. 학년별로 가장 점수가 높은 학생의 이름을 구해라 (collectingAndThen + maxBy 사용)
```java
Map<Integer, String> collect4 = students.stream()
        .collect(Collectors.groupingBy(
                Student::getGrade,
                Collectors.collectingAndThen(
                        Collectors.maxBy(Comparator.comparingInt(Student::getScore)),
                        sOpt -> sOpt.get().getName()
                )
        ));

//collect4 = {1=Kim, 2=Han, 3=Hoon}
```
