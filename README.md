# Что такое SQLAlchemy?

SQLAlchemy – это ORM (Object Relational Mapper)

ORM позволяет работать с базами данных как с объектами Python, а не писать SQL-запросы вручную.

Пример:  
❌ Без ORM (обычный SQL-запрос в Python):  
```python
cursor.execute("INSERT INTO tasks (title) VALUES ('Buy milk')")
```

✅ С ORM (SQLAlchemy):  
```python
new_task = Task(title="Buy milk")
db.session.add(new_task)
db.session.commit()
```

Это удобнее, безопаснее и читабельне

---

## Установка SQLAlchemy

Сначала установим Flask и SQLAlchemy:  
```bash
pip install flask flask-sqlalchemy
```

---

## Настройка Flask и SQLAlchemy

Создадим файл app.py и подключим базу SQLite:
```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)

# Указываем путь к базе данных SQLite
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///database.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False  # Отключаем предупреждения

# Создаем объект базы данных
db = SQLAlchemy(app)
```

💡Что здесь происходит?  
SQLALCHEMY_DATABASE_URI – путь к базе SQLite (файл database.db).  
SQLALCHEMY_TRACK_MODIFICATIONS = False – отключает ненужные предупреждения.  
db = SQLAlchemy(app) – создаем объект базы данных.

---

## Подробное описание работы с Task в SQLAlchemy (Flask ORM)

### Описание модели Task

В SQLAlchemy каждая таблица – это класс в Python.

Добавим модель Task (таблица tasks), в которой будут:  
- id (уникальный идентификатор)  
- title (название задачи)  
- completed (флаг выполнения)  

Модель Task представляет таблицу базы данных tasks, которая будет содержать записи о задачах.

```python
class Task(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100), nullable=False)
    completed = db.Column(db.Boolean, default=False)

    def __repr__(self):
        return f"<Task {self.title}>"
```

📌 Что здесь происходит?  
`id = db.Column(db.Integer, primary_key=True)`  
→ db.Integer → Поле хранит целочисленные значения.  
→ primary_key=True → Значение уникально и автоматически увеличивается (ID записи).  

`title = db.Column(db.String(100), nullable=False)`  
→ db.String(100) → Строка до 100 символов.  
→ nullable=False → Поле обязательно для заполнения (не может быть NULL).  

`completed = db.Column(db.Boolean, default=False)`  
→ db.Boolean → Хранит True/False (статус задачи).  
→ default=False → Новые задачи по умолчанию невыполненные.

---

## Методы SQLAlchemy для работы с Task

Теперь подробно разберем CRUD-операции (Create, Read, Update, Delete).

### 📌 1. Добавление записи в базу (Create)
```python
new_task = Task(title="Buy milk")  # Создаем объект Task
db.session.add(new_task)  # Добавляем объект в сессию
db.session.commit()  # Сохраняем изменения в базе
```

Объяснение кода:  
Создаем объект класса Task, передавая ему title="Buy milk".  
Добавляем объект в сессию db.session.add(new_task).  
Сохраняем изменения в базе с помощью db.session.commit().

Результат:  
Добавится новая строка в таблицу tasks с заголовком "Buy milk" и completed=False.

---

### 📌 2. Получение всех записей (Read - Чтение)
```python
tasks = Task.query.all()
for task in tasks:
    print(task.id, task.title, task.completed)
```

Объяснение кода:  
Task.query.all() → Получает все записи из таблицы.  
Проходим по списку объектов tasks, выводя их **id, title, completed**.  

SQL-запрос в базе эквивалентен:  
`SELECT * FROM tasks;`

---

### 📌 3. Получение записи по ID
```python
task = Task.query.get(1)
print(task.title)
```

💡 Аналог SQL:  
`SELECT * FROM tasks WHERE id=1;`

---

### 📌 4. Поиск записей по условию
```python
task = Task.query.filter_by(title="Buy milk").first()
print(task.id, task.title)
```

Объяснение кода:  
Task.query.filter_by(title="Buy milk") → Выбирает задачи, где title="Buy milk".  
.first() → Возвращает первую найденную запись.  
Если таких записей нет, вернется None.

💡 Аналог SQL:  
`SELECT * FROM tasks WHERE title = 'Buy milk' LIMIT 1;`

---

### 📌 5. Обновление записи в базе (Update)
```python
task = Task.query.get(1)  # Получаем задачу с ID 1
task.title = "Buy coffee"  # Изменяем заголовок
db.session.commit()  # Сохраняем изменения
```

💡 Аналог SQL:  
`UPDATE tasks SET title = 'Buy coffee' WHERE id = 1;`

#### 📌 Альтернативный способ через filter_by()
```python
Task.query.filter_by(title="Buy milk").update({"title": "Buy coffee"})
db.session.commit()
```

💡 Аналог SQL:  
`UPDATE tasks SET title = 'Buy coffee' WHERE title = 'Buy milk';`

---

### 📌 6. Удаление записи (Delete)
```python
task = Task.query.get(1)  # Получаем задачу по ID
db.session.delete(task)  # Удаляем
db.session.commit()  # Подтверждаем изменения
```

💡 Аналог SQL:  
`DELETE FROM tasks WHERE id = 1;`

#### 📌 Альтернативный способ через filter_by()
```python
Task.query.filter_by(title="Buy coffee").delete()
db.session.commit()
```

💡 Аналог SQL:  
`DELETE FROM tasks WHERE title = 'Buy coffee';`

---

## 📌 Полезные функции SQLAlchemy для работы с базой данных во Flask

SQLAlchemy предоставляет множество удобных функций и методов для взаимодействия с базой данных. Давай разберем ключевые функции, которые помогут упростить и ускорить работу с базой в Flask-приложениях.

---

### Получение данных (READ - SELECT)

🔹 `query.all()` – Получение всех записей из таблицы  
```python
tasks = Task.query.all()
for task in tasks:
    print(task.id, task.title, task.completed)
```
💡 Аналог SQL:  
`SELECT * FROM tasks;`

---

🔹 `query.get(id)` – Получение записи по ID  
```python
task = Task.query.get(2)
print(task.title)
```
💡 Аналог SQL:  
`SELECT * FROM tasks WHERE id = 2;`

❗ Если записи нет, вернется None (ошибки не будет).

---

🔹 `query.get_or_404(id)` – Получение записи или ошибка 404  
```python
task = Task.query.get_or_404(5)
```
💡 Используется в Flask для обработки ошибок.

---

🔹 `query.filter_by()` – Фильтрация по одному полю  
```python
task = Task.query.filter_by(title="Buy milk").first()
```
💡 Аналог SQL:  
`SELECT * FROM tasks WHERE title = 'Buy milk' LIMIT 1;`

---

🔹 `query.filter()` – Фильтрация с операторами (==, !=, >, <)  
```python
tasks = Task.query.filter(Task.completed == False).all()
```
💡 Аналог SQL:  
`SELECT * FROM tasks WHERE completed = 0;`

---

🔹 `query.order_by()` – Сортировка  
```python
tasks = Task.query.order_by(Task.id.desc()).all()
```
💡 Аналог SQL:  
`SELECT * FROM tasks ORDER BY id DESC;`

---

🔹 `query.limit(n)` – Ограничение количества записей  
```python
tasks = Task.query.limit(3).all()
```
💡 Аналог SQL:  
`SELECT * FROM tasks LIMIT 3;`

---

## Добавление данных (CREATE - INSERT)

🔹 `db.session.add()` – Добавление новой записи  
```python
new_task = Task(title="Go to the gym")
db.session.add(new_task)
db.session.commit()
```
💡 Аналог SQL:  
`INSERT INTO tasks (title, completed) VALUES ('Go to the gym', 0);`

---

🔹 `db.session.bulk_save_objects()` – Массовое добавление записей  
```python
tasks = [
    Task(title="Buy milk"),
    Task(title="Write code"),
    Task(title="Read book")
]
db.session.bulk_save_objects(tasks)
db.session.commit()
```
💡 Быстрее, чем db.session.add() для каждого объекта!

---

## Обновление данных (UPDATE)

🔹 `query.get(id)` + изменение + commit()  
```python
task = Task.query.get(1)
task.completed = True
db.session.commit()
```
💡 Аналог SQL:  
`UPDATE tasks SET completed = 1 WHERE id = 1;`

---

🔹 `query.filter_by().update()` – Массовое обновление записей  
```python
Task.query.filter_by(completed=False).update({"completed": True})
db.session.commit()
```
💡 Аналог SQL:  
`UPDATE tasks SET completed = 1 WHERE completed = 0;`

---

## Удаление данных (DELETE)

🔹 `db.session.delete()` – Удаление одной записи  
```python
task = Task.query.get(3)
db.session.delete(task)
db.session.commit()
```
💡 Аналог SQL:  
`DELETE FROM tasks WHERE id = 3;`

---

🔹 `query.filter().delete()` – Массовое удаление записей  
```python
Task.query.filter(Task.completed == True).delete()
db.session.commit()
```
💡 Аналог SQL:  
`DELETE FROM tasks WHERE completed = 1;`

---

## Подсчет записей в таблице

🔹 `query.count()` – Подсчет количества записей  
```python
count = Task.query.count()
print(f"Всего задач: {count}")
```
💡 Аналог SQL:  
`SELECT COUNT(*) FROM tasks;`

---

🔹 `query.filter().count()` – Подсчет записей с фильтром  
```python
completed_tasks = Task.query.filter(Task.completed == True).count()
print(f"Выполненных задач: {completed_tasks}")
```
💡 Аналог SQL:  
`SELECT COUNT(*) FROM tasks WHERE completed = 1;`

---

## Проверка существования записи

🔹 `query.exists()` – Проверка наличия записи  
```python
task_exists = db.session.query(Task.id).filter_by(title="Buy milk").first() is not None
print("Задача существует!" if task_exists else "Задачи нет.")
```
💡 Аналог SQL:  
`SELECT 1 FROM tasks WHERE title = 'Buy milk' LIMIT 1;`

---

## Создание базы данных и таблицы

После объявления модели нужно создать таблицу в базе:  
Добавь этот код в `app.py` (после объявления класса Task):

```python
with app.app_context():
    db.create_all()
```

Теперь запусти файл, и он создаст `database.db`

✅ Проверить базу можно через DB Browser for SQLite или командой:
```bash
sqlite3 database.db
.tables
```

---

## CRUD-операции с SQLAlchemy

Теперь реализуем CRUD-операции:  
- Create (создание)  
- Read (чтение)  
- Update (обновление)  
- Delete (удаление)

---

### 📌 Добавление данных (Create)

Добавим новую задачу в базу:
```python
@app.route('/add/<title>')
def add_task(title):
    new_task = Task(title=title)  # Создаем объект задачи
    db.session.add(new_task)  # Добавляем в сессию
    db.session.commit()  # Сохраняем в базе
    return f"Task '{title}' added!"
```

Теперь перейди в браузере на:  
`http://127.0.0.1:5000/add/Buy milk`  
✅ Задача "Buy milk" добавится в базу.

---

### 📌 Чтение данных (Read)

Вывести все задачи:
```python
@app.route('/tasks')
def get_tasks():
    tasks = Task.query.all()  # Получаем все записи из таблицы
    return {task.id: {"title": task.title, "completed": task.completed} for task in tasks}
```

Запроси в браузере:  
`http://127.0.0.1:5000/tasks`  
✅ Получишь JSON со всеми задачами.

---

### 📌 Обновление данных (Update)

Изменим заголовок задачи:
```python
@app.route('/update/<int:id>/<new_title>')
def update_task(id, new_title):
    task = Task.query.get_or_404(id)  # Получаем задачу по ID
    task.title = new_title  # Меняем заголовок
    db.session.commit()  # Сохраняем изменения
    return f"Task {id} updated to '{new_title}'!"
```

Измени задачу по ID:  
`http://127.0.0.1:5000/update/1/Buy coffee`  
✅ Задача "Buy milk" теперь называется "Buy coffee".

---

### 📌 Удаление данных (Delete)

Удалим задачу по ID:
```python
@app.route('/delete/<int:id>')
def delete_task(id):
    task = Task.query.get_or_404(id)  # Найти задачу
    db.session.delete(task)  # Удалить из базы
    db.session.commit()  # Сохранить изменения
    return f"Task {id} deleted!"
```

Удаление в браузере:  
`http://127.0.0.1:5000/delete/1`  
✅ Задача с ID 1 удалится.

