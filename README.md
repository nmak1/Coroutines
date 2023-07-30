# Домашнее задание к занятию «10. Coroutines: Scopes, Cancellation, Supervision»

Выполненное задание прикрепите ссылкой на ваши GitHub-проекты в личном кабинете студента на сайте [netology.ru](https://netology.ru).

## Вопросы: Cancellation

### Вопрос №1

Отработает ли в этом коде строка `<--`? Поясните, почему да или нет.



```kotlin
fun main() = runBlocking {
    val job = CoroutineScope(EmptyCoroutineContext).launch {
        launch {
            delay(500)
            println("ok") // <--
        }
        launch {
            delay(500)
            println("ok")
        }
    }
    delay(100)
    job.cancelAndJoin()
}
```
* Не отработает, так как функция job.cancelAndJoin() отменяет корутины вывода на печать "ок" до их выполнения.
* Можно заменить job.cancelAndJoin() -> job.join() или изменить задержку на её выполнение, установив значение
* в delay больше 500

### Вопрос №2

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() = runBlocking {
    val job = CoroutineScope(EmptyCoroutineContext).launch {
        val child = launch {
            delay(500)
            println("ok") // <--
        }
        launch {
            delay(500)
            println("ok")
        }
        delay(100)
        child.cancel()
    }
    delay(100)
    job.join()
}
```

* Не отработает, так как функция child.cancel() отменяет корутину вывода на печать первой "ок" до её выполнения.
* При этом второй "ок" выводится на печать нормально.
* Можно заменить child.cancel() -> child.join() или изменить задержку на её выполнение, установив значение
* в delay больше 500
 

## Вопросы: Exception Handling

### Вопрос №1

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() {
    with(CoroutineScope(EmptyCoroutineContext)) {
        try {
            launch {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```
* Не отработает, так как прежде выведется исключение Exception("something bad happened")
* Из презентации к лекции © "Нет смысла писать вот такой код, он не перехватит никаких исключений"

### Вопрос №2

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            coroutineScope {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```
* Отработает, выбросится исключение Exception "something bad happened".
* и в отладчике можно наблюдать Coroutine Creation Stack Trace

### Вопрос №3

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            supervisorScope {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```
* Отработает, так как функция supervisorScope приостановит выброс исключения Exception "something bad happened"
* до выполнения её child с печатью stacktrace.
* Также в отладчике можно наблюдать Coroutine Creation Stack Trace.

### Вопрос №4

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            coroutineScope {
                launch {
                    delay(500)
                    throw Exception("something bad happened") // <--
                }
                launch {
                    throw Exception("something bad happened")
                }
            }
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
    Thread.sleep(1000)
}
```
* Не отработает, так как функция delay приостановит поток, в результате чего выйдет второй Exception.
* Можно отключить задержку выполнения корутины.

### Вопрос №5

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            supervisorScope {
                launch {
                    delay(500)
                    throw Exception("something bad happened") // <--
                }
                launch {
                    throw Exception("something bad happened")
                }
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```
* Отработает, так как корутины запускаются в функции supervisorScope, которая выполняется, когда завершаются
* все её child.

### Вопрос №6

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        CoroutineScope(EmptyCoroutineContext).launch {
            launch {
                delay(1000)
                println("ok") // <--
            }
            launch {
                delay(500)
                println("ok")
            }
            throw Exception("something bad happened")
        }
    }
    Thread.sleep(1000)
}
```
* Не отработает, так как прежде выведется исключение Exception("something bad happened").
* Для выполнения печати первой "ок", нужно заменить функцию launch на async и уменьшить время задержки delay

### Вопрос №7

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        CoroutineScope(EmptyCoroutineContext + SupervisorJob()).launch {
            launch {
                delay(1000)
                println("ok") // <--
            }
            launch {
                delay(500)
                println("ok")
            }
            throw Exception("something bad happened")
        }
    }
    Thread.sleep(1000)
}
```
* Не отработает, так как прежде выведется исключение Exception("something bad happened").
* Для выполнения печати первой "ок", можно обрабатывать ошибки только вывода на печать второй "ок",
* тогда он не будет влиять на вывод печати первой "ок" и уменьшить время задержки delay первой "ок"