## Неотловленная ошибка в корутине

**Глобальный обработчик** - обработчик не отловленных исключений, туда попадают все неотловленные
исключения

```
supend fun throwExceptionSuspend() { throw Exception() }
fun throwException() { throw Exception() }
val handler = CoroutineExceptionHandler { context, exception -> log("handled $exception") }
```

### Если попробовать обернуть `launch` в `try-catch`, `try-catch` не сможет отловить эту ошибку

```
val scope = CoroutineScope(Job())

val job = try {
    scope.launch {
        throwException()
    }
} catch(e: Exception) { null }

job?.join()

// Крэш
```

`launch` формирует `CoroutineContext`, создает пару `Continuation+Job` и отправляет `Continuation`
диспетчеру, который помещает его в очередь, а само исключение проиcходит в потоке который выполнял
последний аргумент функции `launch`, который является функциональным типом и передавался
как `trailing-lambda`

```
public fun CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> Unit
): Job {
```

Выполнение `block` обернуто в `try-catch` и ошибка будет отловлена, `Continuation` передает
результат работы в `Job (launch)`.

`Job (launch)` проверяет что ему пришло, тк ему пришла ошибка он сообщает
об ошибке родителю `Job (scope)` и спрашивает может ли `Job (scope)` ее обработать, он отвечает
нет (это особенность `Job` который является `scope`) и отменяет себя и всех своих детей

`Job (launch)` пытается обработать исключение, ищет в своем контексте `CoroutineExceptionHandler`,
не находит его и ошибка уходит в глобальный обработчик

### Ошибки в `fun`/`suspend fun` внутри корутин можно отлавливать с помощью `try-catch`

```
val scope = CoroutineScope(Job())

scope.launch {
    try {
        throwExceptionSuspend()
        throwException()
    } catch(e: Exception) { }
    // Выполнение кода продолжается
}

job.join()

// Крэша нет, исключение отловлено в try-catch
```

Ошибка не распостраняется за пределы `try-catch`

### Если бы мы добавили `CoroutineExceptionHandler` в `CoroutineContext` `scope` или `launch`

```
val scope = CoroutineScope(Job() + handler)

scope.launch(CoroutineName("1") + handler) { // или если бы добавили сюда scope.launch(handler)
    throwException()
    // Выполнение кода не продолжается исключение отправлено в handler
}

scope.launch(CoroutineName("2")) { // будет отменена после ошибки в job
    superlongWork()
}

// Крэша нет, исключение в корутине launch ей же и обработано
```

`Job (scope)` отменяет себя и всех своих детей, `Job (launch)` отправляет исключение в `handler`,
`job2` будет отменена после ошибкив в `job1`. Выполнение кода приложения продолжается

Для того чтобы job2 не была отменена мы могли бы передать с scope вместо `Job()`, `SupervisorJob()`
или использовать `SupervisorScope()`

```
val scope = CoroutineScope(SupervisorJob() + handler)
// val scope = SupervisorScope(handler)

scope.launch(CoroutineName("1")) {
    throwException()
    // Выполнение кода не продолжается исключение отправлено в handler
}

scope.launch(CoroutineName("2")) { // не будет отменена после ошибки в job
    superlongWork()
}

// Крэша нет, исключение в корутине launch ей же и обработано, job2 не отменена
```

### В случае большей вложенности

Когда в корутине происходит исключение она передает его родителю и спрашивает может ли он обработать
ошибку

Если родитель скоуп - то он отвечает что не сможет и корутина в которой произошло исключение
ищет у себя в контексте hanlder и отдает ее ему, если не находит ошибка уходит в глобальный
обработчик

А если родитель - корутина, то она отвечает, что сможет. И дочерняя корутина передает ошибку
родителю. Родитель спрашивает у своего родителя, сможет ли он ее обработать и если тот может, то
передает ему ошибку и так вверх по иерархии корутин пока не достигнет скоупа, скоуп ответит что не
сможет и рутовая корутина ищет у себя в контексте hanlder и отдает ее ему, если не находит ошибка
уходит в глобальный обработчик

### Каждый родитель, через который прошла ошибка, будет отменяться сам и отменять все свои дочерние корутины

```
val scope = CoroutineScope(Job() + handler)

scope.launch(CoroutineName("1")) { // Отменила себя и детей попробовала передать ошибку в скоуп, передала ошибку в handler
    launch(CoroutineName("1_1")) { // Исключение
        throwException()
    }
    launch(CoroutineName("1_2")) { // Отменена корутиной 1
        doSomeWork()
    }
}

scope.launch(CoroutineName("2")) { // Отменена cкоупом, т.к. используется Job()
    launch(CoroutineName("2_1")) { // Отменена корутиной 2
         doSomeWork()
    }
    launch(CoroutineName("2_2")) { // Отменена корутиной 2
        doSomeWork()
    }
}

// Выполнение кода продолжается, исключение в корутине 1_1 обработано корутиной 1
```

Корутина 1 обработала исключение

### CoroutineExceptionHandler будет работать только в рутовой корутине

```
val scope = CoroutineScope(Job())

scope.launch(CoroutineName("1")) { // Отменила себя и детей попробовала передать ошибку выше, попробовала найти handler, ошибка ушла в глобальный обработчик
    launch(CoroutineName("1_1") + handler) { // Исключение
        throwException()
    }
    launch(CoroutineName("1_2")) { // Крэш приложения
        doSomeWork()
    }
}

scope.launch(CoroutineName("2")) { // Крэш приложения
    launch(CoroutineName("2_1")) { // Крэш приложения
         doSomeWork()
    }
    launch(CoroutineName("2_2")) { // Крэш приложения
        doSomeWork()
    }
}

// Крэш приложения
```

Корутина не обработала исключение, крэш

### Костыль с передачей `SuperviserJob()` не в scope, юзать не нужно, для ознакомления

При передаче `Job` в `launch` этот `Job` будет использован, как родитель создаваемой корутины.
Т.е. мы подменяем родительский `Job` в контексте дочерней корутины. Это приведет к тому, что
Job дочерней корутины не установит связь с реальным Job-ом родительской корутины.

Т.е. передав `SupervisorJob` в билдер, создающий дочернюю корутину мы вместо связи:

`parentJob > childJob`

получим:

`supervisorJob > childJob`

Это нарушение принципа structured concurrency и связи между корутинами.

Есть костыль, который можно использовать, чтобы сохранить связь:

```
scope.launch { // parent coroutine
   launch(SupervisorJob(coroutineContext[Job])) { // child coroutine
   }
}
```

В `SupervisorJob` передаем `Job` родительской корутины. Это приведет к тому, что родительский `Job`
станет родителем `SupervisorJob`. А сам `SupervisorJob` станет родителем для `Job` дочерней
корутины:

`parentJob > supervisorJob > childJob`

Связь между корутинами сохраняется, но частично. При отмене родителя отменятся и его дочерние
корутины. А вот по завершению работы дочерних, родитель все еще будет считать себя `isActive`.

Если в дочерней корутине произойдет ошибка, то `childJob` передаст ее в `supervisorJob`, а тот
ничего не будет отменять и дальше (в `parenJjob`) эту ошибку передавать не будет. Ошибка будет
направлена в обработчик дочерней корутиной.

### Async

Исключения так же будут передаваться родителю и можно использовать `CoroutineExceptionHandler` и
`SupervisorJob`

### Но так же и метод await() бросает исключение

```
val scope = CoroutineScope(Job() + handler)

scope.launch() { // Корутина отменена т.к Job рутовый в scope
    val deffered = async() { // Отменила себя и детей попробовала передать ошибку в скоуп, передала ошибку в handler
        launch { doSomeWork() } // Отменена async
        launch { doSomeWork() } // Отменена async
        throwException()
    }
    
    deffered.await() // Выбрасывается тоже исключение
    doSomeWork() // Не будет выполнена
}
```

### Если await() обернуть в `try-catch`

```
val scope = CoroutineScope(Job() + handler)

scope.launch() { // Корутина отменена т.к Job рутовый в scope
    val deffered = async() { // Отменила себя и детей попробовала передать ошибку в скоуп, передала ошибку в handler
        launch { doSomeWork() } // Отменена async
        launch { doSomeWork() } // Отменена async
        throwException()
    }
    
    try { deffered.await() } catch(e: Exception) { }
    doSomeWork() // Будет выполнена
}
```

### Костыль с передачей `SuperviserJob()` не в scope, юзать не нужно, для ознакомления (async)

```
launch {
   val deferred = async(SupervisorJob(coroutineContext[Job])) { // exсeption }
   val result = deferred.await()
}
```

В этом случае вместо связи:

`launchJob` > `asyncJob`

Мы получаем:

`launchJob` > `supervisorJob` > `asyncJob`

Ошибка из `async` не пойдет вверх по иерархии корутин, при этом `SuperviserJob` ответит `async` что
не может обработать ошибку, `async` не умеет отправлять ошибку в обработчики. Она сохраняет это
исключение внутри себя, чтобы выбросить его, когда вызовут ее метод `await`.

Если обернуть место вызова `await` в `try-catch` ошибка будет обработана

### `coroutineScope` / `supervisorScope`

`сoroutineScope` - это `suspend` функция, внутри которой запускается специальная
корутина `ScopeCoroutine`

Код в блоке `coroutineScope`, становится кодом этой корутины

В примере выше у нас между корутинами 1 и 1_2 образуется связь:

`Coroutine_1` > `ScopeCoroutine` > `Coroutine_1_2`

а не:

`Coroutine_1` > `Coroutine_1_2`

`ScopeCoroutine` не передает ошибку родителю. Если в корутине 1_2 произойдет ошибка, то она пойдет в
`ScopeCoroutine`, которая отменит себя и своих детей.
`ScopeCoroutine` выбросит исключение в свою обертку - suspend функцию coroutineScope, в месте вызова

```
scope.launch(CoroutineName("1")) {
 
   coroutineScope { // Исключение
       launch(CoroutineName("1_1")) {
           // Отменена coroutineScope
       }
 
       launch(CoroutineName("1_2")) {
           // exception
       }
   }
 
   launch(CoroutineName("1_3")) {
      // Отменена scope, тк suspend fun coroutineScope в месте вызова выбросил исключение
   }
 
   launch(CoroutineName("1_4")) {
      // Отменена scope, тк suspend fun coroutineScope в месте вызова выбросил исключение
   }
 
}
```

Обернем `coroutineScope` в `try-catch`:

```
scope.launch(CoroutineName("1")) {
    try {
        coroutineScope { // Исключение
            launch(CoroutineName("1_1")) {
                // Отменена coroutineScope
            }
    
            launch(CoroutineName("1_2")) {
                // exception
            }
        } catch(e: Exception) { }
        
    // Код продолжает выполнение
}
```

Обработчики CoroutineExceptionHandler не работают внутри coroutineScope, потому что ScopeCoroutine
принимает ошибку от корутины 1_2 и говорит, что сможет ее обработать.

supervisorScope
Если coroutineScope принимает ошибки от своих дочерних корутин и просто не шлет их дальше в
родительскую корутину, то supervisorScope даже не принимает ошибку от дочерних. Это приводит к
отличиям в обработке ошибок.

Корутина 1_2, которая пытается передать ошибку наверх в ScopeCoroutine, получает отрицательный ответ
и пытается обработать ошибку сама. Для этого она использует предоставленный ей
CoroutineExceptionHandler. Если же его нет, то будет крэш, который не поймать никаким try-catch.

withContext
withContext это coroutineScope с возможностью добавить/заменить элементы контекста.

## Неотловленная ошибка в колбеке `suspendCoroutine`/ `suspendCancellableCoroutine`

Неотловленная ошибка в теле колбека работает так же как и ошибка в обычной `suspend fun`

```
suspend fun someFunction() = suspendCancellableCoroutine { continuation ->
    throw Exception()
}

```

Неотловленная ошибка в другом потоке попадет в глобальный обработчик, такое может быть при вызове
кода сторонней либы которая в другом потоке выбросит исключение

```

suspend fun someFunction() = suspendCancellableCoroutine { continuation ->
    thread { throw Exception() }
}

```

Отловленные ошибки в `suspendCoroutine`/ `suspendCancellableCoroutine` нужно передавать
в `resumeWithException`

## Неотловленная ошибка в flow

### Можно обернуть в `try-catch` функцию `collect`

```
launch {
   try {
       flow.collect {
           log("collect $it")
       }
   } catch(e: Exception) {
       log("exception $e")
   }
}
```

Ошибка не приведет ни к каким последствиям. А `Flow` просто завершит работу.

### `catch`

* Ловит ошибки из upstream flow
* Можно использовать несколько в цепочке

```
launch {
   flow
       .catch { log("catch $it") }
       .collect {
           log("collect $it")
       }
}
```

### `retry`

* Ловит ошибки из upstream flow
* Перезапустит flow в случае ошибки
* На вход оператор принимает количество попыток. Если все попытки закончатся ошибкой, то
  ретранслирует эту ошибку дальше. Поэтому добавляйте catch после retry если хотите ошибку в итоге
  поймать. Если на вход retry не передавать количество попыток, то по умолчанию оно будет равно
  Long.MAX_VALUE.
* Кроме количества попыток, в оператор retry можно передать блок кода. На вход в этот блок retry
  даст нам пойманную ошибку. А от нас он ждет Boolean ответ, который определяет, надо ли пытаться
  перезапускать Flow

```
launch {
  flow
    .retry(2) {
        it !is ArithmeticException
    }
    .collect {
        log("collect $it")
    }
}
```

Если нужно, чтобы `retry` выждал какую-то паузу прежде чем перезапустить `Flow`, можно использовать
`delay`:

```
launch {
   flow
       .retry(2) {
           if (it is NetworkErrorException) {
               delay(5000)
               true
           }
           false
       }
       .collect {
           log("collect $it")
       }
}
```

### `retryWhen`

* Дает нам в блок кода номер текущей попытки и ошибку. А нам уже надо решить - возвращать true или
  false.

```
launch {
   flow
       .retryWhen { cause, attempt ->
           cause is NetworkErrorException && attempt < 5
       }
       .collect {
           log("collect $it")
       }
}
```

Также в блоке кода `retryWhen` нам доступен метод `emit()`. Т.е. мы можем отправить какие-то свои
данные получателю.





