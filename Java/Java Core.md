## Java Core — Шпаргалка

### 1. `final`, `static`, `this`, `super`

- **`final`**:
    
    - Переменная, объявленная `final`, может быть присвоена только один раз; если это ссылка — нельзя переназначить объект, но можно менять состояние объекта [Википедия](https://en.wikipedia.org/wiki/Final_%28Java%29?utm_source=chatgpt.com).
        
    - `final`-класс нельзя наследовать, а `final`-метод — переопределять [Википедия](https://en.wikipedia.org/wiki/Final_%28Java%29?utm_source=chatgpt.com).
        
- **`static`**:
    
    - Статические поля и методы принадлежат самому классу, а не экземпляру. Обычно используются для хранения констант (`static final`) или утилитарных методов.
        
- **`this`**:
    
    - Ссылка на текущий объект. Используется для явного доступа к полям или методам текущего класса, особенно когда имя параметра совпадает с полем.
        
- **`super`**:
    
    - Ссылка на объект родительского класса. Позволяет вызвать его конструктор (`super(...)`) или переопределённый метод (`super.method()`).
        

---

### 2. Память: Stack vs Heap

- **Stack (стек)**:
    
    - Хранит примитивные типы и ссылки на объекты, создаваемые в методах. Память выделяется/освобождается автоматически (LIFO) [Baeldung on Kotlin](https://www.baeldung.com/java-stack-heap?utm_source=chatgpt.com)[GeeksforGeeks](https://www.geeksforgeeks.org/java/java-stack-vs-heap-memory-allocation/?utm_source=chatgpt.com).
        
    - Быстро и потокобезопасно, но ограничено по объёму.
        
- **Heap (куча)**:
    
    - Используется для динамической памяти — объекты, создаваемые через `new`, хранятся здесь [Baeldung on Kotlin](https://www.baeldung.com/java-stack-heap?utm_source=chatgpt.com)[GeeksforGeeks](https://www.geeksforgeeks.org/java/java-stack-vs-heap-memory-allocation/?utm_source=chatgpt.com).
        
    - Управляется сборщиком мусора. Медленнее, но значительно больше и общедоступна потокам.
        

---

### 3. Перегрузка (overload) vs Переопределение (override)

- **Перегрузка (`overload`)**:
    
    - Несколько методов с одним именем, но разными параметрами в одном классе.
        
    - Компиляция выбирает нужный метод по сигнатуре (compile-time polymorphism) [digitalocean.com](https://www.digitalocean.com/community/tutorials/overriding-vs-overloading-in-java?utm_source=chatgpt.com)[GeeksforGeeks](https://www.geeksforgeeks.org/java/difference-between-method-overloading-and-method-overriding-in-java/?utm_source=chatgpt.com).
        
- **Переопределение (`override`)**:
    
    - Подкласс предоставляет реализацию метода, который уже объявлен в суперклассе с той же сигнатурой.
        
    - Вызывается версия метода объекта, а не ссылка (runtime polymorphism). Рекомендуется использовать `@Override` [Software Engineering Stack Exchange](https://softwareengineering.stackexchange.com/questions/164353/whats-the-difference-between-overloading-a-method-and-overriding-it-in-java?utm_source=chatgpt.com)[digitalocean.com](https://www.digitalocean.com/community/tutorials/overriding-vs-overloading-in-java?utm_source=chatgpt.com).
        

---

### 4. Примитивы vs Ссылочные типы, `null`

- **Примитивные типы** (`int`, `boolean`, `char` и др.):
    
    - Хранят реальные значения. Не могут быть `null`, имеют дефолтные значения (0, false, '\u0000') [Википедия](https://en.wikipedia.org/wiki/Java_syntax?utm_source=chatgpt.com)[Baeldung on Kotlin](https://www.baeldung.com/java-primitives-vs-objects?utm_source=chatgpt.com).
        
- **Ссылочные типы** (объекты, массивы, `String`):
    
    - Хранят ссылки на объекты в heap. Могут быть `null`, если не инициализированы [w3schools.com](https://www.w3schools.com/java/java_data_types_non-prim.asp?utm_source=chatgpt.com)[docs.oracle.com](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html?utm_source=chatgpt.com).
        
- `null` — специальная литерал, указывающая на отсутствие объекта; применяется только для ссылочных типов [docs.oracle.com](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html?utm_source=chatgpt.com)[w3schools.com](https://www.w3schools.com/java/java_data_types_non-prim.asp?utm_source=chatgpt.com).
    

---

### 5. Быстрые команды и примеры

java

КопироватьРедактировать

`// final final int x = 5; // попытка x = 7 — ошибка компиляции  // static class Utils {   static void helper() { ... } }  // this и super class Parent {   void say() { System.out.println("Parent"); } } class Child extends Parent {   void say() { super.say(); System.out.println("Child"); }   void init(int x) { this.x = x; } }  // Перегрузка void print(int a) { … } void print(String s) { … }  // Переопределение @Override void toString() { return "My"; }  // Примитив vs ссылка int a = 0; String s = null;`

---

### Резюме

|Тема|Суть|
|---|---|
|`final`, `static`, `this`, `super`|Модификаторы и ссылки для управления поведением классов/методов|
|Память: Stack vs Heap|Stack — локальные данные; Heap — объекты, управляемые GC|
|Overload vs Override|Перегрузка — compile-time; переопределение — runtime, через наследование|
|Примитивы vs Ссылки, `null`|Примитивы — всегда значение, ссылки — объекты или `null`|