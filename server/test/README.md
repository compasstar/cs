
## MVC 계층별 테스트

### Repository
- `@DataJpaTest` 
- SpringDataJpa 와 같은 메서드는 테스트 필요 X, 커스텀 쿼리만 테스트

### Service
- `@Mock`, `@InjectMocks` 
- 비즈니스 로직은 테스트 필수

### Controller
- `@WebMvcTest`, `MockMvc`
- API 응답, 상태코드, Validation 등 테스트 권장


## 코드예시

### Repository

```java
@DataJpaTest //내장 DB 사용으로 실제 DB 에 영향 X
class UserRepositoryTest {
  @Autowired
  private TestEntityManager entityManager;

  @Autowired
  private UserRepository repository

  @Test
  void findByName_존재하는_유저_조회() {
    //given
    User user = new User();
    user.setName("dasak");
    entityManager.persist(user);

    //when
    Optional<User> found = repository.findByName("dasak");

    //then
    assertTure(found.isPresent());
    assertEquals("dasak", found.get().getName());
  }
}
```

### Service

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

  @Mock
  private UserRepository repository; //Mock 객체를 통해 껍데기 리포지토리 생성

  @InjectMocks
  private UserService service; //UserService 에 @Mock repository 주입

  @Test
  void getUserByName_정상조회() {
    User user = new User();
    user.setName("dasak");

    //Mock 설정
    when(repository.findByName("dasak")).thenReturn(Optional.of(user));

    User result = service.getUserByName("dasak");

    assertEquals("dasak", result.getName());
  }

  @Test
  void getUserByName_없는_사용자_조회() {
    when(repository.findByName("unknown")).thenReturn(Optional.empty());

    assertThrows(RuntimeException.class, () -> service.getUserByName("unknown"));
  }
}
```


### Controller

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

  @Autowired
  private MockMvc mockMvc;

  @MockBean
  private UserService service; //Mock 서비스 생성

  @Test
  void getUser_성공응답() throws Exception {
    User user = new User();
    user.setId(1L);
    user.setName("dasak");

    when(service.getUserByName("dasak")).thenReturn(user);

    mockMvc.perform(get("/users/dasak"))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.name").value("dasak"));
  }
}
```
