# Какими способами реализуется конкатенация строк в kotlin❓

## Ответ:

* Оператор `+`, метод `plus`

```
val s = "abc" + 1

val a = "Hello"
val b = "Baeldung"
val c = a.plus(" ").plus(b)
```

* Многострочные строки

```
val text = """
    for (c in "foo")
        print(c)
    """
```

* Шаблоны строк

```
val i = 10
println("i = $i")
// i = 10

val letters = listOf("a","b","c","d","e")
println("Letters: $letters")
// Letters: [a, b, c, d, e]
```

* StringBuilder, StringBuffer (Тоже самое но синхронизованный)

```
val builder = StringBuilder()
    builder.append("Hello")
           .append(" ")
           .append("Baeldung")
           
```

### Дополнительная информация:

https://kotlinlang.org/docs/strings.html
https://kotlinlang.org/docs/java-to-kotlin-idioms-strings.html
https://www.baeldung.com/kotlin/concatenate-strings
https://javarush.com/groups/posts/2351-znakomstvo-so-string-stringbuffer-i-stringbuilder-v-java