# Build type, Build variant, Build flavor, расскажите что это такое зачем нужно❓

## Ответ:

### Build variant

Применяются для представления различных версий приложения

`Build variant` это сочетание `build type` и `product flavor`, `<product-flavor><Build-Type>`

* `DevelopDebug`
* `ProductionRelease`

### Build type

По умолчанию студия создает `debug` и `release` типы сборки

В зависимости от `build type` используются разные ключи для подписи приложения

Или используются разные зависимости: `implementationRelease` / `implementationDebug` в блоке `dependencies` модуля

### Build flavour

Наиболее часто применяются для задания базовых урлов энпоинтов:

* `develop`
* `production`
* `staging`

А так же задания ресурсов: `strings-develop`

В сочетании с `flavour dimensions` можно сделать `build variant` с несколькими примененными `build flavour`:

* `DevelopInternalDebug`

* `ProductionInternalRelease`

* `ProductionPlaystoreRelease`

Пример:

### Дополнительная информация:

https://developer.android.com/build/build-variants#kts