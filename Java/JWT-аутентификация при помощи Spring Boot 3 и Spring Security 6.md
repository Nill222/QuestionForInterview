

Средний

15 мин

106K

[Веб-разработка*](https://habr.com/ru/hubs/webdev/)[Java*](https://habr.com/ru/hubs/java/)[Проектирование API*](https://habr.com/ru/hubs/api/)



## Введение

Если не любите теорию, переходите сразу [сюда](https://habr.com/ru/articles/784508/#%D0%BF%D1%80%D0%B0%D0%BA%D1%82%D0%B8%D0%BA%D0%B0).

Переход от базовых приложений к более сложным требует использования Spring Security для обеспечения безопасности. Новая версия, Spring Security 6, изменяет некоторые базовые реализации, а русскоязычных материалов на эту тему очень мало. В этой статье мы рассмотрим JWT-аутентификацию и авторизацию с помощью Spring Boot 3 и Spring Security 6, чтобы помочь начинающем разработчикам разобраться и начать пользоваться базовым функционалом этой библиотеки. Цель статьи - показать, как использовать JWT-аутентификацию с API-интерфейсами. Будет разобрано как базовое использование, так и ролевая модель. Статья подойдёт как новичкам, так и продвинутым разработчик для быстрой пошаговой интеграции пользователей в свои проекты.

## Теория

Про это уже рассказывали много раз, поэтому тут только краткая выжимка. JWT или Json Web Token - открытый формат для создания токенов доступа, который является самодостаточным, т.е. содержит в себе всю необходимую информацию для проверки своей подлинности и содержимого без обращения к каким-либо внешним источникам. Токен состоит из 3-х частей:

- **Заголовок** - хранит тип токена и алгоритм шифрования
    
- **Полезная нагрузка** - данные пользователя, разрешения и тд (может быть всё что угодно)
    
- **Подпись** - обеспечивает целостность данных, путём проверки, что токен не был изменён после создания
    

Наглядно устройство токена представлено на рисунке. [Подробнее про JWT.](https://jwt.io/introduction)

![Структура JWT](https://habrastorage.org/r/w1560/getpro/habr/upload_files/89e/51b/f4c/89e51bf4c2253f11e3e731562f0b3b2a.png "Структура JWT")

Структура JWT

Не маловажным является то, что полезную нагрузку из токена в таком представлении может расшифровать каждый кто его увидит, пользуясь любым jwt encoder-ом, например [jwt.io](https://jwt.io/). Про шифрование и скрытие токенов в этой статье не будет.

### Регистрация

При регистрации пользователя происходят следующие шаги:

1. Пользователь обращается к сервису с запросом на регистрацию
    
2. Переданные данные валидируются и на их основе создаётся объект пользователя, пароль шифруется при помощи `PasswordEncoder`
    
3. Данные пользователя сохраняются в базу данных при помощи jpa-репозитория
    
4. `JwtService` генерирует токен, который возвращается клиенту
    

### Вход в аккаунт

Процесс входа не сильно отличается.

1. Пользователь обращается с запросом на вход
    
2. Создаётся экземпляр объекта `UsernamePasswordAuthenticationToken`и при помощи `AuthenticationManager` происходят все необходимые проверки
    
3. Если всё произошло успешно - будет возвращён токен, если нет, ошибка 403
    

### Практика

Для демонстрации и проверки работоспособности приложения используется база данных H2.

#### Зависимости

Зависимости проекта

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- База данных -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- Утилиты -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springdoc</groupId>
        <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
        <version>2.2.0</version>
    </dependency>

    <!-- Безопасность -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-impl</artifactId>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-jackson</artifactId>
    </dependency>

    <!-- Тестирование -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>0.12.3</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>0.12.3</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>0.12.3</version>
        </dependency>
    </dependencies>
</dependencyManagement>

#### Подключение к базе данных

```
spring:
  application:
    name: security-security
  datasource:
    url: jdbc:h2:mem:security-security
    driverClassName: org.h2.Driver
    username: root
    password: root
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    properties:
      hibernate:
        format_sql: true

token:
  signing:
    key: 53A73E5F1C4E0A2D3B5F2D784E6A1B423D6F247D1F6E5C3A596D635A75327855
```

Не стоит забывать, что хранить ключ в pom файле не безопасно, если этот ключ попадёт в руки другого программиста, он сможет подделывать токены доступа, тем самым безопасность вашего приложения будет нарушена, будьте с этим очень аккуратны.

#### Создание моделей

Для работы со Spring Security нужно имплементировать интерфейс `UserDetails` - в нём инкапсулированы основные данные о пользователе, необходимые для процесса аутентификации и авторизации в Spring Security.

```
@Entity
@Builder
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Table(name = "users")
public class User implements UserDetails {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_id_seq")
    @SequenceGenerator(name = "user_id_seq", sequenceName = "user_id_seq", allocationSize = 1)
    private Long id;

    @Column(unique = true, nullable = false)
    private String username;

    @Column(nullable = false)
    private String password;

    @Column(unique = true, nullable = false)
    private String email;

    @Enumerated(EnumType.STRING)
    private Role role;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(new SimpleGrantedAuthority(role.name()));
    }

    @Override public boolean isAccountNonExpired() { return true; }
    @Override public boolean isAccountNonLocked() { return true; }
    @Override public boolean isCredentialsNonExpired() { return true; }
    @Override public boolean isEnabled() { return true; }
}
public enum Role {
    ROLE_USER,
    ROLE_ADMIN
}


В стандартном репозитории пропишем 3 дополнительных метода, для проверки уникальности данных перед регистрацией.

```
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
    boolean existsByUsername(String username);
    boolean existsByEmail(String email);
}
```

Далее необходимо создать DTO для регистрации, входа и передачи токена пользователю.

**Регистрация пользователя**

```
@Data
public class SignUpRequest {
    @Size(min = 5, max = 50)
    @NotBlank
    private String username;

    @Email
    @NotBlank
    private String email;

    @Size(max = 255)
    private String password;
}
```

**Авторизация пользователя**

```
@Data
public class SignInRequest {
    @Size(min = 5, max = 50)
    @NotBlank
    private String username;

    @Size(min = 8, max = 255)
    @NotBlank
    private String password;
}

```

**Ответ с токеном доступа**

@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class JwtAuthenticationResponse {
    private String token;
}

```

#### Сервисы

Для начала реализуем сервис для работы с jwt, все методы описаны комментариями.

```
@Service
public class JwtService {

    @Value("${token.signing.key}")
    private String jwtSigningKey;

    public String extractUserName(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        if (userDetails instanceof User u) {
            claims.put("id", u.getId());
            claims.put("email", u.getEmail());
            claims.put("role", u.getRole());
        }
        return buildToken(claims, userDetails);
    }

    public boolean isTokenValid(String token, UserDetails userDetails) {
        return extractUserName(token).equals(userDetails.getUsername()) && !isExpired(token);
    }

    private <T> T extractClaim(String token, Function<Claims, T> resolver) {
        return resolver.apply(extractAll(token));
    }

    private String buildToken(Map<String, Object> claims, UserDetails userDetails) {
        return Jwts.builder()
                .setClaims(claims)
                .setSubject(userDetails.getUsername())
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + 100000 * 60 * 24))
                .signWith(getKey(), SignatureAlgorithm.HS256)
                .compact();
    }

    private boolean isExpired(String token) {
        return extractClaim(token, Claims::getExpiration).before(new Date());
    }

    private Claims extractAll(String token) {
        return Jwts.parser()
                .setSigningKey(getKey())
                .build()
                .parseClaimsJws(token)
                .getBody();
    }

    private Key getKey() {
        byte[] bytes = Decoders.BASE64.decode(jwtSigningKey);
        return Keys.hmacShaKeyFor(bytes);
    }
}


#### Сервис работы с пользователями

```
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository repository;

    public User save(User user) {
        return repository.save(user);
    }

    public User create(User user) {
        if (repository.existsByUsername(user.getUsername()))
            throw new RuntimeException("Username already used");
        if (repository.existsByEmail(user.getEmail()))
            throw new RuntimeException("Email already used");
        return save(user);
    }

    public User getByUsername(String username) {
        return repository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found"));
    }

    public UserDetailsService userDetailsService() {
        return this::getByUsername;
    }

    public User getCurrentUser() {
        String username =
                SecurityContextHolder.getContext().getAuthentication().getName();
        return getByUsername(username);
    }
}

```

#### Сервис авторизации

Бины `PasswordEncoder` и `AuthenticationManager` будут созданы позже.

```
@Service
@RequiredArgsConstructor
public class AuthenticationService {

    private final UserService users;
    private final JwtService jwt;
    private final PasswordEncoder encoder;
    private final AuthenticationManager authManager;

    public JwtAuthenticationResponse signUp(SignUpRequest req) {
        User user = User.builder()
                .username(req.getUsername())
                .email(req.getEmail())
                .password(encoder.encode(req.getPassword()))
                .role(Role.ROLE_USER)
                .build();

        users.create(user);
        return new JwtAuthenticationResponse(jwt.generateToken(user));
    }

    public JwtAuthenticationResponse signIn(SignInRequest req) {
        authManager.authenticate(
            new UsernamePasswordAuthenticationToken(req.getUsername(), req.getPassword())
        );
        UserDetails user = users.userDetailsService().loadUserByUsername(req.getUsername());
        return new JwtAuthenticationResponse(jwt.generateToken(user));
    }
}
```

#### Кастомный фильтр

Фильтр наследует `OncePerRequestFilter`, что гарантирует единоразовый вызов фильтра для одного запроса.

@Component
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    public static final String HEADER = "Authorization";
    public static final String PREFIX = "Bearer ";

    private final JwtService jwt;
    private final UserService users;

    @Override
    protected void doFilterInternal(HttpServletRequest req,
                                    HttpServletResponse res,
                                    FilterChain chain)
            throws IOException, ServletException {

        String header = req.getHeader(HEADER);

        if (header == null || !header.startsWith(PREFIX)) {
            chain.doFilter(req, res);
            return;
        }

        String token = header.substring(PREFIX.length());
        String username = jwt.extractUserName(token);

        if (username != null &&
            SecurityContextHolder.getContext().getAuthentication() == null) {

            UserDetails user = users.userDetailsService().loadUserByUsername(username);

            if (jwt.isTokenValid(token, user)) {

                UsernamePasswordAuthenticationToken auth =
                        new UsernamePasswordAuthenticationToken(
                                user, null, user.getAuthorities()
                        );

                auth.setDetails(
                        new WebAuthenticationDetailsSource().buildDetails(req)
                );

                SecurityContext context = SecurityContextHolder.createEmptyContext();
                context.setAuthentication(auth);
                SecurityContextHolder.setContext(context);
            }
        }
        chain.doFilter(req, res);
    }
}
```

#### Конфигурация Spring Security

```
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
@RequiredArgsConstructor
public class SecurityConfiguration {

    private final JwtAuthenticationFilter jwtFilter;
    private final UserService users;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf(AbstractHttpConfigurer::disable)
                .cors(c -> c.configurationSource(req -> {
                    CorsConfiguration cfg = new CorsConfiguration();
                    cfg.setAllowedOriginPatterns(List.of("*"));
                    cfg.setAllowedMethods(List.of("GET","POST","PUT","DELETE","OPTIONS"));
                    cfg.setAllowedHeaders(List.of("*"));
                    cfg.setAllowCredentials(true);
                    return cfg;
                }))
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/auth/**").permitAll()
                        .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()
                        .requestMatchers("/admin/**").hasRole("ADMIN")
                        .anyRequest().authenticated()
                )
                .sessionManagement(sm -> sm.sessionCreationPolicy(STATELESS))
                .authenticationProvider(provider())
                .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    public PasswordEncoder encoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public AuthenticationProvider provider() {
        DaoAuthenticationProvider p = new DaoAuthenticationProvider();
        p.setUserDetailsService(users.userDetailsService());
        p.setPasswordEncoder(encoder());
        return p;
    }

    @Bean
    public AuthenticationManager authManager(AuthenticationConfiguration cfg) throws Exception {
        return cfg.getAuthenticationManager();
    }
}
```

Стоит обратить внимание на ограничение энпоинтов,

- `permitAll` - Эндпоинт доступен всем пользователям, и авторизованным и нет
    
- `authenticated` - Только авторизованные пользователи
    
- `hasRole` - Пользователь должен иметь конкретную роль, и, соответственно быть авторизованным
    
- `hasAnyRole` - Должен иметь одну из перечисленных ролей _(не представлено в коде)_
    

На самом деле, если вас не специфичная система безопасности в проекте, умение настраивать только `authorizeHttpRequests` хватит для успешной разработки приложения, до тех пор, пока вы не станете Middle разработчиком. Но если необходимо разобраться подробнее, про это можно почитать [тут](https://docs.spring.io/spring-security/reference/index.html).

#### Контроллеры

```
@RestController
@RequestMapping("/auth")
@RequiredArgsConstructor
@Tag(name = "Аутентификация")
public class AuthController {

    private final AuthenticationService authenticationService;

    @Operation(summary = "Регистрация пользователя")
    @PostMapping("/sign-up")
    public JwtAuthenticationResponse signUp(
            @RequestBody @Valid SignUpRequest request
    ) {
        return authenticationService.signUp(request);
    }

    @Operation(summary = "Авторизация пользователя")
    @PostMapping("/sign-in")
    public JwtAuthenticationResponse signIn(
            @RequestBody @Valid SignInRequest request
    ) {
        return authenticationService.signIn(request);
    }
}
```

Далее представлен код контроллеров созданных для демонстрации.

```
@RestController
@RequestMapping("/example")
@RequiredArgsConstructor
@Tag(name = "Аутентификация")
public class ExampleController {

    private final UserService service;

    @GetMapping
    @Operation(summary = "Доступен только авторизованным пользователям")
    public String example() {
        return "Hello, world!";
    }

    @GetMapping("/admin")
    @Operation(summary = "Доступен только авторизованным пользователям с ролью ADMIN")
    @PreAuthorize("hasRole('ADMIN')")
    public String exampleAdmin() {
        return "Hello, admin!";
    }

    @GetMapping("/get-admin")
    @Operation(summary = "Получить роль ADMIN (для демонстрации)")
    public void getAdmin() {
        service.getAdmin();
    }
}

```

#### Тестирование

Все конечные точки можно просмотреть в документации Swagger, для этого переходим на [http://localhost:8080/swagger-ui/index.html](http://localhost:8080/swagger-ui/index.html). Если всё сделано правильно, должен появится следующий интерфейс.

![[Pasted image 20251116144356.png]]
Интерфейс Swagger

Для тестирования будем использовать Postman.

Для начала убедимся, что без авторизации нет возможности обратится к защищённому эндпоинту, должен вернуться статус 403, а ответ быть пустым.

![Обращение к закрытому эндпоинту](https://habrastorage.org/r/w1560/getpro/habr/upload_files/f3e/853/e78/f3e853e783c2bb7d25ca8ba0f8e22da7.png "Обращение к закрытому эндпоинту")

Обращение к закрытому эндпоинту

Для того, что бы авторизоваться, нужно передать соответствующие данные в теле запроса, предварительно выбрав тип тела, в нашем случае JSON.

![Регистрация пользователь](https://habrastorage.org/r/w1560/getpro/habr/upload_files/6c8/adc/e71/6c8adce7149d800a1e38f9073ad8bcd4.png "Регистрация пользователь")

Регистрация пользователь

Если всё прошло успешно, приложение вернёт токен, если произошли какие-либо ошибки (ошибка валидации или занятое имя пользователя) вернётся статус _403_. _Для того, что бы отображать текст ошибок, нужно подключить библиотеку ControllerAdvice (можно дополнить Zalando Problem)._

Давайте сразу проверим что хранится внутри токена, для этого зайдём на [jwt.io](https://jwt.io/) и попробует декодировать токен.

![Содержание токена](https://habrastorage.org/r/w1560/getpro/habr/upload_files/7e0/341/294/7e0341294ab6df6ae6e28bc015e8bed4.png "Содержание токена")

Содержание токена

Так же, указав свой секретный ключ, есть возможность проверить валидность токена.

Полученный токен необходимо передавать в заголовках к каждому запросу, это можно сделать вручную, но Postman предоставляет простой интерфейс для аутентификации во время запросов, нам лишь необходимо выбрать типа и указать необходимые данные.

Настроить авторизацию можно на всё коллекцию, на папку и на конкретный запрос, для этого необходимо найти вкладку **_Authorization_** и выбрать необходимый тип аутентификации, в нашем случае **_Bearer Token_**.

![Выбор типа аутентификации](https://habrastorage.org/r/w1560/getpro/habr/upload_files/8b8/c56/81f/8b8c5681f0137f772cfc826c17258673.png "Выбор типа аутентификации")

Выбор типа аутентификации

В появившееся поле вводим токен.

![Указываем токен](https://habrastorage.org/r/w1560/getpro/habr/upload_files/d58/22c/881/d5822c881e7ae9eb26db132467f8ea38.png "Указываем токен")

Указываем токен

После этого Postman автоматически добавить нужный заголовок _(Authorization)_ с нужным содержанием, но как было сказано ранее, это так же можно сделать вручную.

![Заголовок с токеном](https://habrastorage.org/r/w1560/getpro/habr/upload_files/e46/1cb/eff/e461cbeff7cd4b992fcb485a3772db03.png "Заголовок с токеном")

Заголовок с токеном

После того как токен введён, повторим запрос.

![Запрос к защищённому ресурсу](https://habrastorage.org/r/w1560/getpro/habr/upload_files/d28/670/727/d28670727b437f3dcd88d8bdf6b2c001.png "Запрос к защищённому ресурсу")

Запрос к защищённому ресурсу

На этот раз запрос прошёл и вернул строку _"Hello, World!"_, что говорит о том, что система безопасности работает.

По аналогии, при авторизации необходимо указать username и пароль, если они верные, вернётся токен, а эндпоинт, требующий роль администратора не будет доступен до тех пор, пока пользователь не будет иметь соответсвующую роль.

#### Бонус

Postman предоставляет множество функций, которые упрощают тестирование приложений, одна из таких - переменные и тесты, используя их можно сильно сэкономить время.

Для начала нажмём на коллекцию и выберем вкладку **_Variables_**, после чего создадим 2 переменные:

|   |   |
|---|---|
|**Переменная**|**Значение**|
|url|[http://localhost:8080](http://localhost:8080/)|
|token||

Должно получится следующим образом.

![Настройка переменных](https://habrastorage.org/r/w1560/getpro/habr/upload_files/89d/2a2/879/89d2a2879c47b6b5fc7b5c756da301d8.png "Настройка переменных")

Настройка переменных

Переменные можно использовать в любом поле ввода Postman, они имеют следующий синтаксис `{{varname}}`. Нам необходимо указать переменную с токеном в качестве токена аутентификации для всей коллекции.

![Переменная как токен аутентификации](https://habrastorage.org/r/w1560/getpro/habr/upload_files/500/7a8/379/5007a8379a1c40162158d134f09e7bfa.png "Переменная как токен аутентификации")

Переменная как токен аутентификации

Если всё сделано верно, текст выделится другим цветом и при наведении отобразится информация о переменной.

Далее вернёмся к запросу регистрации, первым делом в URL заменим домен на переменную - `{{url}}/auth/sign-up`, тем самым сократив запись и, если в будущем потребуется протестировать приложение на другом домене, это можно будет сделать путём замены значения переменной, а не исправлением каждого запроса.

Вторым шагом необходимо внутри запроса перейти во вкладку **Tests** и вставить следующий код.

```
pm.test("Status code is 200", function () {    pm.response.to.have.status(200);});pm.test("Set token to variable", () => {    var responseJson = pm.response.json();    pm.collectionVariables.set("token", responseJson.token);})
```

Данный код создаёт 2 теста, первый проверяет что статус ответа 200, второй же получает из ответа токен и устанавливает его в переменную. Теперь, при выполнении запроса у вас появится новая вкладка **_Tests Result_** и если он показывает 2/2, значит оба теста прошли и токен установился в переменную.

![Прохождение тестов](https://habrastorage.org/r/w1560/getpro/habr/upload_files/4b1/ddd/851/4b1ddd85117e3d91c8bea63c05325428.png "Прохождение тестов")

Прохождение тестов

Тесты так же необходимо продублировать в запрос авторизации.

Теперь при регистрации и авторизации токен будет автоматически вставлять в переменную, что избавлять вас от рутинной работы. Так же в запросах регистрации и авторизации лучше установить **_Authorization_** на **_No Auth_**, что бы просроченные токены не мешали авторизации и регистрации.

Исходный код можно найти тут - [https://github.com/MinusD/SpringSecurity](https://github.com/MinusD/SpringSecurity).

#### Заключение

В результате получилось полноценное приложение с регистрацией, авторизацией и ролевой моделью. Если у вас есть предложение по улучшению кода или текста, жду вас в комментариях или в issues на GitHub.