Уверен, каждый из читателей, знает что такое шаблон проектирования “Singleton”, но не каждый знает как его программировать эффективно и правильно. Данная статья является попыткой агрегирования существующих знаний по этому вопросу.  
  
Кроме того, можно рассматривать статью как продолжение [замечательного исследования](http://habrahabr.ru/blogs/complete_code/27108), публиковавшегося на Хабрахабре ранее.  
  

##### Неленивый Singleton в Java

  
Автору известно два способа реализации шаблона с нормальной инициализацией.  
  

###### 1 Static field

  

```
public class Singleton {	
public static final Singleton INSTANCE = new Singleton();}
```

  
**+** Простая и прозрачная реализация  
**+** Потокобезопасность  
**-** Не ленивая инициализация  
  

###### 2 Enum Singleton

  
По мнению Joshua Bloch’а это лучший способ реализации шаблона [1].  
  

```
public enum Singleton {	INSTANCE;}
```

  
**+** Остроумно  
**+** Сериализация из коробки  
**+** Потокобезопасность из коробки  
**+** Возможность использования EnumSet, EnumMap и т.д.  
**+** Поддержка switch  
**-** Не ленивая инициализация  
  

##### Ленивый Singleton в Java

  
На момент написания статьи существует как минимум три корректных реализации шаблона Singleton с ленивой инициализацией на Java.  
  

###### 1 Synchronized Accessor

  

```
public class Singleton {	
	private static Singleton instance;		
	public static synchronized Singleton getInstance() {		
		if (instance == null) {			
			instance = new Singleton();		}		
		return instance;	}}
```

  
**+** Ленивая инициализация  
**-** Низкая производительность (критическая секция) в наиболее типичном доступе  
  

###### 2 Double Checked Locking & volatile

  

```
public class Singleton {        
	private static volatile Singleton instance;	        
	public static Singleton getInstance() {		
		Singleton localInstance = instance;		
		if (localInstance == null) {			
			synchronized (Singleton.class) {				
				localInstance = instance;				
				if (localInstance == null) {					
					instance = localInstance = new Singleton();				}			}		}		
				return localInstance;	}}
```

  
**+** Ленивая инициализация  
**+** Высокая производительность  
**-** Поддерживается только с JDK 1.5 [5]  
  

###### 2.1 Почему не работает без volatile?

  
Проблема идиомы Double Checked Lock заключается в модели памяти Java, точнее в порядке создания объектов. Можно условно представить этот порядок следующими этапами [2, 3]:  
  
Пусть мы создаем нового студента: Student s = new Student(), тогда  
  
1) local_ptr = malloc(sizeof(Student)) // выделение памяти под сам объект;  
2) s = local_ptr // инициализация указателя;  
3) Student::ctor(s); // конструирование объекта (инициализация полей);  
  
Таким образом, между вторым и третьим этапом возможна ситуация, при которой другой поток может получить и начать использовать (на основании условия, что указатель не нулевой) не полностью сконструированный объект. На самом деле, эта проблема была частично решена в JDK 1.5 [5], однако авторы JSR-133 [5] рекомендуют использовать voloatile для Double Cheсked Lock. Более того, их отношение к подобным вещам легко прослеживается из коментария к спецификации:  
  

> There exist a number of common but dubious coding idioms, such as the double-checked locking idiom, that are proposed to allow threads to communicate without synchronization. Almost all such idioms are invalid under the existing semantics, and are expected to remain invalid under the proposed semantics.

  
Таким образом, хотя проблема и решена, использовать Double Checked Lock без volatile крайне опасно. В некоторых случаях, зависящих от реализации JVM, операционной среды, планировщика и т.д., такой подход может не работать. Однако, серией опытов сопровождаемых просмотром ассемблерного кода, генерированного JIT’ом автору, такой случай вопросизвести не удалось.  
  
Наконец, Double Checked Lock можно использовать без исключений с immutable объектами (String, Integer, Float, и т.д.).  
  

###### 3 On Demand Holder idiom

  

```
public class Singleton {			
	public static class SingletonHolder {		
		public static final Singleton HOLDER_INSTANCE = new Singleton();	}			public static Singleton getInstance() {		
			return SingletonHolder.HOLDER_INSTANCE;	}}
```

  
  
**+** Ленивая инициализация  
**+** Высокая производительность  
**-** Невозможно использовать для не статических полей класса  
  

##### Performance

  
Для сравнения производительности выше рассмотренных методов, была использована микро-бенчмарка [6], определяющая количество элементарных операций (инкремент поля) в секунду над Singleton объектом, из двух параллельных потоков.  
  
Измерения производились на двухядерной машине Intel Core 2 Duo T7300 2GHz, 2Gb ram и Java HotSpot(TM) Client VM (build 17.0-b17). За единицу скора считается количество инкрементов (а следовательно и захватов объекта) в секунду * 100 000.  
  
(больше — лучше)  

||Client|Server|
|---|---|---|
|Synchronized accessor|42,6|86,3|
|Double Checked Lock & volatile|179,8|202,4|
|On Demand Holder|181,6|202,7|

  
  
Вывод: если правильно подобрать реализацию шаблона можно получить ускорение (speed up) от **2х** до **4х**.  
  

##### Summary

  
Можно выделить следующие короткие советы по использованию того или иного подхода для реализации шаблона “Одиночка” [1].  
  
1) Использовать нормальную (не ленивую) инициализацию везде где это возможно;  
2) Для статических полей использовать On Demand Holder idom;  
3) Для простых полей использовать Double Chedked Lock & volatile idom;  
4) Во всех остальных случаях использовать Syncronized accessor;  
  

##### Java Class Library & Singleton

  
Примечательно, что разработчики Java Class Library выбрали наиболее простой способ реализации шаблона — Syncronized Accessor. C одной стороны — это гарантия совместимости и правильной работы. С другой — это потеря процессорного времени на вход и выход из критической секции при каждом обращении.  
  
Быстрый поиск grep’ом по исходникам дал понять, что таких мест в JCL очень много.