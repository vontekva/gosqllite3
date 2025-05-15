# Документация gosqllite3

**Супер упрощенная библеотека для работы с sqllite3**

## **Функции**  

ВАЖНО: в parametrs во всех функциях за исключенем request, commit, close передается список, первым значением передается **НАЗВАНИЕ ТАБЛИЦЫ**: ["tablename, "id INT"] (пример)
### **`connect(file: str)`**  
Открывает соединение с базой данных SQLite.  
- **`file`** (str) – путь к файлу БД (например, `"database.db"`). Если файла нет, он будет создан 

### **`create(parameters: list, auto_commit: bool = True)`**  
Создает таблицу с указанными параметрами.  
- **`parameters`** (list) – список столбцов в формате `["id INTEGER PRIMARY KEY", "name TEXT", "age INTEGER"]`.  
- **`auto_commit`** (bool) – если `True` (по умолчанию), изменения сохраняются сразу

### **`selectall(parameters: list = None, where: str = None)`**  
Выбирает все данные из таблицы (или с условием).  
- **`parameters`** (list) – список столбцов для выборки (например, `["name", "age"]`). Если `None`, выбирает все (`SELECT *`).  
- **`where`** (str) – условие в формате SQL (например, `"age > 18"`)

### **`delete(parameters: list = None, where: str = None, auto_commit: bool = True)`**  
Удаляет записи из таблицы.  
- **`parameters`** (list) – столбцы для уточнения удаления (редко используется).  
- **`where`** (str) – условие (например, `"id = 5"`). **Если `None`, удаляет все записи!**  
- **`auto_commit`** (bool) – автосохранение изменений.  

### **`insert(parameters: list, values: list, auto_commit: bool = True)`**  
Добавляет новую запись в таблицу.  
- **`parameters`** (list) – столбцы (например, `["name", "age"]`).  
- **`values`** (list) – значения (например, `["Alice", 25]`).  
- **`auto_commit`** (bool) – автосохранение.  

### **`update(parameters: list, where: str = None, auto_commit: bool = True)`**  
Обновляет записи в таблице.  
- **`parameters`** (list) – пары `["столбец = новое_значение", ...]` (например, `["age = 26", "name = 'Bob'"]`).  
- **`where`** (str) – условие (например, `"id = 1"`). **Если `None`, обновит все записи!**  
- **`auto_commit`** (bool) – автосохранение.  

### **`request(request_text: str)`**  
Выполняет произвольный SQL-запрос.  
- **`request_text`** (str) – SQL-команда (например, `"ALTER TABLE users ADD COLUMN email TEXT"`).  

### **`commit()`**  
Сохраняет изменения в БД (если `auto_commit=False`).  

### **`close()`**  
Закрывает соединение с БД.  


