# Дипломная работа. Поисковая система

Наша цель - разработать класс поискового движка, который способен быстро находить указанное слово среди pdf-файлов, причём ранжировать результаты по количеству вхождений. Также у нас будет сервер, который обслуживает входящие запросы с помощью этого движка. Но обо всём по-порядку.

## Входные данные:
Сделайте Fork этого репозитория в свой гитхаб-аккаунт через кнопку слева вверху:

![image](https://user-images.githubusercontent.com/53707586/152979745-ef37549f-9207-46b6-a160-33d16f3e5360.png)

Склонируйте форкнутый репозиторий и откройте его в идее как мавен-проект.

Помимо стандартных папок и файлов, в папке вашего проекта будет папка `pdfs` - в ней находятся .pdf-файлы, по которым будет искать поисковый движок.

![image](https://user-images.githubusercontent.com/53707586/152980485-675392c1-4d98-4e22-abdc-7a85678f53a7.png)

В проекте есть заготвки кода:

| Класс      | Описание |
| ----------- | ----------- |
| `Main`      | Сейчас в нём находится заготовка использования поискового движка. После его реализации, содержимое `main` нужно будет заменить на запуск сервераЮ обслуживающего поисковые запросы       |
| `PageEntry`   | Класс, описывающий один элемент результата одного поиска. Он состоит из имени пдф-файла, номера страницы и количества раз, которое встретилось это слово на ней        |
| `SearchEngine`   | Интерфейс, описывающий поисковый движок. Всё что должен уметь делать поисковый движок, это на запрос из слова отвечать списком элементов результата ответа        |
| `BooleanSearchEngine`   | Реализация поискового движка, которую вам предстоит написать. Слово `Boolean` пришло из теории информационного поиска, тк наш движок будет искать в тексте ровно то слово, которое было указано, без использования синонимов и прочих приёмов нечётного поиска        |

## PDF
Для работы с пдф мы будем использовать библиотеку `com.itextpdf:itext7-core:7.1.15`, которая уже подключена в ваш `pom.xml`:
```xml
        <dependency>
            <groupId>com.itextpdf</groupId>
            <artifactId>itext7-core</artifactId>
            <version>7.1.15</version>
            <type>pom</type>
        </dependency>
```

Основные инструменты из этой библиотеки будут выглядеть следующим образом:

Чтобы создать объект пдф-документа, нужно указать объект `File` этого документа следующим классам: `var doc = new PdfDocument(new PdfReader(pdf));`.
Чтобы получить объект одной страницы документа, нужно вызвать `doc.getPage(номерСтраницы)`. Полистайте методы `doc` чтобы найти способ узнать количество страниц в документе.
Чтобы получить текст со страницы, используйте `var text = PdfTextExtractor.getTextFromPage(page);`.
Чтобы разбить текст на слова (а они в этих документах разделены могут быть не только пробелами), используйте `var words = text.split("\\P{IsAlphabetic}+");`.

## Индексация и поиск
Здесь речь пойдёт о реализации класса `BooleanSearchEngine`. Мы хотим чтобы поиск работал быстро, поэтому предпочтём сканирование всех пдф-ок в конструкторе класса с сохранением информации для каждого слова, чтобы метод `search` отрабатывал быстро, по сути возвращая уже посчитанный в конструкторе для слова ответ. Т.е. в конструкторе для каждого слова нужно сохранить готовый на возможный будущий запрос ответ `List<PageEntry>` (для этого можно использовать мапу, где ключом будет слово, а значением - поисковый ответ для него). Такое предварительное сканирование файлов по которым мы будем искать называется _индексацией_.

## Выходные данные:
В итоге, вам нужно реализовать логику индексации в конструкторе. Сканируя каждый пдф-файл вы перебираете его страницы, для каждой страницы извлекаете из неё слова и подсчитываете их количество. После чего, для каждого уникального слова создаёте объект `PageEntry` и сохраняете в поле (а что за поле - предлагалось подумать выше). Учтите также, что мы хотим регистронезависимый поиск, т.е. по слову "бизнес" должны учитываться и "бизнес", и "Бизнес" в документах (для этого при обработке достаточно каждое слово переводить в нижний регистр с помощью встроенного метода класса `String` для этих целей).

Также, списки ответов для каждого слова должны быть отсортированы в порядке уменьшения поля `count`. Для этого предлагается классу `PageEntry` сразу реализовывать интерфейс `Comparable`.

После того как вы это реализуете, можете протестировать работу вашего движка в `Main`. Протестируйте на тех словах, на которых хотите. Вот пример работы на слове "бизнес":
```
[PageEntry{pdf=Этапы оценки проекта_ понятия, методы и полезные инструменты.pdf, page=12, count=6}, PageEntry{pdf=Этапы оценки проекта_ понятия, методы и полезные инструменты.pdf, page=4, count=3}, PageEntry{pdf=Этапы оценки проекта_ понятия, методы и полезные инструменты.pdf, page=5, count=3}, PageEntry{pdf=1. DevOps_MLops.pdf, page=5, count=2}, PageEntry{pdf=Что такое блокчейн.pdf, page=1, count=2}, PageEntry{pdf=Что такое блокчейн.pdf, page=3, count=2}, PageEntry{pdf=Этапы оценки проекта_ понятия, методы и полезные инструменты.pdf, page=2, count=1}, PageEntry{pdf=Этапы оценки проекта_ понятия, методы и полезные инструменты.pdf, page=11, count=1}, PageEntry{pdf=1. DevOps_MLops.pdf, page=3, count=1}, PageEntry{pdf=1. DevOps_MLops.pdf, page=4, count=1}, PageEntry{pdf=Что такое блокчейн.pdf, page=2, count=1}, PageEntry{pdf=Что такое блокчейн.pdf, page=4, count=1}, PageEntry{pdf=Что такое блокчейн.pdf, page=5, count=1}, PageEntry{pdf=Что такое блокчейн.pdf, page=7, count=1}, PageEntry{pdf=Что такое блокчейн.pdf, page=9, count=1}, PageEntry{pdf=Продвижение игр.pdf, page=7, count=1}, PageEntry{pdf=Как управлять рисками IT-проекта.pdf, page=2, count=1}]
```

## Сервер
После завершения работы над движком, вам следует написать сервер по примеру того, как вы уже делали в предыдущих заданиях. В `main` должен запускаться сервер, слушающий порт `8989`, к которому будут происходить подключения и на входной поток подавать одно слово (обозначим как `word`), отвечать результатом вызова метода `search(word)`, но в виде JSON-текста (библиотеку для работы с JSON подключите к `pom.xml`).

Пример ответа на запрос `бизнес`:
```json
[
  {
    "pdfName": "Этапы оценки проекта_ понятия, методы и полезные инструменты.pdf",
    "page": 12,
    "count": 6
  },
  {
    "pdfName": "Этапы оценки проекта_ понятия, методы и полезные инструменты.pdf",
    "page": 4,
    "count": 3
  },
  {
    "pdfName": "Этапы оценки проекта_ понятия, методы и полезные инструменты.pdf",
    "page": 5,
    "count": 3
  },
  {
    "pdfName": "1. DevOps_MLops.pdf",
    "page": 5,
    "count": 2
  },
  {
    "pdfName": "Что такое блокчейн.pdf",
    "page": 1,
    "count": 2
  },
  {
    "pdfName": "Что такое блокчейн.pdf",
    "page": 3,
    "count": 2
  },
  {
    "pdfName": "Этапы оценки проекта_ понятия, методы и полезные инструменты.pdf",
    "page": 2,
    "count": 1
  },
  {
    "pdfName": "Этапы оценки проекта_ понятия, методы и полезные инструменты.pdf",
    "page": 11,
    "count": 1
  },
  {
    "pdfName": "1. DevOps_MLops.pdf",
    "page": 3,
    "count": 1
  },
  {
    "pdfName": "1. DevOps_MLops.pdf",
    "page": 4,
    "count": 1
  },
  {
    "pdfName": "Что такое блокчейн.pdf",
    "page": 2,
    "count": 1
  },
  {
    "pdfName": "Что такое блокчейн.pdf",
    "page": 4,
    "count": 1
  },
  {
    "pdfName": "Что такое блокчейн.pdf",
    "page": 5,
    "count": 1
  },
  {
    "pdfName": "Что такое блокчейн.pdf",
    "page": 7,
    "count": 1
  },
  {
    "pdfName": "Что такое блокчейн.pdf",
    "page": 9,
    "count": 1
  },
  {
    "pdfName": "Продвижение игр.pdf",
    "page": 7,
    "count": 1
  },
  {
    "pdfName": "Как управлять рисками IT-проекта.pdf",
    "page": 2,
    "count": 1
  }
]
```

## Реализация:

1. В самом начале создала класс `PageEntry` имплементирующий интерфейс `Comparable<PageEntry>`, чтобы полученные результаты можно было отсортировать по убыванию. У данного класса следующие поля: имя pdf-файла, страница на которой встречается искомое слова, и количество его вхождений на этой странице:

```java
public class PageEntry implements Comparable<PageEntry> {
    private final String pdfName;
    private final int page;
    private final int count;

    public PageEntry(String pdfName, int page, int count) {
        this.pdfName = pdfName;
        this.page = page;
        this.count = count;
    }

    @Override
    public int compareTo(PageEntry o) {

        int result = Integer.compare(this.count, o.count);
        if (result == 0) {
            result = o.pdfName.compareTo(this.pdfName);
        }
        if (result == 0) {
            result = Integer.compare(o.page, this.page);
        }
        return result;
    }

    @Override
    public String toString() {
        return "PageEntry{" +
                "pdfName='" + pdfName + '\'' +
                ", page=" + page +
                ", count=" + count +
                '}';
    }
}
```
2. Далее всю логику работы программы описала в классе `BooleanSearchEngine`, имплементирующий интерфейс `SearchEngine`. В нем переопределила метод `search(String word)`:
```java
@Override
    public List<PageEntry> search(String word)  {
        word = word.toLowerCase();
        return mapResult.get(word);
    }
```
3. Описала метод `unique(String[] words)`. В нем формируется HashMap из слова и количества его вхождений. Если слово встречается впервые, то оно добавляется в HashMap. Если ткое слово уже есть, то его количество увеличивается на 1:
```java
public static HashMap<String, Integer> unique(String[] words) {

        HashSet<String> distinctKey = new HashSet<>();
        HashMap<String, Integer> map = new HashMap<>();

        for (int i = 0; i < words.length; i++) {
            if (distinctKey.add(words[i]))
                map.put(words[i], 1);
            else
                map.put(words[i], (map.get(words[i])) + 1);
        }
        return map;
    }
```
4. Далее в самом теле класса `BooleanSearchEngine` в цикле создается массив pdf-файлов, для каждого файла еще один цикл, который перебирает все страницы. Далее еще один цикл разбивает текст на отдельные слова и каждое слова записывает в HashMap, гле ключ - это само слово, а значение - количество его вхождений на данной странице. И так по всем страницам, всех документов.
```java
public BooleanSearchEngine(File pdfsDir) {

        HashMap<String, PageEntry> mapOnePage = new HashMap<>();

        try {
            File dir = new File(String.valueOf(pdfsDir));
            File[] fileArray = dir.listFiles();

            String fileName;
            int numberOfPage;
            int countOfWords;


            for (int i = 0; i < fileArray.length; i++) {
                var doc = new PdfDocument(new PdfReader(fileArray[i]));
                int numberOfPages = doc.getNumberOfPages();
                fileName = fileArray[i].toString();

                for (int j = 1; j < numberOfPages + 1; j++) {
                    numberOfPage = j;
                    PdfPage onePage = doc.getPage(j);
                    var text = PdfTextExtractor.getTextFromPage(onePage);
                    var words = text.split("\\P{IsAlphabetic}+");

                    HashMap<String, Integer> mapWordsCounts = unique(words);

                    PageEntry pageEntry;
                    List <PageEntry> listValues = new ArrayList<>();

                    List<String> tempListKeys = new ArrayList<>(mapWordsCounts.keySet());
                    for (int k = 0; k < tempListKeys.size(); k++) {
                        String tempWord = tempListKeys.get(k);
                        countOfWords = mapWordsCounts.get(tempWord);
                        pageEntry = new PageEntry(fileName, numberOfPage, countOfWords);
                        mapOnePage.put(tempWord, pageEntry);
                        if (!mapResult.containsKey(tempWord)) {
                            listValues = new ArrayList<>();
                            listValues.add(pageEntry);
                            mapResult.put(tempWord, listValues);
                        } else {
                            mapResult.get(tempWord).add(pageEntry);
                            Collections.sort(mapResult.get(tempWord), Collections.reverseOrder());
                        }
                    }
                }
            }
        }
        catch (IOException e) {
            e.printStackTrace();
        }
    }
```
5. Предпоследний шаг, создала класс `Client`, который устанавливает взаимосвязь с сервером в классе `Main`, и просит пользователя ввести искомое слово, ожидает ответ.
```java
public class Client {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        try (
                Socket socket = new Socket("localhost", 8989);
                BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
        ) {
            System.out.println("Какое слово ищем: ");
            String word = scanner.nextLine();
            out.println(word);
            System.out.println(in.readLine());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
6. Когда запрос получен, он отправляется на сервер и происходит поиск совпадений по уже сформированной HashMap, в шаге 4. Это позволяет значительно ускорить поиск необходимого слова.

```java
public class Main {
    public static void main(String[] args) throws Exception {
        BooleanSearchEngine engine = new BooleanSearchEngine(new File("pdfs"));

        int port = 8989;
        System.out.println("Запуск сервера " + port + "...");

        while (true) {
            try (ServerSocket serverSocket = new ServerSocket(port);
                 Socket clientSocket = serverSocket.accept();
                 PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true);
                 BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()))) {
                final String word = in.readLine();

                GsonBuilder builder = new GsonBuilder();
                Gson gson = builder.create();

                out.println(gson.toJson(engine.search(word)));
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```
## Итог:

![Alt text](https://github.com/alisasuslova/Diplom_1/blob/master/%D1%80%D0%B5%D0%B7.JPG)
