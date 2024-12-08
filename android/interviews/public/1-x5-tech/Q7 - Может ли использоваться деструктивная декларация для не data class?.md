# Может ли использоваться деструктивная декларация для не `data class`❓

## Ответ:

Да ✅

```
class TestDestructive() {
    operator fun component1() = "Component 1"
    operator fun component2() = 2
}
```

```
fun main() {
    val (firstProperty, secondProperty) = TestDestructive()
    println("$firstProperty, $secondProperty")
}
```

Результат

```
Component 1, 2
```

### Дополнительная информация:

https://kotlinlang.org/docs/destructuring-declarations.html