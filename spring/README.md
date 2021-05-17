### pom.xml에 테스트 디펜던시 라이브러리 추가하기

-   junit4: java 유닛 테스트 라이브러리
-   Mockito: Mock객체를 생성하기위한 라이브러리

``` xml
<!-- ================================================================ -->
<!--                                 Junit4                               -->
<!-- ================================================================ -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <scope>test</scope>
        </dependency>
<!-- ================================================================ -->
<!--                             Mockito                               -->
<!-- ================================================================ -->
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-core</artifactId>
            <version>3.8.0</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-inline</artifactId>
            <version>3.9.0</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-all</artifactId>
            <version>1.10.19</version>
            <scope>test</scope>
        </dependency>
```

### 어노테이션

#### @RunWith

-   스프링 부트에 속한 어노테이션(@Autowired, @Component, ...등)을 사용할 수 있도록 해주는 어노테이션.

``` java
@RunWith(SpringRunner.class)
```

#### @SpringBootTest

-   스프링 부트를 실행시켜 테스트를 진행시키도록 해주는 어노테이션.

``` java
@SpringBootTest(classes = Application.class)
```

package 하위에 있는 파일들을 스캔해서 모든 외부 configuration들의 위치를 읽고 자동으로 configuration을 설정하여 스프링 어플리케이션을 실행시켜 줍니다.

기존에 설정된 configuration을 변경하지 않고 테스트 가능한 biz 테스트에 적합합니다.

#### @ContextConfiguration

-   스프링 통합 테스트에 필요한 ApplicationContext를 로드시켜주는 어노테이션

``` java
@ContextConfiguration( classes = Application.class )
```

SpringBootTest와 다르게 지정한 class를 기준으로 confiration을 설정하고 스프링 어플리케이션을 실행시켜주게 합니다. controller테스트 ( MVC 테스트 )와 같이 HTTP 요청/응답을 모킹해야하는 경우에 사용하기 적합합니다.

#### @WebMvcTest

-   스프링 MVC 테스트를 위한 어노테이션. HTTP 요청/응답을 모킹할 수있게 도와줍니다.

``` java
@WebMvcTest( controllers = ApprovalPathController.class)
```

#### @Mock
-   Mockito.mock 함수로 Mock Ojbect를 생성하는 것과 동일
-   모킹된 인스턴스를 생성후 주입

``` java
@Mock
List<String> mockedList;
```


#### @MockBean
-   @Mock과 비슷하지만 Spring Context (IoC Container)의 타입에 맞는 인스턴스를 주입한다.

``` java
@MockBean
UserRepository mockRepository;
```

#### @InjectMocks

- @Mock 또는 @MockBean으로 생성된 Mock Object를 주입 시켜 줍니다.

``` java
@InjectMocks
```


#### @Spy
- @Mock과 다르게 실제 객체를 사용하지만, 일부 메소드를 Stubbing할 수 있습니다.

``` java
@Spy
```

### Stubbing 하기
``` java
when(메소드(INPUT)).thenReturn(Output)
```

### 검증 하기
``` java
verify(Mock, 호출 횟수).메소드(INPUT)
```

### Biz 테스트 세팅하기

#### 1\. 어노테이션 세팅

``` java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class JoinUsBizTest {
```

#### 2\. 테스트할 biz에서 사용한 객체 모킹하기

``` java


    @Mock
    private UserBiz userBiz;

    @Mock
    private PrivacyPolicyConsentBiz privacyPolicyConsentBiz;

    @Mock
    private UserGrpRscGrpRelBiz userGrpRscGrpRelBiz;

    @Mock
    private MailValidationBiz mailValidationBiz;
```

#### 3\. biz에 모킹한 객체 주입 및 스파이 설정하기

``` java
 @InjectMocks
 @Spy
 private JoinUsBiz joinUsBiz;
```

### Controller 테스트 세팅하기

#### 1\. 어노테이션 세팅

``` java
@RunWith(SpringRunner.class)
@WebMvcTest(controllers = ApprovalPathController.class)
@ContextConfiguration( classes = Application.class )
public class ApprovalPathControllerTest {
```

#### 2\. 테스트할 biz에서 사용한 객체 모킹하기

``` java
    @MockBean
    private IApprovalPathBiz approvalPathBiz;

    @MockBean
    private CustomInterceptor customInterceptor;
```

#### 3\. controller에 모킹한 객체 주입하기

``` java
@InjectMocks
private ApprovalPathController approvalPathController;
```

#### 4\. MockMvc 설정하기

``` java
 private MockMvc mvc;

 @Before
    public void setup() throws Exception {
        MockitoAnnotations.openMocks(this);
        mvc = MockMvcBuilders
                .standaloneSetup(approvalPathController)
                .addInterceptors(customInterceptor).build();

        when(customInterceptor.preHandle(any(), any(), any())).thenReturn(true);
    }

```

-   CustomInterceptor를 거치게 되면, 세션이 없어 컨트롤러 진입이 막히지만, 모킹한 customIntercepter를 삽입하여 컨트롤러에 진입가능하도록 변경합니다.