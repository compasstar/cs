
## ✅ 즉시평가와 지연평가

### 즉시평가
```java
public void debug(Object message) {
    if (isDebug) {
        System.out.println("[DEBUG] " + message);
    }
}

public static void main(String[] args) {
    Logger logger = new Logger();
    logger.setDebug(false);
    logger.debug(100 + 200);
}
```
디버그 모드를 껐기에, 로그는 출력되지 않는다. 하지만, 그럼에도 `100 + 200` 연산은 실행된다. '즉시평가' 이기 때문이다. '지연평가'를 사용하면 `100 + 200` 연산을 막을 수 있다.

### 지연평가
```java
public void debug(Supplier<?> supplier) {
    if (isDebug) {
        System.out.println("[DEBUG] " + supplier.get());
    }
}

public static void main(String[] args) {
    Logger logger = new Logger();
    logger.setDebug(false);
    logger.debug(() -> 100 + 200);
}
```
람다(Supplier)로 파라미터를 전달하면 `supplier.get()` 이 실행이 되지 않기에 로그가 남지 않는다. 


## ✅ PECS (Producer Extends Consumer Super) - 제네릭 와일드카드

### 생산자(Producer) <? extends T>
- 컬렉션에서 값을 읽어 온다
- 읽기 에서는 하위타입으로 꺼낼 수 있다
- `? extends T` 사용

```java
void makeAnimalSound(List<? extends Animal> animals) {
    for (Animal a : animals) {
        a.makeSound();
    }
}
```
- 하위타입인 `List<Dog>`, `List<Cat>` 어느 동물 리스트가 오던 동작해야 한다

  
### 소비자(Consumer) <? super T>
- 컬렉션에서 값을 저장 한다
- 저장 에서는 상위타입으로 넣을 수 있다
- `? super T` 사용

```java
void addDogToList(List<? super Dog> dogs) {
  dogs.add(new Dog()); 
}
```
- `List<Dog> dogs` 에 `new Dog()` 추가 OK
- `List<Animal> animals`에 `new Dog()` 추가 OK
