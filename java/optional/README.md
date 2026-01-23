# Optional

## ✅ Optional 의 생성

```java
Optional.of("Hello"); // 값이 null 이 아님이 확실할 때
Optional.ofNullable(null); // 값이 null 일 수도 아닐 수도 있을 때
Optional.empty(); // 비어있는 Optional 명시적 생성
```

## ✅ Optional 값 획득

### 1. isPresent()

값이 있으면 true
```java
optValue.isPresent(); // true
optEmpty.isPresent(); // false
```

### 2. get()
값 내부 값을 꺼냄, 값이 없으면 예외 (NoSuchElementException)
```java
String getValue = optValue.get();
optEmpty.get(); // 예외 발생
```

### 3. orElse()
값이 있으면 그 값, 없으면 지정된 값 기본값 사용
```java
String value1 = optValue.orElse("기본값");
Object empty1 = optEmpty.orElse("기본값");
```

### 4. orElseGet()
값이 없을 때만 람다(Supplier)가 실행되어 기본값 생성
```java
String value2 = optValue.orElseGet(() -> {
    System.out.println("람다 호출 - optValue");
    return "New Value";
});
```

### 5. orElseThrow()
값이 있으면 반환, 없으면 예외 발생

```java
String value3 = optValue.orElseThrow(() -> new RuntimeException("값이 없습니다!"));
```

### 6. or()
값이 있으면 Optional 인채로 반환, 없으면 Supplier 가 제공하는 다른 Optional 반환
```java
Optional<String> result1 = optValue.or(() -> Optional.of("Fallback")); // Optional[Hello]
Optional<String> result2 = optEmpty.or(() -> Optional.of("Fallback")); // Optional[Fallback]
```


## ✅ Optional 값 처리

### 1. ifPresent()
값이 존재하면 Consumer 실행, 없으면 아무 일도 하지 않음

```java
optValue.ifPresent(v -> System.out.println("optValue 값: " + v));
optEmpty.ifPresent(v -> System.out.println("optEmpty 값: " + v));
```

### 2. ifPresentOfElse()
값이 있으면 Consumer 실행, 없으면 Runnable 실행
```java
optValue.ifPresentOrElse(
        v -> System.out.println("optValue 값: " + v),
        () -> System.out.println("optValue 는 비어있음")
);
optEmpty.ifPresentOrElse(
        v -> System.out.println("optEmpty 값: " + v),
        () -> System.out.println("optEmpty 는 비어있음")
);
```

### 3. map()
값이 있으면 Function 적용 후 Optional 로 반환, 없으면 Optional.empty()

``` java
Optional<Integer> lengthOpt1 = optValue.map(String::length); // Optional[5]
Optional<Integer> lengthOpt2 = optEmpty.map(String::length); // Optional.empty
```

### 4. flatMap()

map() 과 유사하나, 이미 Optional 을 반환하는 경우 중첩을 제거
```java
Optional<Optional<String>> nestedOpt = optValue.map(s -> Optional.of(s)); // Optional[Optional[Hello]]
Optional<String> flattenedOpt = optValue.flatMap(s -> Optional.of(s)); // Optional[Hello]
```

### 5. fileter()

값이 있고 조건을 만족하면 그 값을 그대로, 불만족시 Optional.empty()
```java
Optional<String> filtered1 = optValue.filter(s -> s.startsWith("H")); // Optional[Hello]
Optional<String> filtered2 = optValue.filter(s -> s.startsWith("X")); // Optional.emtpy
```

### 6. stream()

값이 있으면 단일 요소 스트림, 없으면 빈 스트림
```java
optValue.stream()
        .forEach(s -> System.out.println("optValue.stream() -> " + s));

// 값이 없으므로 실행 안 됨
optEmpty.stream()
        .forEach(s -> System.out.println("optValue.stream() -> " + s));
```
