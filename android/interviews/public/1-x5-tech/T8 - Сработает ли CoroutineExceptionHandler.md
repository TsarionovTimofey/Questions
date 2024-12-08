# Сработает ли CoroutineExceptionHandler?

### Условие:

```
fun main() {
    val coroutineExceptionHandler = CoroutineExceptionHandler { coroutineContext, exception ->
        println("Some text")
    }

    val topLevelScope = CoroutineScope(Job())
    
    topLevelScope.launch {
        launch(coroutineExceptionHandler) {
            throw RuntimeException("Some exception message text")
        }
    }
}
```

### Ответ:

Нет, выведется сообщение из глобального обработчика, процесс завершится с ошибкой

1. `Job` вложенного `launch` передаст ошибку рутовому `launch` и спросит может ли он ее обработать,
   он ответит да
2. `Job` рутового `launch` передаст ошибку `Job `CoroutineScope` и спросит может ли он ее
   обработать. Он ответит нет (Особенность у имплементации Job у CoroutineScope)
3. Рутовый Job посмотрит в своем контексте есть ли у него CoroutineExceptionHandler, не найдет его,
   ошибка уйдет в глобальный обработчик

### Дополнительная информация:

https://kotlinlang.org/docs/exception-handling.html

Шпаргалка по исключениям в корутинах