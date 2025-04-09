# 🧠 Введение в SQLAlchemy и Flask ORM

## Что такое SQLAlchemy?

**SQLAlchemy** — это **ORM** (Object Relational Mapper), который позволяет работать с базами данных как с объектами Python, без необходимости писать SQL-запросы вручную.

### Пример:

#### ❌ Без ORM (обычный SQL-запрос в Python):
```python
cursor.execute("INSERT INTO tasks (title) VALUES ('Buy milk')")
```

#### ✅ С ORM (SQLAlchemy):
```python
new_task = Task(title="Buy milk")
db.session.add(new_task)
db.session.commit()
```

> Это **удобнее**, **безопаснее** и **читабельнее**.

---

## 📦 Установка SQLAlchemy

Установим Flask и SQLAlchemy:
```bash
pip install flask flask-sqlalchemy
```

---

## ⚙️ Настройка Flask и SQLAlchemy

Создадим файл `app.py` и подключим базу SQLite:

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)

# Путь к базе данных SQLite
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///database.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

# Создание объекта базы данных
db = SQLAlchemy(app)
```

### 💡 Что происходит:
- `SQLALCHEMY_DATABASE_URI` — путь к базе данных.
- `SQLALCHEMY_TRACK_MODIFICATIONS` — отключает предупреждения.
- `db = SQLAlchemy(app)` — инициализация ORM.

---

## 📋 Описание модели Task (таблица `tasks`)

В SQLAlchemy каждая таблица — это класс:

```python
class Task(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100), nullable=False)
    completed = db.Column(db.Boolean, default=False)

    def __repr__(self):
        return f"<Task {self.title}>"
```

### 🧩 Объяснение полей:
- `id` — целое число, первичный ключ.
- `title` — строка до 100 символов, не может быть `NULL`.
- `completed` — булево значение, по умолчанию `False`.

---

## 🔁 CRUD-операции с Task

### 1. ➕ Добавление (Create)
```python
new_task = Task(title="Buy milk")
db.session.add(new_task)
db.session.commit()
```

### 2. 📖 Получение всех записей (Read)
```python
tasks = Task.query.all()
for task in tasks:
    print(task.id, task.title, task.completed)
```

### 3. 🔍 Получение записи по ID
```python
task = Task.query.get(1)
print(task.title)
```

### 4. 🔎 Поиск по условию
```python
task = Task.query.filter_by(title="Buy milk").first()
print(task.id, task.title)
```

### 5. 📝 Обновление записи (Update)
```python
task = Task.query.get(1)
task.title = "Buy coffee"
db.session.commit()
```

#### Альтернатива:
```python
Task.query.filter_by(title="Buy milk").update({"title": "Buy coffee"})
db.session.commit()
```

### 6. ❌ Удаление записи (Delete)
```python
task = Task.query.get(1)
db.session.delete(task)
db.session.commit()
```

#### Альтернатива:
```python
Task.query.filter_by(title="Buy coffee").delete()
db.session.commit()
```

---

## 🧰 Полезные методы SQLAlchemy

### 📖 Получение данных
```python
Task.query.all()
Task.query.get(2)
Task.query.get_or_404(5)
Task.query.filter_by(title="Buy milk").first()
Task.query.filter(Task.completed == False).all()
Task.query.order_by(Task.id.desc()).all()
Task.query.limit(3).all()
```

### ➕ Добавление данных
```python
db.session.add(Task(title="Go to the gym"))
db.session.commit()
```

#### Массовое добавление:
```python
tasks = [
    Task(title="Buy milk"),
    Task(title="Write code"),
    Task(title="Read book")
]
db.session.bulk_save_objects(tasks)
db.session.commit()
```

### 📝 Обновление
```python
task = Task.query.get(1)
task.completed = True
db.session.commit()
```

#### Массовое обновление:
```python
Task.query.filter_by(completed=False).update({"completed": True})
db.session.commit()
```

### ❌ Удаление
```python
task = Task.query.get(3)
db.session.delete(task)
db.session.commit()
```

#### Массовое удаление:
```python
Task.query.filter(Task.completed == True).delete()
db.session.commit()
```

### 🔢 Подсчет записей
```python
Task.query.count()
Task.query.filter(Task.completed == True).count()
```

### ❓ Проверка существования записи
```python
exists = db.session.query(Task.id).filter_by(title="Buy milk").first() is not None
```

---

## 🏗️ Создание базы данных и таблиц

Добавь в `app.py` после модели:

```python
with app.app_context():
    db.create_all()
```

✅ После запуска файла `database.db` будет создан.

---

## 🌐 Реализация CRUD-операций через Flask-маршруты

### ➕ Добавление задачи
```python
@app.route('/add/<title>')
def add_task(title):
    new_task = Task(title=title)
    db.session.add(new_task)
    db.session.commit()
    return f"Task '{title}' added!"
```

**Пример запроса:**  
`http://127.0.0.1:5000/add/Buy milk`

---

### 📖 Получение задач
```python
@app.route('/tasks')
def get_tasks():
    tasks = Task.query.all()
    return {task.id: {"title": task.title, "completed": task.completed} for task in tasks}
```

**Пример запроса:**  
`http://127.0.0.1:5000/tasks`

---

### 📝 Обновление задачи
```python
@app.route('/update/<int:id>/<new_title>')
def update_task(id, new_title):
    task = Task.query.get_or_404(id)
    task.title = new_title
    db.session.commit()
    return f"Task {id} updated to '{new_title}'!"
```

**Пример запроса:**  
`http://127.0.0.1:5000/update/1/Buy coffee`

---

### ❌ Удаление задачи
```python
@app.route('/delete/<int:id>')
def delete_task(id):
    task = Task.query.get_or_404(id)
    db.session.delete(task)
    db.session.commit()
    return f"Task {id} deleted!"
```

**Пример запроса:**  
`http://127.0.0.1:5000/delete/1`
