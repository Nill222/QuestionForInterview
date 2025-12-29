## 54. Какие бывают потоки ввода/вывода?

В Java I/O потоки делятся на **байтовые** и **символьные**:

- **Байтовые потоки (**`InputStream` **/** `OutputStream`**)**
    
    - Работают с данными в виде байтов.
        
    - Используются для работы с бинарными файлами (изображения, аудио, видео).
        
    - Примеры: `FileInputStream`, `FileOutputStream`, `BufferedInputStream`, `DataOutputStream`.
        
- **Символьные потоки (**`Reader` **/** `Writer`**)**
    
    - Работают с текстовыми данными (char, Unicode).
        
    - Используются для работы с текстовыми файлами.
        
    - Примеры: `FileReader`, `FileWriter`, `BufferedReader`, `PrintWriter`.
        
- **Дополнительно:**
    
    - **Buffered** — добавляют буферизацию для ускорения.
        
    - **Data** — позволяют читать/писать примитивы (`int`, `double`).
        
    - **Object** — для сериализации объектов (`ObjectInputStream`, `ObjectOutputStream`).
        

## 55. Сериализация

- **Что это:** процесс преобразования объекта в поток байтов для сохранения или передачи.
    
- **Зачем нужна:**
    
    - Сохранение состояния объекта в файл/БД.
        
    - Передача объектов по сети (например, RMI, сокеты).
        
    - Кэширование.
        
- **Ключевое слово** `transient`**:**
    
    - Поля, помеченные `transient`, **не сериализуются**.
        
    - Используется для чувствительных данных (пароли) или временных полей, которые не нужно сохранять.
        
- **Сериализация** `static`**-полей:**
    
    - `static` поля **не сериализуются**, так как они принадлежат классу, а не объекту.
        
    - Их значение определяется средой выполнения, а не состоянием конкретного экземпляра.
        

## 56. Возможно ли сохранить объект не в байт-код, а в XML-файл?

Да ✅.

- Стандартная сериализация сохраняет объект в бинарном виде (`ObjectOutputStream`).
    
- Но можно использовать **альтернативные механизмы сериализации**:
    
    - **JAXB (Java Architecture for XML Binding)** — преобразует объект в XML и обратно.
        
    - **XStream** — библиотека для сериализации объектов в XML/JSON.
        
    - **Jackson** — чаще для JSON, но умеет и XML.
        

Пример с JAXB:

java

```
@XmlRootElement
class Person {
    public String name;
    public int age;
}

// Сохранение в XML
Person p = new Person();
p.name = "Егор";
p.age = 25;

JAXBContext context = JAXBContext.newInstance(Person.class);
Marshaller marshaller = context.createMarshaller();
marshaller.setProperty(Marshaller.JAXB_FORMATTED_OUTPUT, true);
marshaller.marshal(p, new File("person.xml"));
```