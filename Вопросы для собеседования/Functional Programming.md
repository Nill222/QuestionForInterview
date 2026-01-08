## Функциональные интерфейсы: определение, default и static методы, применение

- **Определение:** функциональный интерфейс — интерфейс с ровно одним абстрактным методом (Single Abstract Method, SAM). Аннотация `@FunctionalInterface` необязательна, но фиксирует намерение и позволяет компилятору отловить ошибки (если появится второй абстрактный метод). Допускаются любые количества `default`, `static`, и `private` методов — они не нарушают SAM, потому что не являются абстрактными.
    
- **Default-методы:** позволяют эволюцию API без ломки реализаций — можно добавлять поведение по умолчанию и композировать операции. Например, в `java.util.function.Function` есть `default` методы `compose` и `andThen` для композиции функций. Если класс реализует несколько интерфейсов с одинаковыми default-методами, он обязан разрешить конфликт — переопределить или явно вызвать `X.super.m()`.
    
- **Static-методы:** относятся к самому интерфейсу, а не к экземплярам; используются для утилитарных операций (фабрики, помощники, предикаты по умолчанию, константы), например `Predicate.isEqual(x)`.
    
- **Область применения:**
    
    - Лямбда-выражения и ссылки на методы в Stream API, `Optional`, `CompletableFuture`, коллекциях.
        
    - Событийные/колбэк‑механизмы (обработчики).
        
    - Композиция поведения (функции высшего порядка), валидации, трансформации, конвейеры обработки.
        

## Лямбда-выражения и замыкания: синтаксис и характеристики

- **Синтаксис:**
    
    - Без параметров: `() -> expr`
        
    - Один параметр: `x -> expr`
        
    - Несколько параметров: `(x, y) -> { /* блок */ return result; }`
        
    - Типы параметров выводятся по контексту (target typing) — тип функционального интерфейса определяет сигнатуру.
        
- **Замыкания (capturing):**
    
    - Лямбда может захватывать значения из внешней области видимости.
        
    - Захваченные локальные переменные должны быть effectively final (не изменяться после присвоения). Изменяемые объекты можно использовать, но ссылку на локальную переменную менять нельзя.
        
    - Внутренняя реализация — как синтетический класс без собственного состояния, с полями для захваченных значений.
        
- **Характеристики:**
    
    - Не создают именованных классов; меньше бойлерплейта, лучше читаемость.
        
    - Являются объектами (экземпляры функционального интерфейса), могут передаваться, возвращаться, храниться.
        
    - Сериализация поддерживается только если целевой функциональный интерфейс Serializable и это явно необходимо.
        
    - Производительность: обычно лучше, чем анонимные классы, благодаря оптимизациям компилятора и JVM (invokedynamic); но злоупотребление лямбдами в горячих путях требует профилинга.
        

## Function, Supplier, Predicate, Consumer: что это и когда применять

### Обзор и базовые методы

- **Function<T,R>:** преобразование T в R.
    
    - Основной метод:
        
        - `R apply(T t)`
            
    - **Составление:**
        
        - `default <V> Function<V,R> compose(Function<? super V,? extends T> before)`
            
        - `default <V> Function<T,V> andThen(Function<? super R,? extends V> after)`
            
    - **Применение:** маппинги (`map`), трансформации DTO, фабрики значений по ключу (`Map.computeIfAbsent`), нормализация входных данных.
        
- **Supplier<T>:** поставщик значения без аргументов.
    
    - Основной метод:
        
        - `T get()`
            
    - **Применение:** ленивые вычисления, фабрики объектов, отложенная инициализация, генераторы (например, `Stream.generate`).
        
- **Predicate<T>:** логический тест над T.
    
    - Основной метод:
        
        - `boolean test(T t)`
            
    - **Комбинаторы:**
        
        - `default Predicate<T> and(Predicate<? super T> other)`
            
        - `default Predicate<T> or(Predicate<? super T> other)`
            
        - `default Predicate<T> negate()`
            
        - `static <T> Predicate<T> isEqual(Object targetRef)`
            
    - **Применение:** фильтрация (`filter`), валидация, построение сложных условий, правила доступа.
        
- **Consumer<T>:** побочное действие над T (без результата).
    
    - Основной метод:
        
        - `void accept(T t)`
            
    - **Композиция:**
        
        - `default Consumer<T> andThen(Consumer<? super T> after)`
            
    - **Применение:** логирование, отправка сообщений/событий, мутации состояния или структур данных, операции `forEach`.
        

### Примеры кода

java

```
// Function: нормализация строки и подсчёт длины
Function<String, Integer> len = String::length;
Function<String, String> norm = s -> s.trim().toLowerCase();
Function<String, Integer> pipeline = norm.andThen(len);
int n = pipeline.apply("  Hello  "); // 5

// Supplier: отложенная ресурсозагрузка
Supplier<Connection> connSupplier = () -> DriverManager.getConnection(url, u, p);

// Predicate: сложная фильтрация
Predicate<User> active = User::isActive;
Predicate<User> admin = u -> u.getRoles().contains(Role.ADMIN);
Predicate<User> visible = active.and(admin.negate());

// Consumer: побочное действие
Consumer<Order> audit = o -> logger.info("Processed {}", o.id());
Consumer<Order> notifyUser = o -> mailer.send(o.userEmail(), "Order shipped");
Consumer<Order> afterProcess = audit.andThen(notifyUser);
```

## Ссылка на метод: что это, и это не «ссылка на объект»

- **Что такое:** синтаксический сахар для лямбда, которая только вызывает существующий метод, с оператором `::`. Совпадение сигнатур делается по контексту (целевой функциональный интерфейс).
    
- **Виды:**
    
    - Статический метод: `ClassName::staticMethod`
        
    - Метод экземпляра конкретного объекта: `instance::method`
        
    - Метод экземпляра произвольного объекта типа: `TypeName::instanceMethod`
        
    - Конструктор: `ClassName::new` (включая массивы `Type[]::new`)
        
- **Не путать:** это не «ссылка на объект» — это способ создать значение функционального интерфейса, указывающее на вызываемый метод. Для instance‑метода конкретного объекта захватывается этот объект; для «произвольного объекта типа» первый аргумент — сам экземпляр (неявно), а параметры — остальные.
    
- **Примеры:**
    

java

```
List<String> items = List.of("a", "bbb", "cc");

// Статический
items.stream().map(Integer::parseInt); // если строки — числа

// Экземпляр конкретного объекта
var cmp = Collator.getInstance(Locale.US);
names.sort(cmp::compare);

// Экземпляр произвольного типа
items.stream().map(String::length).forEach(System.out::println);

// Конструктор
Stream.generate(() -> "x").limit(3).map(StringBuilder::new);
```

## Собственные функциональные интерфейсы: когда и как

- **Когда создавать:**
    
    - Нужна сигнатура, которой нет в `java.util.function` (например, специфичные примитивные комбинации).
        
    - Требуются доменные имена методов для читаемости (например, `Validator<T>.validate(T)`), даже если есть эквиваленты.
        
    - Нужно добавить контракт/документацию или маркерные интерфейсы с default‑методами.
        
- **Правила:**
    
    - Ровно один абстрактный метод (SAM).
        
    - Добавляйте `@FunctionalInterface` для защиты от ошибок.
        
    - При необходимости — default‑методы для композиции/утилит, static‑методы для фабрик.
        
- **Примеры:**
    

java

```
@FunctionalInterface
public interface Validator<T> {
    ValidationResult validate(T value);

    default Validator<T> and(Validator<T> other) {
        return v -> {
            var r1 = this.validate(v);
            return r1.isOk() ? other.validate(v) : r1;
        };
    }

    static <T> Validator<T> notNull() {
        return v -> v != null ? ValidationResult.ok() : ValidationResult.error("null");
    }
}

@FunctionalInterface
public interface ThrowingFunction<T, R, E extends Exception> {
    R apply(T t) throws E;

    static <T, R> Function<T, R> unchecked(ThrowingFunction<T, R, ?> f) {
        return t -> {
            try { return f.apply(t); }
            catch (Exception e) { throw new RuntimeException(e); }
        };
    }
}

// Специализация для примитива, которого нет в JDK:
@FunctionalInterface
public interface ShortToByteFunction {
    byte applyAsByte(short s);
}
```

- **Использование в коде:**
    

java

```
Function<Path, String> read = ThrowingFunction.unchecked(Files::readString);
Validator<User> v = Validator.<User>notNull()
    .and(u -> u.email() != null ? ValidationResult.ok() : ValidationResult.error("email missing"));
```

## Практические ответы для собеседования (краткие формулировки)

- **Определение функционального интерфейса:** интерфейс с одним абстрактным методом (SAM). `@FunctionalInterface` фиксирует контракт. Default/static/private методы допустимы и не нарушают SAM.
    
- **Default и static в интерфейсах:** default — эволюция API и композиция поведения; static — утилиты и фабрики на уровне интерфейса.
    
- **Лямбда и замыкания:** лямбда — компактная реализация SAM; захватывает внешние значения, которые должны быть effectively final; типы параметров выводятся контекстом.
    
- **Function/Supplier/Predicate/Consumer:** преобразование; поставка; логический тест с комбинаторами; побочное действие с композициями.
    
- **Ссылка на метод:** это форма лямбды, которая вызовет уже существующий метод; бывает 4 вида; это не «ссылка на объект», а «референс» для генерации значения функционального интерфейса.
    
- **Свои интерфейсы:** создаём при специфических сигнатурах, доменной читаемости и нужде в дефолтной композиции; защищаем `@FunctionalInterface`.