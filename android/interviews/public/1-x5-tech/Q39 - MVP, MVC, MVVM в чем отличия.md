# MVP, MVC, MVVM в чем отличия❓

## Ответ:

### MVC

* `User` взаимодействует с `Controller`
* `Controller` запрашивает данные у `Model` и обновляет `View`

* `UserView` - класс в котором располагаются методы для работы с вью (`View`)
* `UserActivity` - контроллер (`Controller`)
* `UserRepository` - репозиторий с пользовательскими данными (`Model`)

1. `User` кликает на кнопку
2. `UserActivity` вызывает `userRepository.fetchUserData()`
3. `UserActivity` по результату вызывает `userView.showUser() / userView.showError()`

### MVP

* `User` взаимодействует с `View`
* `View` при ивентах от `User` отправляет их на обработку в `Presenter`
* `Presenter` обрабатывает ивент (запрашивает данные у `Model`) и обновляет `View`

* `UserPresenter` - Презентер (`Presenter`)
* `UserActivity` - Активити (`View`)
* `UserRepository` - репозиторий с пользовательскими данными (`Model`)

1. `User` кликает на кнопку
2. `UserActivity` вызывает `userPresenter.onButtonClicked()`
3. `UserPresenter` по результату вызывает `userView.showUser() / userView.showError()`

### MVVM

* `User` взаимодействует с `View`
* `View` подписывается на состояние `ViewModel` и отрисовывает себя от состояний вью модели
* `View` подписывается на события `ViewModel` и обрабатывает их
* `View` при ивентах от `User` отправляет их на обработку в `ViewModel`
* `ViewModel` обрабатывает ивент (запрашивает данные у `Model`) и обновляет свое состояние или отправляет новое значение в
события

* `UserViewModel` - Вьюмодель (`ViewModel`)
* `UserActivity` - Активити (`View`)
* `UserRepository` - репозиторий с пользовательскими данными (`Model`)

1. `User` кликает на кнопку
2. `UserActivity` вызывает `userViewModel.onButtonClicked()`
3. `UserViewModel` обрабатывает ивент (запрашивает данные у `Model`) по результату обновляет поле `_userState`
4. `UserActivity` подписанный на `userState` обновляет себя в соответствии с новым состоянием