# Какие требования предъявляются к property чтобы ее можно было объявить как `const`❓

## Ответ:

* Это должно быть top-level property или член объявления объекта или `companion object`
* Оно должно быть инициализировано значением типа `String` или примитивного типа
* Не должно иметь кастомного геттера

### Дополнительная информация:

https://kotlinlang.org/docs/properties.html#compile-time-constants