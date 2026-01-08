119 как создать conection
## 1. Загрузка драйвера

- В современных версиях Java (JDBC 4.0+) драйвер загружается автоматически при наличии в classpath.
    
- Раньше требовалось явно вызвать:
    
    java
    
    ```
    Class.forName("com.mysql.cj.jdbc.Driver");
    ```
    
- **Важно:** драйвер должен быть подключён как зависимость (например, через Maven).
    

## 2. Создание `Connection`

- Используется класс `DriverManager`:
    
    java
    
    ```
    String url = "jdbc:mysql://localhost:3306/mydb";
    String user = "root";
    String password = "secret";
    
    Connection conn = DriverManager.getConnection(url, user, password);
    ```
    
- **URL формируется так:** `jdbc:<vendor>://<host>:<port>/<database>?params`
    

## 3. Создание `Statement` или `PreparedStatement`

- **Statement:** для простых запросов без параметров.
    
    java
    
    ```
    Statement stmt = conn.createStatement();
    ```
    
- **PreparedStatement:** для параметризованных запросов (безопаснее, защищает от SQL‑инъекций).
    
    java
    
    ```
    PreparedStatement ps = conn.prepareStatement(
        "SELECT * FROM users WHERE id = ?");
    ps.setInt(1, 10);
    ```
    

## 4. Выполнение запроса

- Для **SELECT**:
    
    java
    
    ```
    ResultSet rs = ps.executeQuery();
    while (rs.next()) {
        String name = rs.getString("name");
        int age = rs.getInt("age");
        System.out.println(name + " - " + age);
    }
    ```
    
- Для **INSERT/UPDATE/DELETE**:
    
    java
    
    ```
    int rows = ps.executeUpdate();
    System.out.println("Rows affected: " + rows);
    ```
    

## 5. Закрытие ресурсов

- Всегда закрываем `ResultSet`, `Statement`, `Connection` (или используем try‑with‑resources).
    
    java
    
    ```
    try (Connection conn = DriverManager.getConnection(url, user, password);
         PreparedStatement ps = conn.prepareStatement("...")) {
        // работа с БД
    } catch (SQLException e) {
        e.printStackTrace();
    }
    ```
    
- Это гарантирует освобождение ресурсов даже при исключениях.
    

## Итоговая последовательность

1. Подключить драйвер (автоматически или через `Class.forName`).
    
2. Получить `Connection` через `DriverManager.getConnection`.
    
3. Создать `Statement` или `PreparedStatement`.
    
4. Выполнить запрос (`executeQuery` или `executeUpdate`).
    
5. Обработать результат (`ResultSet`).
    
6. Закрыть ресурсы (желательно через try‑with‑resources).

120 закрытие 
Закрытие `Connection` — это критически важный шаг при работе с JDBC, потому что соединение с базой данных является дорогим ресурсом. Если его не закрыть, можно получить утечки соединений и исчерпание пула.

## Правильные способы закрытия `Connection`

### 1. Использовать **try-with-resources** (рекомендуемый способ)

- Начиная с Java 7, можно использовать конструкцию `try-with-resources`, которая автоматически закрывает все ресурсы, реализующие интерфейс `AutoCloseable` (а `Connection`, `Statement`, `ResultSet` его реализуют).
    
- Пример:
    
    java
    
    ```
    String url = "jdbc:mysql://localhost:3306/mydb";
    String user = "root";
    String password = "secret";
    
    try (Connection conn = DriverManager.getConnection(url, user, password);
         PreparedStatement ps = conn.prepareStatement("SELECT * FROM users");
         ResultSet rs = ps.executeQuery()) {
    
        while (rs.next()) {
            System.out.println(rs.getString("name"));
        }
    
    } catch (SQLException e) {
        e.printStackTrace();
    }
    ```
    
- Здесь `conn`, `ps`, и `rs` будут закрыты автоматически в обратном порядке выхода из блока.
    

### 2. Закрывать вручную в `finally`

- Если не используется `try-with-resources`, нужно закрывать соединение в блоке `finally`.
    
- Пример:
    
    java
    
    ```
    Connection conn = null;
    try {
        conn = DriverManager.getConnection(url, user, password);
        Statement stmt = conn.createStatement();
        ResultSet rs = stmt.executeQuery("SELECT * FROM users");
        // работа с результатами
    } catch (SQLException e) {
        e.printStackTrace();
    } finally {
        if (conn != null) {
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
    ```
    

### 3. Важно помнить

- **Закрывать нужно все ресурсы:** сначала `ResultSet`, потом `Statement`, и только потом `Connection`.
    
- **Не закрывать соединение слишком рано:** если используется пул соединений (например, HikariCP, C3P0), вызов `conn.close()` не уничтожает соединение, а возвращает его в пул.
    
- **Не хранить Connection как глобальную переменную:** соединение должно жить только в рамках операции/транзакции.
    

✅ На собеседовании можно ответить так: _"Правильный способ — использовать try-with-resources, чтобы_ `Connection`_,_ `Statement` _и_ `ResultSet` _закрывались автоматически. Если это невозможно, закрывать вручную в блоке finally, проверяя на null. В реальных проектах обычно применяется пул соединений, где_ `close()` _возвращает соединение в пул."_