# JVM. Организация памяти, сборщики мусора, VisualVM

> Задание:  просмотрите код ниже и опишите (текстово или с картинками) каждую
> строку с точки зрения происходящего в JVM
>
> Не забудьте упомянуть про:
>
>  - ClassLoader’ы
>  - области памяти (стэк (и его фреймы), heap)
>  - сборщик мусора



    public static void main(String[] args) {
        int i = 1;                      // 1
        Object o = new Object();        // 2
        Integer ii = 2;                 // 3
        printAll(o, i, ii);             // 4
        System.out.println("finished"); // 7
    }
    
    private static void printAll(Object o, int i, Integer ii) {
        Integer uselessVar = 700;                   // 5
        System.out.println(o.toString() + i + ii);  // 6

### Работа ClassLoader
При созаднии нашего класса Java объявляет поиск нужных классов для того чтобы их подгрузить в MetaSpace.
Для этого программе помогают три уровня наследования ClassLoader.
1. запрос поступает в Application ClassLoader, ,
2. Application ClassLoader перенаправляет запрос на поиск класса в Platform ClassLoader.
3. Platform ClassLoader в свою очередь отправляет запрос в Bootstrap ClassLoader.
4. Bootstrap ClassLoader обрабатывает запрос сам, так как не имеет других классов для диллигирования. Область поиска и у данного класслоудера минимальная из всех трех.
    1. В случае нахождения нужного класса он по обратной цепочке направляет подгруженные данные.
       Bootstrap ClassLoader -> Platform ClassLoader -> Application ClassLoader.
    2. В том случае если не находится нужный класс, запрос на его поиск возвращается Platform ClassLoader. Он обладает более широким кругом поиска чем Platform ClassLoader.
        1. В случае нахождения данные пересылаются согласно цепочки: Platform ClassLoader -> Application ClassLoader.
        2. В случае если текущий класслоудер не находит, запрос возвращается Application ClassLoader.
5. Application ClassLoader обладает наиболее широким спектором для поиска. При неудачах предыдущих класслоудеров, запрос обрабатывает он.
    1. Находит запрашиваемый класс - пересылает в Metaspace.
    2. Не находит - выбрасывает ошибку ClassNotFoundException.

### Описание работы кода (программы)
0. При запуске класса Main, создается фрейм в стек памяти
   ```public static void main(String[] args) {```
1. переменной типа int присваивается значение 1. Хранится в Stack memory
   ``` int i = 1; ```
2. Создается объект о с участием Metaspace. Сам объект хранится в heap, но ссылка на него сохраняется в Stack Memory
   ```Object o = new Object();```
3. Создание и присвоение переменной ii ссылочного типа значения 2. Значение хранится в куче, ссылка - в стэк мемори.
   ```Integer ii = 2;```
4. вызов метода printAll. Создание нового фрейма в стаке, ссылка на входные параметры ii, o из кучи.
   ```printAll(o, i, ii);```
5. Создание и присвоение переменной ссылочного типа  значения 700. Ссылка хранится в Stack memory, значение - в head.
   ```Integer uselessVar = 700;```
6. Вызов системного метода println(). Создание фрейма в стек памяти, ссылки на значение наших входящих параметов из кучи.
   ```Integer uselessVar = 700;   ```
7. Вызов метода println() сохраняется в Stack memory.
   ```System.out.println("finished");```

### Сборщик мусора

1. На первом этапе удалит удалит ссылку на объект `Integer` с наименованием `uselessVar`  и числовым значением `700`т.к. она мертвая и связь цепи GC ROOTS отсутствуют.
2. Ссылка на объект`"о"` , которая находиться к куче с числовым значением `"1"` в стеке frame (кадра) Main и ссылка  на объект `Integer` с наименованием `"ii"` и числовым значением `"2"` будут последующими т.к. вызываются в стеке frame (кадра) `Main`.

