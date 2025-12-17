## 1. JOIN — виды соединений

- **INNER JOIN**: возвращает только те строки, где совпадают значения в обеих таблицах.  
    [w3schools.com](https://www.w3schools.com/sql/sql_join.asp?utm_source=chatgpt.com)[Википедия](https://ru.wikipedia.org/wiki/Join_%28SQL%29?utm_source=chatgpt.com)
    
- **LEFT (OUTER) JOIN**: возвращает все строки из левой таблицы и соответствующие из правой; если соответствий нет — заполняет `NULL`.  
    [w3schools.com](https://www.w3schools.com/sql/sql_join.asp?utm_source=chatgpt.com)[Википедия](https://ru.wikipedia.org/wiki/Join_%28SQL%29?utm_source=chatgpt.com)
    
- **RIGHT (OUTER) JOIN**: зеркальный аналог LEFT JOIN — показывает все строки из правой таблицы.  
    [w3schools.com](https://www.w3schools.com/sql/sql_join.asp?utm_source=chatgpt.com)[Википедия](https://ru.wikipedia.org/wiki/Join_%28SQL%29?utm_source=chatgpt.com)
    
- **FULL (OUTER) JOIN**: объединяет LEFT и RIGHT JOIN — показывает все строки из обеих таблиц, где нет соответствий — `NULL`.  
    [w3schools.com](https://www.w3schools.com/sql/sql_join.asp?utm_source=chatgpt.com)[Википедия](https://ru.wikipedia.org/wiki/Join_%28SQL%29?utm_source=chatgpt.com)
    
- **CROSS JOIN**: результат — декартово произведение (каждая строка одной таблицы соединяется с каждой строкой другой).  
    [Википедия](https://ru.wikipedia.org/wiki/Join_%28SQL%29?utm_source=chatgpt.com)
    

**Резюме по JOIN типам:**

|Тип JOIN|Что возвращает|
|---|---|
|`INNER JOIN`|Только совпадающие строки в обеих таблицах|
|`LEFT OUTER JOIN`|Все строки левой таблицы + совпадения из правой (иначе NULL)|
|`RIGHT OUTER JOIN`|Все строки правой + совпадения из левой (иначе NULL)|
|`FULL OUTER JOIN`|Все строки из обеих таблиц (несовпадения выводятся с NULL)|
|`CROSS JOIN`|Декартово произведение всех комбинаций строк|

> В SQL-религиозно важно: `LEFT JOIN = LEFT OUTER JOIN` и т.д. Ключевое слово `OUTER` — опционально.  
> Аналогично, `JOIN` по умолчанию считается `INNER JOIN`.  
> [Reddit](https://www.reddit.com/r/learnSQL/comments/104i1kd/how_to_use_left_right_inner_outer_full_and_self/?utm_source=chatgpt.com)[Википедия](https://ru.wikipedia.org/wiki/Join_%28SQL%29?utm_source=chatgpt.com)

---

## 2. GROUP BY, HAVING, ORDER BY

- **`GROUP BY`** — группирует строки по указанным столбцам для использования агрегатных функций (`COUNT`, `SUM`, `AVG` и т.п.).
    
- **`HAVING`** — фильтрует группы после агрегации; работает как `WHERE`, но применяется к агрегатам.
    
- **`ORDER BY`** — сортирует результат выборки по указанным столбцам, по умолчанию по возрастанию (`ASC`), или указывает `DESC`.
    

---

## 3. Подзапросы и оконные функции

- **Подзапросы** бывают:
    
    - **Скалярные** (возвращают одно значение — используются в SELECT, WHERE, etc.)
        
    - **Сложные** — в `WHERE IN`, `EXISTS`, `FROM` (как виртуальная таблица).
        
- **Оконные функции**:
    
    - `ROW_NUMBER()` — номер строки в окне, начинается с 1.
        
    - `RANK()` — ранжирует, но при равных значениях присваивает одинаковый ранг и оставляет "пропуск" в нумерации.
        

---

## 4. Работа с NULL

- **`NULL`** — означает **отсутствие значения**, и **никакое сравнение с NULL не возвращает true**:
    
    `NULL = NULL     -- false NULL <> NULL    -- false`
    
- Используйте:
    
    - `IS NULL` и `IS NOT NULL` для проверки `NULL`
        
    - `COALESCE(a, b, ...)` — возвращает первое ненулевое значение.
        
    - `NULLIF(a, b)` — возвращает `NULL`, если `a = b`, иначе `a`.
    -Всё о джойнах:
    https://www.youtube.com/watch?v=aY7z4HcHm5M
## 1. GROUP BY — HAVING — ORDER BY

### **GROUP BY**

Используется для группировки строк по одному или нескольким столбцам и позволяет применять агрегатные функции (например, `COUNT()`, `SUM()`, `AVG()`).

`SELECT Country, COUNT(*) AS cnt FROM Customers GROUP BY Country;`

Также можно сортировать результат:

`GROUP BY Country ORDER BY cnt DESC;`

[w3schools.com](https://www.w3schools.com/sql/sql_groupby.asp?utm_source=chatgpt.com)[Википедия](https://en.wikipedia.org/wiki/Select_%28SQL%29?utm_source=chatgpt.com)

### **HAVING**

Фильтрует группы после агрегации. В отличие от `WHERE`, позволяет использовать агрегатные функции:

`SELECT Country, COUNT(*) AS cnt FROM Customers GROUP BY Country HAVING COUNT(*) > 5 ORDER BY cnt DESC;`

[w3schools.com](https://www.w3schools.com/sql/sql_having.asp?utm_source=chatgpt.com)[Википедия](https://en.wikipedia.org/wiki/Select_%28SQL%29?utm_source=chatgpt.com)

### **ORDER BY**

Сортирует результаты выборки. Можно использовать `ASC` (по умолчанию) и `DESC`:

`SELECT Country, COUNT(*) AS cnt FROM Customers GROUP BY Country HAVING COUNT(*) > 5 ORDER BY cnt DESC;`

[Википедия](https://en.wikipedia.org/wiki/Select_%28SQL%29?utm_source=chatgpt.com)

---

## 2. Подзапросы (Subqueries)

Подзапрос — это вложенный запрос внутри другого запроса (в `SELECT`, `WHERE`, `FROM`, `HAVING`). Он помогает строить более сложные условия и структуры, используя результат одного запроса в другом.

**Примеры:**

- **Скалярный подзапрос** (возвращает одно значение):
    
    `SELECT name,        (SELECT COUNT(*) FROM orders WHERE orders.user_id = users.id) AS orders_count FROM users;`
    
- **IN**, **EXISTS**:
    
    `SELECT * FROM users WHERE id IN (SELECT user_id FROM orders);`
    
    `SELECT * FROM users u WHERE EXISTS (   SELECT 1 FROM orders o WHERE o.user_id = u.id );`
    

---

## 3. Оконные функции: `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`

Оконные функции выполняют вычисления по определённому "окну" строк, при этом каждая строка остаётся в результирующем наборе.

### **Основные функции**

- `ROW_NUMBER()` — задаёт уникальный номер каждой строке в рамках окна.
    
- `RANK()` — даёт одинаковый ранг одинаковым значениям, но при этом оставляет пропуски после них.
    
- `DENSE_RANK()` — также выдаёт одинаковые ранги, но без пропусков.
    

**Пример:**

`SELECT employee_id, salary,        ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS row_num,        RANK()       OVER (PARTITION BY department ORDER BY salary DESC) AS rank_num,        DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dense_rank FROM employees;`

Использование агрегатов внутри окон:

`SELECT depname, empno, salary,        AVG(salary) OVER (PARTITION BY depname) AS avg_salary FROM empsalary;`

[GeeksforGeeks](https://www.geeksforgeeks.org/sql/window-functions-in-sql/?utm_source=chatgpt.com)[Википедия](https://en.wikipedia.org/wiki/Window_function_%28SQL%29?utm_source=chatgpt.com)

Видео-объяснение:  
[YouTube+1](https://www.youtube.com/watch?v=rIcB4zMYMas&utm_source=chatgpt.com)[YouTube](https://m.youtube.com/watch?v=cXhv4kmIzFw&utm_source=chatgpt.com)

---

## 4. Работа с NULL

### Почему NULL особенный?

- `NULL` означает "отсутствие значения".
    
- Любое сравнение с `NULL`, например `column = NULL` — даёт `FALSE`.
    

### Как правильно проверять на NULL:

`SELECT * FROM table WHERE column IS NULL;  SELECT * FROM table WHERE column IS NOT NULL;`

### Удобные функции для работы с NULL

- `COALESCE(a, b, c)` — возвращает первое ненулевое значение из списка.
    
- `NULLIF(a, b)` — возвращает `NULL`, если `a = b`, иначе — `a`.
    

---

## Резюме и рекомендации

- **GROUP BY** — агрегация
    
- **HAVING** — фильтрация после агрегации
    
- **ORDER BY** — сортировка результата
    
- **Подзапросы** — логика внутри логики (скалярные, IN, EXISTS, FROM)
    
- **Оконные функции** — аналитика без потери строк
    
- **NULL** — специальное значение, требует особого обращения
https://www.youtube.com/watch?v=rIcB4zMYMas
https://www.youtube.com/watch?v=PTOIpOu3D68


![[Основы SQL  1.pdf]]