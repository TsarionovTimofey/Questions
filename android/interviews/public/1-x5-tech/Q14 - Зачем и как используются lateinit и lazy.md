# Зачем и как используются `lateinit` и `Lazy`❓

## Ответ:

1. Модификатор `lateinit` используется для не нулабельных изменяемых `property` которые не могут быть
   инициалированы через конструктор (Пример: эти `property` будут инициализарованы через `Di`, или в
   setUp методе unit тестов)
2. `Lazy` - Один из стандартных property делегатов, принимает лямбду и возвращает
   экземпляр `Lazy<T>` при первом обращении к `get()`, последующие вызовы вернут закешированное
   значение (Пример: Обьявление `member` p`roperty` которое обращается к верстке фрагмента, или `member`
   `property` значение которого может быть понадобится а может и нет)

### Дополнительная информация:

https://kotlinlang.org/docs/properties.html#late-initialized-properties-and-variables
https://kotlinlang.org/docs/delegated-properties.html#lazy-properties