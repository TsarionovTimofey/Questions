# Выполнится ли assert?

### Условие:

```
data class Student(val firstName: String, val secondName: String)

fun main() {
    val student = Student("Ivan", "Ivanov")

    with(student) {
        println("student ${student.firstName} ${student.secondName}")
    }

    val newSecondName = "Petrov"
    val applyStudent = student.apply { Student(firstName, newSecondName) }
    val letStudent = student.let { Student(it.firstName, newSecondName) }

    assert(applyStudent == letStudent)
}
```

### Ответ:

Нет ❌

### Дополнительная информация:

https://kotlinlang.org/docs/scope-functions.html