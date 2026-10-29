# Анализ потоков данных (Data Flow Analysis)

## 1. Цепочки Def-Use для переменных
* **Переменная:** `user_id`
  - Def: Строка 1 (аргумент функции)
  - Use: Строка 6 (`get_balance(user_id)`), Строка 8 (`update_balance(user_id, ...)`)

* **Переменная:** `amount`
  - Def: Строка 1 (аргумент функции)
  - Use: Строка 4 (`amount > 0`), Строка 7 (`balance - amount`)

* **Переменная:** `db_connection`
  - Def: Строка 1 (аргумент функции)
  - Use: Не используется (Мертвое определение)

* **Переменная:** `log_entry`
  - Def 1: Строка 2 (`log_entry = "Start processing"`) — *Мертвое присваивание*
  - Def 2: Строка 10 (`log_entry = "Transaction finished with status: " + status`)
  - Use: Строка 13 (`file.write(log_entry)`)

* **Переменная:** `status`
  - Def: Строка 5 (`status = "approved"`) — *Внутри блока if*
  - Use: Строка 10 (в конкатенации строки), Строка 15 (`return status`)

* **Переменная:** `balance`
  - Def: Строка 6 (`balance = get_balance(user_id)`)
  - Use: Строка 7 (`balance - amount`)

* **Переменная:** `new_balance`
  - Def: Строка 7 (`new_balance = balance - amount`)
  - Use: Строка 8 (`update_balance(user_id, new_balance)`)

* **Переменная:** `file`
  - Def: Строка 12 (`file = open(...)`)
  - Use: Строка 13 (`file.write(log_entry)`) — *Утечка ресурса (нет close)*

## 2. Выявленные скрытые дефекты
1. **Использование неинициализированной переменной:** Переменная `status` определяется только внутри блока `if amount > 0`. Если `amount <= 0`, условие не выполнится, и на строке 10 при попытке склеить строку возникнет ошибка `UnboundLocalError`.
2. **Мертвое присваивание (Dead Store):** На строке 2 переменной `log_entry` присваивается значение `"Start processing"`, однако на строке 10 это значение полностью перезаписывается без предварительного чтения или логирования.
3. **Утечка ресурса (Resource Leak):** На строке 12 дескриптор файла `file = open(...)` захватывает системный ресурс, но нигде не освобождается (отсутствует вызов `file.close()`).
