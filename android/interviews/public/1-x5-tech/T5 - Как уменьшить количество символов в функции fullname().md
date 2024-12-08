# Как уменьшить количество символов в функции `fullname()`?

### Условие:

```
data class Person(
    private val firstName: String,
    private val secondName: String
)

fun Person.fullName() = "${this.firstName} ${this.secondName}"

fun main() {
    println(Person("John", "Doe"))
}
```

### Ответ:

```
data class Person(
    val firstName: String,
    val secondName: String
)

fun Person.fullName() = "$firstName $secondName"

fun main() {
    println(Person("John", "Doe"))
}
```

### Дополнительная информация:

https://kotlinlang.org/docs/strings.html#string-templates