# Отчет по лабораторной работе №1

**Автор:** Гуторова Инна  
**Группа:** К3341  

# FastAPI Time Manager

Это веб-приложение для управления задачами, с категорией, тегами, напоминаниями и JWT-аутентификацией. Стек технологий: FastAPI + SQLModel + PostgreSQL.

## Состав проекта

- Аутентификация и регистрация
- CRUD для задач, тегов, напоминаний, категорий
- Связи между задачами и тегами
- JWT защита всех эндпоинтов
- Документация Swagger (автоматическая от FastAPI)

# Модели

## Task

```
class Task(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    title: str
    description: Optional[str]
    due_date: Optional[datetime]
    completed: bool = False
    user_id: int = Field(foreign_key="user.id")
    category_id: Optional[int] = Field(foreign_key="category.id")
```

## Tag

```
class Tag(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    name: str
```

## Reminder

```
class Reminder(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    message: str
    remind_at: datetime
    task_id: int = Field(foreign_key="task.id")
```

## Category

```
class Category(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    name: str
```

## TaskTag

```
class TaskTagLink(SQLModel, table=True):
    task_id: int = Field(foreign_key="task.id", primary_key=True)
    tag_id: int = Field(foreign_key="tag.id", primary_key=True)
    is_primary: bool
```

## User

```
class User(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    username: str = Field(index=True, unique=True)
    email: str = Field(index=True, unique=True)
    hashed_password: str
    created_at: datetime = Field(default_factory=datetime.utcnow)
    is_active: bool = True
```

# Эндпоинты

> Все эндпоинты, кроме регистрации и логина, требуют авторизации через JWT-токен.

## Аутентификация (`/auth`)

- `POST /auth/register` — Register
- `POST /auth/login` — Login
- `GET /auth/me` — Read Current User
- `PATCH /auth/password` — Update Password

## Пользователи (`/users`)

- `GET /users/me/details` — Get Current User Extended
- `GET /users/` — Get All Users
- `GET /users/{user_id}` — Get User By Id
- `GET /users/{user_id}/details` — Get User Extended By Id

## Задачи (`/tasks`)

- `POST /tasks/` — Create Task
- `GET /tasks/` — Read Tasks
- `GET /tasks/{task_id}` — Read Task
- `PATCH /tasks/{task_id}` — Update Task
- `DELETE /tasks/{task_id}` — Delete Task
- `POST /tasks/{task_id}/tags/{tag_id}` — Add Tag To Task
- `DELETE /tasks/{task_id}/tags/{tag_id}` — Remove Tag From Task

## Категории (`/categories`)

- `POST /categories/` — Create Category
- `GET /categories/` — Read Categories
- `GET /categories/{category_id}` — Read Category
- `PATCH /categories/{category_id}` — Update Category
- `DELETE /categories/{category_id}` — Delete Category

## Теги (`/tags`)

- `POST /tags/` — Create Tag
- `GET /tags/` — Read Tags
- `GET /tags/{tag_id}` — Read Tag
- `DELETE /tags/{tag_id}` — Delete Tag

## Напоминания (`/reminders`)

- `POST /reminders/` — Create Reminder
- `GET /reminders/` — Read Reminders
- `GET /reminders/{reminder_id}` — Read Reminder
- `DELETE /reminders/{reminder_id}` — Delete Reminder

## Главная (`/`)

- `GET /` — Read Root

## Ссылки

- [Исходный код](https://github.com/TonikX/ITMO_ICT_WebDevelopment_tools_2024-2025/pull/15)
```