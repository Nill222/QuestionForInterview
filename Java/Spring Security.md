**Spring Security** — это главный фреймворк для защиты приложений на платформе Spring. Он отвечает за:

- **Аутентификацию (Authentication)** — проверку личности пользователя (логин/пароль, OAuth2-токен, JWT).
    
- **Авторизацию (Authorization)** — проверку прав доступа: что конкретный пользователь может делать в системе.
    
- **Защиту от атак**: CSRF (подделка запросов), session fixation (фиксация сессии), clickjacking, brute-force и т. д.
    

Почему это важно?

- Современные приложения практически всегда обрабатывают персональные или корпоративные данные.
    
- Уязвимость в безопасности = прямые убытки, утечки и репутационный удар.
    
- Spring Security считается стандартом: на сегодняшний день подавляющее большинство продакшн-приложений на Spring используют его.
    

Spring Security встроен глубоко в экосистему Spring Boot: достаточно добавить зависимость `spring-boot-starter-security`, и по умолчанию всё приложение защищено (доступ разрешён только аутентифицированным пользователям).

> Основная задача разработчика — настроить и расширить базовую защиту под конкретные требования: web-приложение с формой логина, REST API с JWT, микросервисы с OAuth2 и т. д.

### 2. Архитектура Spring Security

Архитектура Spring Security построена на **цепочке фильтров (Filter Chain)**.

#### 2.1. Как проходит запрос

1. **Клиент** (браузер, мобильное приложение, другой сервис) формирует HTTP-запрос.
    
    - Если включён HTTPS — трафик шифруется.
        
    - Запрос может содержать `Authorization: Bearer <token>` или `Cookie: JSESSIONID`.
        
2. **Серверное приложение (Spring Boot)** получает запрос.
    
    - До контроллеров запрос попадает в **цепочку фильтров (Filter Chain)**.
        
3. **Spring Security Filter Chain** проверяет:
    
    - аутентифицирован ли пользователь?
        
    - какие права у него есть?
        
    - разрешён ли доступ к ресурсу?
        
4. Если проверка не пройдена → ответ 401 (Unauthorized — нет аутентификации) или 403 (Forbidden — нет прав).
    
5. Если проверка успешна → запрос уходит в контроллер, сервис, БД.
    

#### 2.2. Ключевые элементы

- **DelegatingFilterProxy** — точка входа в Spring Security из контейнера сервлетов (Tomcat, Jetty и т. д.).
    
- **FilterChainProxy** — управляет списком фильтров.
    
- **SecurityFilterChain** — конфигурация безопасности для конкретного набора URL (например, `/api/**` защищается через JWT, а `/login` работает через форму).
    
- **AuthenticationManager** — главный менеджер, который отвечает за аутентификацию. Он делегирует проверку конкретным провайдерам (`AuthenticationProvider`).
    
- **UserDetailsService** — интерфейс, который загружает информацию о пользователе из БД (логин, пароль, роли).
    
- **PasswordEncoder** — отвечает за хэширование и проверку паролей (BCrypt и т. д.).
    
- **SecurityContext** — объект, где хранится информация о текущем пользователе (`Authentication`).
    

#### 2.3. Пример реальной цепочки фильтров

В приложении с формой логина Spring Security строит цепочку фильтров, которая включает:

1. `SecurityContextPersistenceFilter` — загружает данные о пользователе из сессии.
    
2. `CsrfFilter` — проверяет CSRF-токен.
    
3. `UsernamePasswordAuthenticationFilter` — перехватывает POST `/login`, проверяет логин/пароль.
    
4. `BasicAuthenticationFilter` — обрабатывает заголовок `Authorization: Basic ...`.
    
5. `BearerTokenAuthenticationFilter` — проверяет JWT или OAuth2-токен.
    
6. `AnonymousAuthenticationFilter` — если пользователь не залогинен, присваивает ему "анонимную" роль.
    
7. `ExceptionTranslationFilter` — обрабатывает ошибки безопасности.
    
8. `FilterSecurityInterceptor` — финальная проверка прав на доступ.
    

#### 2.4. Servlet vs Reactive

- **Servlet (Tomcat/Jetty/Undertow)**: используется классическая цепочка фильтров (`Filter`).
    
- **Reactive (WebFlux/Netty)**: применяется `WebFilter`, `SecurityWebFilterChain`, реактивный `ReactiveAuthenticationManager`. Отличие — контекст безопасности хранится в _reactive context_, а не в `ThreadLocal`.
    

### 3. Аутентификация и Авторизация

#### 3.1. Аутентификация (Authentication)

Аутентификация = процесс проверки личности пользователя.  
Spring Security отвечает на вопрос: **кто выполняет запрос?**

Основные сценарии:

1. **Форма логина (Form Login)**
    
    - Пользователь отправляет POST `/login` с `username` и `password`.
        
    - `UsernamePasswordAuthenticationFilter` перехватывает запрос.
        
    - `AuthenticationManager` вызывает `DaoAuthenticationProvider`.
        
    - `UserDetailsService` ищет пользователя в БД.
        
    - `PasswordEncoder` сверяет пароль (BCrypt, Argon2).
        
    - Если проверка успешна → создаётся объект `Authentication`, который сохраняется в `SecurityContext`.
        
2. **HTTP Basic**
    
    - Запрос содержит `Authorization: Basic base64(user:password)`.
        
    - Применяется в простых REST API (только через HTTPS).
        
3. **JWT (Bearer Token)**
    
    - В заголовке: `Authorization: Bearer <jwt>`.
        
    - Токен проверяется (`exp`, `iss`, `aud`, подпись).
        
    - Если валидный → создаётся `Authentication`.
        
    - Stateful-сессий нет, каждая проверка независимая.
        
4. **OAuth2 / OpenID Connect**
    
    - Пользователь логинится через внешнего провайдера (Google, GitHub).
        
    - Spring Security получает access token / id token.
        
    - Ресурсный сервер (resource server) проверяет токен (обычно JWT).
        
5. **LDAP / Active Directory**
    
    - Проверка пользователей через корпоративные директории.
        
6. **Кастомная аутентификация**
    
    - Можно написать свой `AuthenticationProvider`.
        
    - Например, проверка HMAC-подписи запроса или API-ключей.
        

#### 3.2. Авторизация (Authorization)

Авторизация = проверка прав доступа.  
Spring Security отвечает: **какие действия разрешены?**

Уровни:

1. **URL-уровень**  
    В конфигурации `HttpSecurity`:
    
    ```
    http.authorizeHttpRequests(auth -> auth    .requestMatchers("/admin/**").hasRole("ADMIN")    .requestMatchers("/user/**").hasAnyRole("USER", "ADMIN")    .anyRequest().authenticated());
    ```
    
    `/admin/**` доступен только ADMIN, `/user/**` доступен USER и ADMIN, остальные URL требуют аутентификации.
    
2. **Методный уровень**  
    Используются аннотации в сервисах:
    
    ```
    @PreAuthorize("hasRole('ADMIN')")public void deleteUser(Long id) { ... }
    ```
    
    Или более сложные выражения (SpEL):
    
    ```
    @PreAuthorize("#id == principal.id or hasRole('ADMIN')")public Account getAccount(Long id) { ... }
    ```
    
    Таким образом проверка может быть завязана не только на роли, но и на бизнес-логику.
    
3. **Доступ на уровне доменных объектов (ACL)**
    
    - Используется `AclService` и `PermissionEvaluator`.
        
    - Пример: только автор статьи может её редактировать.
        

#### 3.3. UserDetails и PasswordEncoder

1. **UserDetailsService**  
    Интерфейс, который возвращает объект `UserDetails`.
    
    Пример кастомной реализации:
    
    ```
    @Servicepublic class CustomUserDetailsService implements UserDetailsService {    private final UserRepository userRepository;    public UserDetails loadUserByUsername(String username) {        UserEntity user = userRepository.findByUsername(username)            .orElseThrow(() -> new UsernameNotFoundException("Not found"));        return new org.springframework.security.core.userdetails.User(            user.getUsername(),            user.getPassword(),            user.getRoles().stream().map(SimpleGrantedAuthority::new).toList()        );    }}
    ```
    
2. **PasswordEncoder**  
    Пароли никогда не хранятся в открытом виде.
    
    - `BCryptPasswordEncoder` (по умолчанию).
        
    - `Argon2PasswordEncoder` (ещё более современный).
        
    
    Пример:
    
    ```
    @BeanPasswordEncoder passwordEncoder() {    return new BCryptPasswordEncoder();}
    ```
    
    При регистрации пароль хэшируется, при логине — сравнивается хэш.
    

#### 3.4. Почему это безопасно?

- **TLS/HTTPS** защищает от перехвата трафика (человек посередине не увидит пароль).
    
- **Пароли хэшируются** (bcrypt/argon2): даже если украдут БД, пароль в открытом виде не получить.
    
- **Токены подписываются**: JWT содержит цифровую подпись, которую нельзя подделать без приватного ключа.
    
- **Авторизация многоуровневая**: даже если злоумышленник получил токен, доступ ограничен ролями и ACL.
    
- **Механизмы защиты от атак**: CSRF-токены, сессионная изоляция, ограничение попыток входа.
    

Аутентификация отвечает на вопрос **«кто выполняет запрос?»**.  
Авторизация отвечает на вопрос **«что разрешено делать?»**.  
Вместе они образуют основу Spring Security.

### 4. Жизненный цикл запроса в Spring Security

Чтобы уверенно работать со Spring Security, полезно понимать: **что происходит, когда клиент делает запрос к приложению**.

#### 4.1. Общая последовательность

1. Клиент (браузер, мобильное приложение, другой сервис) → отправляет запрос (GET/POST/PUT и т. д.).
    
2. Запрос попадает на сервер (Tomcat/Jetty/Undertow).
    
3. Контейнер запускает **цепочку фильтров (Filter Chain)**.
    
4. Среди фильтров есть `DelegatingFilterProxy`, который передаёт управление в Spring Security.
    
5. Запрос проходит через серию фильтров Spring Security.
    
6. Если аутентификация и авторизация успешные → запрос доходит до контроллера.
    
7. Контроллер выполняет бизнес-логику и возвращает ответ.
    
8. Ответ снова проходит через фильтры (часть данных сохраняется в сессию, добавляются заголовки безопасности).
    

#### 4.2. Детально: шаг за шагом

Предположим, используется **Spring Boot MVC + Form Login**.

**Шаг 1. Сетевой уровень**

- Клиент открывает страницу `/login`.
    
- TLS/HTTPS устанавливает зашифрованное соединение.
    
- Tomcat принимает запрос и превращает его в `HttpServletRequest`.
    

**Шаг 2. Контейнер запускает фильтры**

- Tomcat вызывает все фильтры, зарегистрированные в приложении.
    
- Среди них есть `DelegatingFilterProxy("springSecurityFilterChain")`.
    

**Шаг 3. Вход в Spring Security Filter Chain**

- `FilterChainProxy` определяет, какие фильтры применяются для данного URL.
    
- Например, для `/login` сработает `UsernamePasswordAuthenticationFilter`.
    

**Шаг 4. Работа фильтров**

Примерный порядок фильтров (по умолчанию):

1. `**SecurityContextPersistenceFilter**`  
    Загружает `SecurityContext` из сессии (если есть).  
    Если нет — создаёт новый (анонимный).
    
2. `**CsrfFilter**`  
    Проверяет наличие CSRF-токена (для POST/PUT/DELETE).
    
3. `**LogoutFilter**`  
    Перехватывает `/logout`, удаляет сессию, куки.
    
4. `**UsernamePasswordAuthenticationFilter**`  
    Если POST `/login` → берёт `username` и `password`.  
    Создаёт объект `UsernamePasswordAuthenticationToken`.  
    Отправляет его в `AuthenticationManager`.
    
5. `**AuthenticationManager**` **→** `**AuthenticationProvider**`  
    Например, `DaoAuthenticationProvider`.  
    Загружает пользователя через `UserDetailsService`.  
    Сравнивает пароль через `PasswordEncoder`.  
    Если проверка успешна → возвращает `Authentication` с ролями.
    
6. `**SecurityContextHolder**`  
    Сохраняет `Authentication` (в `ThreadLocal`).  
    Если приложение stateful → данные пишутся в `HttpSession`.
    
7. `**FilterSecurityInterceptor**`  
    Финальная проверка: есть ли у пользователя доступ к URL/методу.  
    Использует `AccessDecisionManager`.
    

Если где-то возникает ошибка (например, пароль неверный, токен невалидный) → выбрасывается `AuthenticationException`, и клиент получает 401/403.

**Шаг 5. Контроллер**

- Если проверки пройдены, `DispatcherServlet` вызывает контроллер.
    
- Например:
    
    ```
    @GetMapping("/admin")@PreAuthorize("hasRole('ADMIN')")public String adminPage() { ... }
    ```
    
- Если пользователь имеет роль `ADMIN` → метод выполнится.
    

**Шаг 6. Обратный путь (Response)**

- Контроллер возвращает `ResponseEntity` или `View`.
    
- Ответ снова проходит через фильтры (например, `HeaderWriterFilter` добавляет заголовки безопасности).
    
- Tomcat отправляет ответ клиенту.
    

#### 4.3. Вариации

- **JWT API (stateless)**  
    Нет `HttpSession`.  
    `BearerTokenAuthenticationFilter` проверяет токен в каждом запросе.  
    Контекст создаётся заново каждый раз.
    
- **WebFlux (reactive)**  
    Вместо `Filter` используется `WebFilter`.  
    Вместо `ThreadLocal SecurityContextHolder` → реактивный `Context`.  
    Вся работа выполняется асинхронно.
    

#### 4.4. Ключевой принцип

Вся безопасность в Spring Security реализуется **до выполнения бизнес-логики**.  
Если пользователь не прошёл проверку — контроллер даже не вызовется.

- Spring Security работает как **прослойка между клиентом и контроллером**.
    
- Каждый запрос проходит через цепочку фильтров.
    
- Аутентификация и авторизация выполняются ДО бизнес-логики.
    

### 5. Stateful vs Stateless (сессии и JWT)

Один из ключевых вопросов в безопасности: **как хранить и проверять информацию о пользователе между запросами**.

#### 5.1. Stateful (с состоянием, через сессии)

#### Как работает

1. Пользователь проходит аутентификацию (например, через форму).
    
2. Сервер проверяет логин/пароль.
    
3. При успешной аутентификации сервер сохраняет информацию о пользователе в **HttpSession**.
    
    - Это объект в памяти сервера.
        
    - В нём хранится `SecurityContext` с `Authentication`.
        
4. Клиенту отправляется **cookie (JSESSIONID)**.
    
5. При каждом запросе клиент передаёт cookie, сервер по нему восстанавливает пользователя из сессии.
    

#### Пример

- Первый вход → `POST /login`
    
- Ответ сервера → `Set-Cookie: JSESSIONID=abc123`
    
- Дальнейшие запросы → `Cookie: JSESSIONID=abc123`
    

#### Плюсы

- Простая модель.
    
- Безопасно для закрытых корпоративных приложений.
    
- Работает «из коробки» в Spring Security.
    

#### Минусы

- Масштабирование сложнее: при большом числе серверов необходимо **шарить сессии** (sticky sessions, Redis, Hazelcast).
    
- Неудобно для REST API и микросервисов.
    
- Плохо подходит для mobile-first и SPA (React, Angular, iOS/Android).
    

#### 5.2. Stateless (без состояния, через JWT или токены)

#### Как работает

1. Пользователь аутентифицируется (или получает токен через OAuth2).
    
2. Сервер **не хранит состояние в памяти**.
    
3. Клиент получает **JWT (JSON Web Token)**.
    
4. JWT хранится на стороне клиента (localStorage, secure cookie, mobile keystore).
    
5. При каждом запросе клиент передаёт заголовок:
    
    ```
    Authorization: Bearer <jwt-token>
    ```
    
6. Сервер проверяет токен (подпись, срок действия, права).
    

#### JWT структура

JWT состоит из трёх частей (разделённых точками):

```
header.payload.signature
```

- **Header** → алгоритм подписи (например, HS256).
    
- **Payload** → данные: `sub` (user id), `roles`, `exp` (срок жизни).
    
- **Signature** → проверка подлинности (HMAC или RSA).
    

#### Пример payload

```
{  "sub": "user123",  "roles": ["USER"],  "exp": 1736448900}
```

#### Плюсы

- Простое масштабирование: сервер stateless, количество инстансов не ограничено.
    
- Удобно для микросервисов.
    
- Подходит для любых клиентов (web, mobile, API).
    

#### Минусы

- JWT нельзя отозвать до истечения срока (если не хранить blacklist).
    
- Payload доступен в открытом виде (Base64, не зашифрован, только подписан).
    
- Требуется дополнительная схема для refresh-токенов.
    

#### 5.3. Refresh токены

Чтобы access-токен не был «вечным», часто используется схема с двумя токенами:

1. **Access Token** — короткоживущий (например, 15 минут).
    
2. **Refresh Token** — долгоживущий (например, 30 дней).
    

Алгоритм:

- Клиент получает оба токена при логине.
    
- Access Token используется в запросах.
    
- Когда срок действия истекает → клиент отправляет Refresh Token на `/refresh`.
    
- Сервер выдаёт новый Access Token.
    

#### 5.4. Spring Security реализация

- **Stateful (Session):**
    
    - `HttpSessionSecurityContextRepository`
        
    - Работает «по умолчанию» при form login.
        
- **Stateless (JWT):**
    
    - `BearerTokenAuthenticationFilter`
        
    - Настройка в `SecurityFilterChain`:
        
        ```
        http  .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)  .and()  .oauth2ResourceServer().jwt();
        ```
        

#### 5.5. Когда использовать что

- **Stateful (сессии):**
    
    - Внутренние корпоративные приложения.
        
    - Небольшое количество серверов.
        
    - Пользователи работают только через браузер.
        
- **Stateless (JWT):**
    
    - REST API для мобильных приложений.
        
    - Микросервисная архитектура.
        
    - Высокая нагрузка и требование к масштабируемости.
        

Stateful → сервер хранит состояние, проще, но хуже масштабируется.  
Stateless → клиент хранит токен, подходит для API и микросервисов.  
JWT — стандартное решение для распределённых систем, но не универсальная «панацея».

### 6. Пароли и PasswordEncoder — как хранить безопасно

Одна из самых частых ошибок начинающих разработчиков — хранение паролей в открытом виде (plain text) или использование простых хэшей вроде **MD5/SHA1**.  
Spring Security решает эту задачу через механизм **PasswordEncoder**.

#### 6.1. Почему нельзя хранить пароли в открытом виде

Пример таблицы пользователей:

|id|username|password|
|---|---|---|
|1|admin|qwerty123|
|2|user|123456|

Проблемы:

- Если база утечёт → можно сразу войти.
    
- Пользователи часто используют один и тот же пароль в других сервисах.
    
- Даже администраторы БД видят пароли, что создаёт риск инсайда.
    

#### 6.2. Хэширование вместо хранения «как есть»

Хранить необходимо не пароль, а **хэш**.

Пример (BCrypt):

```
$2a$10$DowJHd/8DqzRzTQaI7Em5Ocu8l7v.8dxyl0nHKz3Oy4rQ0cl6iTga
```

Преимущества:

- Невозможность «обратного разворота» (хэш — односторонняя функция).
    
- Разные хэши для одинаковых паролей (из-за соли).
    
- Замедление брутфорса (BCrypt специально «тормозной»).
    

#### 6.3. Алгоритмы в Spring Security

Доступные современные алгоритмы:

- **BCrypt** (`BCryptPasswordEncoder`)
    
    - Самый распространённый, используется по умолчанию.
        
    - Применяет соль + параметр сложности (cost).
        
- **Argon2** (`Argon2PasswordEncoder`)
    
    - Победитель Password Hashing Competition.
        
    - Устойчив к атакам с использованием GPU/ASIC.
        
- **PBKDF2** (`Pbkdf2PasswordEncoder`)
    
    - Медленный алгоритм на основе HMAC.
        
- **SCrypt** (`SCryptPasswordEncoder`)
    
    - Альтернатива BCrypt, более затратный по памяти.
        

Не рекомендуется использовать:

- MD5, SHA-1, SHA-256 → слишком быстрые, легко поддаются брутфорсу.
    

#### 6.4. PasswordEncoder в коде

Spring рекомендует использовать делегирующий энкодер:

```
@Beanpublic PasswordEncoder passwordEncoder() {    return PasswordEncoderFactories.createDelegatingPasswordEncoder();}
```

Он создаёт `DelegatingPasswordEncoder`, который по умолчанию = BCrypt, но поддерживает и другие алгоритмы.

Пример записи пароля в БД:

```
{bcrypt}$2a$10$DowJHd/8DqzRzTQaI7Em5Ocu8l7v.8dxyl0nHKz3Oy4rQ0cl6iTga
```

#### 6.5. Проверка пароля

Алгоритм:

1. Пользователь вводит пароль.
    
2. `PasswordEncoder.matches(rawPassword, storedHash)` сравнивает введённый пароль и хэш.
    
3. В Spring Security это происходит автоматически в `DaoAuthenticationProvider`.
    

Пример:

```
String raw = "qwerty123";String encoded = passwordEncoder.encode(raw);boolean matches = passwordEncoder.matches(raw, encoded); // true
```

#### 6.6. Best Practices

- Использовать **BCrypt** или **Argon2**.
    
- Минимальная длина пароля — 8–12 символов, лучше 12–16.
    
- Ограничение числа попыток входа (защита от brute force).
    
- Хранить только хэш, никогда не логировать «сырой» пароль.
    
- Регулярно обновлять алгоритмы (например, миграция с BCrypt → Argon2).
    
- Добавлять 2FA для критичных систем.
    

### 7. AuthenticationManager и UserDetailsService

Spring Security спроектирован так, чтобы аутентификацию можно было адаптировать под любые источники — SQL-БД, LDAP или OAuth2.  
В основе лежат три ключевых компонента:

- **AuthenticationManager** — главный «оркестратор» аутентификации.
    
- **AuthenticationProvider** — конкретный исполнитель проверки (например, БД или JWT).
    
- **UserDetailsService** — загрузчик информации о пользователе (обычно из БД).
    

#### 7.1. Authentication (объект аутентификации)

При входе пользователя Spring Security формирует объект `Authentication`.

До проверки:

```
UsernamePasswordAuthenticationToken [Principal=admin, Credentials=123456, Authenticated=false]
```

После успешной проверки:

```
UsernamePasswordAuthenticationToken [Principal=User(admin,ROLE_ADMIN), Credentials=[PROTECTED], Authenticated=true]
```

Ключевые поля:

- `Principal` → информация о пользователе (обычно UserDetails).
    
- `Credentials` → чем подтверждён вход (пароль, токен).
    
- `Authorities` → права (`ROLE_USER`, `ROLE_ADMIN`).
    

#### 7.2. AuthenticationManager

Интерфейс с единственным методом:

```
public interface AuthenticationManager {    Authentication authenticate(Authentication authentication) throws AuthenticationException;}
```

Назначение: принять запрос на аутентификацию и вернуть результат (`Authentication`) или выбросить исключение.

#### 7.3. AuthenticationProvider

`AuthenticationManager` делегирует проверку одному или нескольким **AuthenticationProvider**.

Примеры встроенных провайдеров:

- `DaoAuthenticationProvider` → проверка логина/пароля через `UserDetailsService`.
    
- `LdapAuthenticationProvider` → проверка в LDAP/Active Directory.
    
- `JwtAuthenticationProvider` → проверка JWT-токена.
    

#### 7.4. UserDetailsService

Интерфейс для загрузки пользователя из источника (например, БД):

```
public interface UserDetailsService {    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;}
```

Пример реализации (через JPA):

```
@Servicepublic class MyUserDetailsService implements UserDetailsService {    private final UserRepository userRepository;    @Override    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {        UserEntity user = userRepository.findByUsername(username)            .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));        return org.springframework.security.core.userdetails.User            .withUsername(user.getUsername())            .password(user.getPassword()) // уже захэшированный пароль!            .roles(user.getRole())            .build();    }}
```

#### 7.5. Как работает связка

1. Клиент отправляет `POST /login` с логином и паролем.
    
2. Фильтр `UsernamePasswordAuthenticationFilter` перехватывает запрос и создаёт `UsernamePasswordAuthenticationToken`.
    
3. `AuthenticationManager` передаёт токен в `DaoAuthenticationProvider`.
    
4. `DaoAuthenticationProvider`:
    
    - вызывает `UserDetailsService.loadUserByUsername(username)`
        
    - получает пользователя из БД
        
    - сравнивает пароль через `PasswordEncoder.matches()`
        
5. При успехе возвращается `Authentication` с `Authenticated=true`.
    
6. `SecurityContextHolder` сохраняет пользователя, и он доступен в контроллерах через `@AuthenticationPrincipal`.
    

#### 7.6. Пример SecurityConfig

```
@Configuration@EnableMethodSecuritypublic class SecurityConfig {    private final UserDetailsService userDetailsService;    private final PasswordEncoder passwordEncoder;    @Bean    public AuthenticationManager authenticationManager(HttpSecurity http) throws Exception {        return http            .getSharedObject(AuthenticationManagerBuilder.class)            .userDetailsService(userDetailsService)            .passwordEncoder(passwordEncoder)            .and()            .build();    }    @Bean    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {        http            .authorizeHttpRequests(auth -> auth                .requestMatchers("/admin/**").hasRole("ADMIN")                .anyRequest().authenticated()            )            .formLogin(Customizer.withDefaults());        return http.build();    }}
```

#### 7.7. Best Practices

- Всегда использовать `PasswordEncoder` при загрузке пользователей.
    
- Отдельные таблицы пользователей и ролей (user/role).
    
- Для кастомной логики (например, логин по email или номеру телефона) реализовать свой `UserDetailsService`.
    
- При необходимости подключать несколько `AuthenticationProvider` (например, для БД и JWT одновременно).
    

- `AuthenticationManager` — центральная точка аутентификации.
    
- `AuthenticationProvider` — конкретный механизм проверки.
    
- `UserDetailsService` — загрузка пользователя из источника данных.
    
- В связке они дают гибкую систему аутентификации в Spring Security.
    

### 8. SecurityContext и SecurityContextHolder

Когда пользователь прошёл аутентификацию, Spring Security должен где-то хранить информацию о нём, чтобы:

- не проверять пароль заново на каждый запрос,
    
- быстро понимать, кто сделал запрос,
    
- проверять права доступа в контроллерах и сервисах.
    

Для этого используется связка:

- **SecurityContext** — контейнер, где лежит информация о текущем пользователе.
    
- **SecurityContextHolder** — утилита, которая даёт доступ к текущему `SecurityContext`.
    

#### 8.1. SecurityContext

`SecurityContext` — это объект, который хранит только одну вещь: **Authentication**.

Пример:

```
SecurityContext context = SecurityContextHolder.getContext();Authentication auth = context.getAuthentication();
```

Внутри `Authentication` лежит:

- `Principal` — пользователь (обычно UserDetails).
    
- `Authorities` — список ролей/прав (`ROLE_USER`, `ROLE_ADMIN`).
    
- `Details` — доп. данные (например, IP, sessionId).
    

#### 8.2. SecurityContextHolder

Это **глобальное хранилище**, через которое всегда получаем доступ к текущему пользователю.

```
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();String username = authentication.getName();
```

`SecurityContextHolder` обычно работает через **ThreadLocal**:

- Для каждого запроса создаётся отдельный поток (в servlet-модели).
    
- В этом потоке хранится `SecurityContext`.
    
- Когда запрос завершился → контекст очищается.
    

#### 8.3. Варианты хранения (Modes)

Spring Security поддерживает несколько стратегий хранения контекста:

1. **MODE_THREADLOCAL** (по умолчанию)
    
    - Каждый поток имеет свой SecurityContext.
        
    - Самый популярный вариант.
        
2. **MODE_INHERITABLETHREADLOCAL**
    
    - Дочерние потоки наследуют контекст родителя.
        
    - Нужно редко (например, при async-задачах).
        
3. **MODE_GLOBAL**
    
    - Один общий контекст для всех.
        
    - Почти никогда не используется (небезопасно).
        

#### 8.4. SecurityContextPersistenceFilter

Кто вообще сохраняет и достаёт SecurityContext?

Это делает фильтр `SecurityContextPersistenceFilter`:

- В начале запроса он загружает контекст (из сессии или создаёт новый).
    
- В конце запроса он сохраняет обновлённый контекст обратно.
    

Для stateless (JWT) — новый контекст создаётся на каждый запрос.  
Для stateful (сессии) — контекст достаётся из `HttpSession`.

#### 8.5. Доступ к пользователю в контроллерах

Spring Security даёт несколько способов получить текущего юзера:

#### Через Authentication

```
@GetMapping("/me")public String me(Authentication auth) {    return "Hello " + auth.getName();}
```

#### Через SecurityContextHolder

```
@GetMapping("/me")public String me() {    Authentication auth = SecurityContextHolder.getContext().getAuthentication();    return "Hello " + auth.getName();}
```

#### Через аннотацию @AuthenticationPrincipal

```
@GetMapping("/me")public String me(@AuthenticationPrincipal UserDetails user) {    return "Hello " + user.getUsername();}
```

#### 8.6. Пример в сервисах

Можно использовать пользователя не только в контроллерах, но и в сервисах:

```
@Servicepublic class AccountService {    public void doSomething() {        Authentication auth = SecurityContextHolder.getContext().getAuthentication();        String user = auth.getName();        System.out.println("Action by: " + user);    }}
```

#### 8.7. Особенности в WebFlux (реактивных приложениях)

В реактивных приложениях **нет ThreadLocal**, поэтому `SecurityContextHolder` не работает напрямую.  
Вместо этого используется **ReactiveSecurityContextHolder**, который хранит контекст в Reactor Context.

Пример:

```
Mono<String> currentUser = ReactiveSecurityContextHolder.getContext()    .map(ctx -> ctx.getAuthentication().getName());
```

#### 8.8. Best Practices

- Использовать `@AuthenticationPrincipal` для чистоты кода в контроллерах.
    
- Избегать прямого вызова `SecurityContextHolder` везде — лучше в сервисах.
    
- В реактивных приложениях всегда использовать `ReactiveSecurityContextHolder`.
    
- Для асинхронных задач (например, `@Async`) → использовать `DelegatingSecurityContextExecutorService`, чтобы прокидывать контекст в другие потоки.
    

- `SecurityContext` хранит текущего пользователя.
    
- `SecurityContextHolder` даёт доступ к нему.
    
- В Servlet (stateful/stateless) используется ThreadLocal, в WebFlux — Reactor Context.
    
- После логина вся инфа о пользователе живёт именно тут.
    

### 9. Авторизация в Spring Security

Аутентификация отвечает на вопрос: **«Кто это?»**, а авторизация — **«Что этот пользователь может?»**.

Spring Security делает это через цепочку: **Voters → AccessDecisionManager → фильтры / методы**.

#### 9.1. Основные понятия

**Authentication (аутентификация)**

- Проверка личности пользователя (логин/пароль, токен, OAuth2).
    

**Authorization (авторизация)**

- Проверка прав доступа (roles/permissions/authorities).
    

**Principal**

- Представление пользователя внутри `Authentication` (обычно `UserDetails`).
    

**Authorities / Roles**

- Права пользователя. Например: `ROLE_USER`, `ROLE_ADMIN`.
    
- Разница: роль — это упрощённое обозначение группы прав, authority — конкретное право (`READ_ACCOUNT`).
    

**AccessDecisionManager**

- Компонент, который принимает решение о доступе.
    
- Делегирует голосование к **Voters**.
    

**Voters**

- Голосуют за/против доступа.
    
- Типы:
    
    - `RoleVoter` — проверяет роли.
        
    - `AuthenticatedVoter` — проверяет, аутентифицирован ли пользователь.
        
    - `WebExpressionVoter` — проверяет SpEL-выражения (`hasRole('ADMIN')`).
        

#### 9.2. URL vs Методная авторизация

#### 1. URL-уровень (HTTP запросы)

Пример конфигурации через DSL:

```
http    .authorizeHttpRequests(auth -> auth        .requestMatchers("/public/**").permitAll()        .requestMatchers("/admin/**").hasRole("ADMIN")        .anyRequest().authenticated()    );
```

Как это работает:

1. `FilterSecurityInterceptor` получает запрос и `Authentication`.
    
2. Определяет, какой URL затрагивает правило.
    
3. AccessDecisionManager вызывает Voters → решение (grant/deny).
    
4. Если deny → 403 Forbidden.
    

#### 2. Методная безопасность

Spring позволяет ставить аннотации прямо на сервисы:

```
@PreAuthorize("hasRole('ADMIN')")public void deleteUser(Long id) { ... }
```

- Под капотом: AOP-прокси вызывает AccessDecisionManager.
    
- Можно использовать сложные выражения:
    

```
@PreAuthorize("#userId == principal.id or hasRole('ADMIN')")
```

- `principal` — текущий пользователь.
    
- `#userId` — параметр метода.
    

#### 9.3. Expression Language (SpEL)

`SpEL` позволяет проверять условия на основе `Authentication`:

- `hasRole('ROLE_ADMIN')` — проверка роли.
    
- `hasAuthority('WRITE_PRIVILEGE')` — проверка права.
    
- `principal.username == 'bob'` — доступ для конкретного пользователя.
    
- `#param == authentication.name` — сравнение с текущим именем.
    

#### 9.4. Внутренний процесс принятия решений

1. Запрос дошёл до `FilterSecurityInterceptor` (web) или метод вызван (service).
    
2. Получаем `Authentication` из `SecurityContext`.
    
3. Определяем необходимые права (например, URL pattern или аннотация).
    
4. AccessDecisionManager вызывает все Voters.
    
5. Каждое голосование:
    
    - `ACCESS_GRANTED` → плюс.
        
    - `ACCESS_DENIED` → минус.
        
    - `ABSTAIN` → игнор.
        
6. Итоговое решение зависит от стратегии:
    
    - **AffirmativeBased** — хотя бы один grant → доступ разрешён.
        
    - **UnanimousBased** — все должны дать grant → доступ разрешён.
        
    - **ConsensusBased** — большинство vote → grant.
        

#### 9.5. Роли и authorities

- **Роль (**`**ROLE_...**`**)** — это грубая категоризация.
    
- **Authority** — конкретное право (`READ_ACCOUNT`).
    
- В Spring Security `RoleVoter` ищет префикс `ROLE_`.
    
- Можно назначать сразу несколько ролей/authorities пользователю.
    

Пример в `UserDetails`:

```
@Overridepublic Collection<? extends GrantedAuthority> getAuthorities() {    return List.of(new SimpleGrantedAuthority("ROLE_USER"),                   new SimpleGrantedAuthority("READ_ACCOUNT"));}
```

#### 9.6. Аудит и логирование доступа

- Spring Security умеет логировать: кто попытался, с какого IP, какой результат.
    
- Используется для мониторинга и выявления подозрительной активности.
    

Пример:

```
@Beanpublic AuditorAware<String> auditorProvider() {    return () -> Optional.ofNullable(SecurityContextHolder.getContext().getAuthentication().getName());}
```

- Можно подключить к `@CreatedBy` и `@LastModifiedBy` в JPA.
    

#### 9.7. Best Practices

- Не используйте только URL-паттерны — всегда комбинируйте с методной безопасностью.
    
- Для REST API: проверяйте authorities, а не только роли.
    
- Для сложных политик: используйте SpEL expressions.
    
- Минимизируйте публичный доступ — `permitAll()` только там, где реально безопасно.
    
- Логируйте попытки доступа для аудита.
    
- В реактивных приложениях используйте `ReactiveAuthorizationManager`.
    

#### 9.8. Практический пример

```
@RestController@RequestMapping("/accounts")public class AccountController {    @GetMapping("/{id}")    @PreAuthorize("#id == principal.id or hasRole('ADMIN')")    public Account getAccount(@PathVariable Long id) {        return accountService.getAccount(id);    }    @PostMapping("/")    @PreAuthorize("hasAuthority('WRITE_ACCOUNT')")    public Account createAccount(@RequestBody Account account) {        return accountService.createAccount(account);    }}
```

- На GET доступ имеют либо администраторы, либо владелец.
    
- На POST — только те, у кого есть право `WRITE_ACCOUNT`.
    

- URL и методная авторизация — два уровня защиты.
    
- AccessDecisionManager + Voters → решают, можно ли.
    
- SpEL даёт гибкость для сложных правил.
    
- Роли и authorities — разные понятия, но совместно работают для fine-grained access.
    

### 10. OAuth2, JWT и безопасные токены (Tokens)

Spring Security поддерживает **OAuth2** и **JWT** с полным набором фильтров и компонентов. Это позволяет защищать API и микросервисы даже в stateless-сценариях (без серверных сессий).

#### 10.1. Основные понятия

**OAuth2 (Open Authorization 2.0)**  
Стандарт делегированной аутентификации. Позволяет сторонним клиентам получать доступ к ресурсам пользователя без передачи его пароля.  
Основные роли:

1. **Resource Owner** — владелец ресурса (пользователь).
    
2. **Client** — приложение, запрашивающее доступ.
    
3. **Authorization Server** — сервер авторизации, выдающий токены.
    
4. **Resource Server** — защищённый API, проверяющий токены.
    

**JWT (JSON Web Token)**  
Стандартизированный формат токена: `header.payload.signature`.

- Подпись (HMAC, RS256, ES256) защищает от подделки.
    
- Payload содержит claims: `sub` (subject/пользователь), `exp` (срок действия), `roles`, `iss` (issuer), `aud` (audience).
    
- Stateless: сервер не хранит сессий, достаточно валидного токена.
    

**Access Token**  
Короткоживущий токен, используемый для доступа к ресурсам.

**Refresh Token**  
Долгоживущий токен, позволяющий получить новый access token без повторной аутентификации.

**Bearer Token**  
Передаётся в заголовке:

```
Authorization: Bearer <token>
```

#### 10.2. Почему токены безопасны

Даже при использовании HTTPS токены дают дополнительные гарантии:

1. **Подпись** — изменения payload приводят к недействительной подписи.
    
2. **Срок жизни (**`**exp**`**)** — короткие токены ограничивают время атаки.
    
3. **Audience / Issuer (**`**aud**` **/** `**iss**`**)** — токен работает только в целевых сервисах.
    
4. **Refresh flow** — refresh token хранится безопасно (например, в HttpOnly cookie).
    
5. **Stateless** — сервер не хранит токены, снижается риск компрометации сессий.
    

Пример JWT:

```
{  "header": {"alg": "RS256", "typ": "JWT"},  "payload": {"sub": "user123", "roles": ["ROLE_USER"], "exp": 1700000000},  "signature": "abcdef123456..."}
```

#### 10.3. Spring Security Resource Server

Простейшая настройка JWT Resource Server:

```
@BeanSecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {    http        .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())        .oauth2ResourceServer(oauth2 -> oauth2.jwt());    return http.build();}
```

- `BearerTokenAuthenticationFilter` проверяет токен в заголовке.
    
- Создаётся объект `Authentication` без сессии (stateless).
    

#### 10.4. OAuth2 Authorization Server

Если система сама выполняет функции сервера авторизации, используется проект **Spring Authorization Server**.

Поддерживаются flows:

1. Authorization Code (с PKCE для SPA).
    
2. Client Credentials (machine-to-machine).
    
3. Refresh Tokens.
    

Best practices:

- хранение client secret в зашифрованном виде,
    
- короткоживущие access tokens + refresh tokens в HttpOnly cookie,
    
- настройка JWKs endpoint для проверки подписей.
    

#### 10.5. Stateful vs Stateless

**Stateful (сессии + cookies):**

- `SecurityContext` хранится в `HttpSession`.
    
- Logout — удаление сессии.
    
- CSRF-защита необходима.
    

**Stateless (JWT / Bearer):**

- Нет сессий → масштабируемо для микросервисов.
    
- Logout реализуется удалением токена на клиенте.
    
- CSRF не требуется, если токен в заголовке.
    

#### 10.6. Refresh Tokens и защита

- **Access token**: короткий срок, передаётся в заголовке.
    
- **Refresh token**: долгий срок, хранится в HttpOnly cookie.
    
- **Flow**: клиент отправляет refresh token → получает новый access token.
    

Пример endpoint для обновления:

```
@PostMapping("/token/refresh")public JwtResponse refresh(@CookieValue("refreshToken") String token) {    Authentication auth = authService.verifyRefreshToken(token);    return authService.generateAccessToken(auth);}
```

#### 10.7. Проверка токена

Spring Security (с библиотекой Nimbus) проверяет:

- подпись,
    
- срок действия (`exp`),
    
- издателя (`iss`),
    
- аудиторию (`aud`).
    

#### 10.8. Типичные ошибки и защита

|Ошибка|Последствие|Решение|
|---|---|---|
|JWT без подписи|Возможна подделка|Использовать RS256/ES256|
|Долгоживущий access token|Долгий доступ при краже|Короткий TTL + refresh flow|
|Refresh token доступен в JS|XSS-уязвимость|Хранение в HttpOnly cookie|
|Access token в localStorage|XSS → похищение|Хранение в памяти (in-memory)|
|Нет проверки audience|Токен чужого сервиса|Проверка `aud` в фильтре|

#### 10.9. Практический пример JWT

1. Пользователь логинится → сервер выдаёт access и refresh токены.
    
2. Клиент хранит access token в памяти, refresh — в HttpOnly cookie.
    
3. Каждый запрос отправляется с `Authorization: Bearer <access_token>`.
    
4. Spring Security проверяет подпись и claims → создаётся `Authentication`.
    
5. При истечении access token клиент использует refresh token для обновления.
    

### 11. Регистрация и безопасное хранение паролей (Registration & Password Security)

#### 11.1. Основные принципы

1. Пароли никогда не хранятся в открытом виде.
    
2. Используются адаптивные хеш-функции (BCrypt, Argon2, SCrypt).
    
3. Для каждого пользователя применяется уникальная соль (salt).
    
4. Передача паролей происходит только через HTTPS.
    
5. Желательно применять политику сложности (минимальная длина, обязательные символы, blacklist простых паролей).
    

#### 11.2. Поток регистрации

1. Пользователь вводит имя и пароль.
    
2. Контроллер принимает данные.
    
3. Сервис проверяет уникальность логина/email.
    
4. Пароль хешируется через `PasswordEncoder`.
    
5. Данные сохраняются в БД.
    

#### 11.3. Настройка PasswordEncoder

```
@Beanpublic PasswordEncoder passwordEncoder() {    return new BCryptPasswordEncoder(12); // strength=12}
```

- BCrypt — стандарт по умолчанию.
    
- Argon2 — современный и устойчивый к GPU-атакам.
    
- SCrypt — тоже вариант с высокой защитой от brute-force.
    

#### 11.4. Пример UserService

```
@Service@RequiredArgsConstructorpublic class UserService {    private final UserRepository userRepository;    private final PasswordEncoder passwordEncoder;    public User registerUser(String username, String rawPassword) {        if (userRepository.existsByUsername(username)) {            throw new IllegalArgumentException("Пользователь уже существует");        }        User user = new User();        user.setUsername(username);        user.setPassword(passwordEncoder.encode(rawPassword));        user.setRoles(Set.of("ROLE_USER"));        return userRepository.save(user);    }}
```

#### 11.5. Формы и CSRF

Для stateful приложений (сессии) CSRF-токен обязателен:

```
<form th:action="@{/register}" method="post">
    <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
</form>
```

#### 11.6. Регистрация + Login + JWT

1. Регистрация → сохранение пользователя с хешированным паролем.
    
2. Login → проверка через `PasswordEncoder.matches()`.
    
3. При успехе создаётся `Authentication`.
    
4. Генерация JWT access token:
    

```
String token = jwtService.generateToken(authentication);
```

5. Все запросы к API защищаются через `SecurityFilterChain` или `@PreAuthorize`.
    

#### 11.7. Best practices

- Rate-limiting для защиты от brute-force.
    
- Подтверждение email перед активацией.
    
- Пароли от 12+ символов, буквы, цифры, спецсимволы.
    
- Lockout / throttling при множественных неудачных логинах.
    
- Аудит логов для регистрации и входов.
    

#### 11.8. Реактивный подход (WebFlux)

Используется `ReactiveUserDetailsService` и `ReactiveSecurityContextHolder`.

Пример:

```
Mono<UserDetails> findByUsername(String username) {    return userRepository.findByUsername(username)        .map(user -> User.withUsername(user.getUsername())                         .password(user.getPassword())                         .roles(user.getRoles().toArray(new String[0]))                         .build());}
```

#### 11.9. Полный поток (Registration → Login → JWT)

1. Регистрация: пользователь создаётся с хешированным паролем.
    
2. Login: проверка пароля.
    
3. Успешная аутентификация → генерация JWT.
    
4. API-защита через `BearerTokenAuthenticationFilter`.
    
5. Refresh flow обновляет access token при его истечении.