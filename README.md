# 💻 LaptopShop

Магазин ноутбуков

Документация по практическим занятиям 7–11

Дисциплина: Фронтенд и бэкенд разработка

4 семестр, 2025/2026 уч. год

---

## 🚀 Стек

Node.js • Express • React • JWT • bcrypt

---

## 📁 Структура проекта

Все практики (7–11) реализованы в одном монорепозитории.
Бэкенд и фронтенд запускаются независимо в двух терминалах.

```bash
laptop-shop-rbac/

backend/

  index.js          — Express сервер, все маршруты, middleware

  package.json

frontend/

  public/index.html

  src/

    api/index.js              — Axios клиент + interceptors

    context/AuthContext.js    — глобальное состояние пользователя

    components/Navbar.js      — шапка с бейджем роли

    pages/CatalogPage.js      — каталог ноутбуков

    pages/ProductPage.js      — детальная страница товара

    pages/LoginPage.js        — страница входа

    pages/RegisterPage.js     — регистрация с выбором роли

    pages/ProfilePage.js      — профиль и права пользователя

    pages/AdminUsersPage.js   — управление пользователями (admin)

README.md
```

---

## ▶️ Запуск проекта

Открыть два терминала и выполнить:

### Терминал 1 — Бэкенд (порт 3001)

```bash
cd backend

npm install

node index.js
```

### Терминал 2 — Фронтенд (порт 3000)

```bash
cd frontend

npm install

npm start
```

После запуска открыть в браузере:
http://localhost:3000

---

# 📚 Практическое занятие 7 — Хеширование паролей (bcrypt)

## Тема

Базовые методы аутентификации. Хеширование паролей с использованием bcrypt и соли.

## Теория

Хеширование паролей — преобразование пароля в хеш с помощью математической функции.
Хеш нельзя обратно превратить в пароль (односторонняя функция).
При входе пароль снова хешируется и сравнивается с хранимым хешем.

## Алгоритм bcrypt

bcrypt — алгоритм на основе шифра Blowfish, созданный специально для хеширования паролей.

Ключевые особенности:

* Встроенная соль — случайная строка, добавляемая к паролю перед хешированием. Защищает от атак по радужным таблицам.
* Параметр cost (rounds) — задаёт число раундов хеширования. Чем больше, тем медленнее перебор.
* Типичное значение rounds = 10–12 для production-приложений.

## Установка

```bash
npm install bcrypt
```

## Реализация в проекте

Файл: backend/index.js

```js
const bcrypt = require('bcrypt');



// Хеширование при регистрации

const passwordHash = await bcrypt.hash(password, 10);



// Проверка при входе

const isValid = await bcrypt.compare(password, user.passwordHash);
```

## Реализованные маршруты

| Маршрут            | Метод | Описание                                     |
| ------------------ | ----- | -------------------------------------------- |
| /api/auth/register | POST  | Регистрация пользователя (хешируется пароль) |
| /api/auth/login    | POST  | Вход в систему (сравнение хешей)             |

## Сущность Пользователь

| Поле         | Тип    | Описание                          |
| ------------ | ------ | --------------------------------- |
| id           | string | Уникальный идентификатор (nanoid) |
| email        | string | Email — используется как логин    |
| first_name   | string | Имя пользователя                  |
| last_name    | string | Фамилия пользователя              |
| passwordHash | string | Хеш пароля (bcrypt, 10 rounds)    |

---

# 🔐 Практическое занятие 8 — JWT-токены и защищённые маршруты

## Тема

Работа с JSON Web Token (JWT). Создание и валидация токенов, защита маршрутов.

## Теория

JWT (JSON Web Token) — открытый стандарт (RFC 7519) для передачи данных после аутентификации.

Токен состоит из трёх частей, разделённых точкой:

* Header — алгоритм подписи (например, HS256)
* Payload — полезная нагрузка: id, email, роль пользователя
* Signature — подпись, гарантирующая что токен выдан именно нашим сервером

## Алгоритм работы

1. Клиент отправляет логин и пароль на сервер.
2. Сервер проверяет данные и выдаёт access-токен.
3. При каждом запросе клиент передаёт токен в заголовке Authorization: Bearer <token>.
4. Сервер проверяет токен через authMiddleware и предоставляет доступ.

## Установка

```bash
npm install jsonwebtoken
```

## Реализация в проекте

```js
const jwt = require('jsonwebtoken');

const ACCESS_SECRET = 'laptop_rbac_access_2025';

const ACCESS_EXPIRES_IN = '15m';



// Создание токена

const token = jwt.sign({ sub: user.id, email: user.email, role: user.role },

                        ACCESS_SECRET, { expiresIn: ACCESS_EXPIRES_IN });



// Валидация токена (authMiddleware)

const payload = jwt.verify(token, ACCESS_SECRET);

req.user = payload; // { sub, email, role, iat, exp }
```

## Маршруты

| Маршрут            | Метод  | Доступ    | Описание                   |
| ------------------ | ------ | --------- | -------------------------- |
| /api/auth/register | POST   | Публичный | Регистрация пользователя   |
| /api/auth/login    | POST   | Публичный | Вход, выдача access-токена |
| /api/auth/me       | GET    | 🔒 Токен  | Текущий пользователь       |
| /api/products      | GET    | 🔒 Токен  | Список ноутбуков           |
| /api/products/:id  | GET    | 🔒 Токен  | Ноутбук по id              |
| /api/products      | POST   | 🔒 Токен  | Создать ноутбук            |
| /api/products/:id  | PUT    | 🔒 Токен  | Обновить ноутбук           |
| /api/products/:id  | DELETE | 🔒 Токен  | Удалить ноутбук            |

## Сущность Товар (Ноутбук)

| Поле        | Тип    | Описание                              |
| ----------- | ------ | ------------------------------------- |
| id          | string | Уникальный идентификатор (nanoid)     |
| title       | string | Название ноутбука                     |
| category    | string | Категория (Apple, Игровые, Бизнес...) |
| description | string | Описание характеристик                |
| price       | number | Цена в рублях                         |
| image       | string | URL фотографии товара                 |

---

# 🔄 Практическое занятие 9 — Refresh-токены

## Тема

Разделение access- и refresh-токенов. Автоматическое обновление пары токенов.

## Теория

Access-токен живёт короткое время (15 минут) — чтобы при его краже у злоумышленника было минимум времени.
Но постоянно вводить пароль неудобно. Refresh-токен решает эту проблему:

* Access-токен — короткоживущий (15 минут), передаётся в каждом запросе
* Refresh-токен — долгоживущий (7 дней), используется только для получения нового access-токена

Ротация токенов — при обновлении старый refresh-токен удаляется, выдаётся новая пара

## Схема работы

1. Вход: сервер выдаёт пару access + refresh токенов.
2. Запросы: клиент использует access-токен.
3. Истёк access: клиент отправляет refresh-токен на /api/auth/refresh.
4. Сервер проверяет refresh, удаляет старый, выдаёт новую пару.
5. Истёк refresh: пользователь должен войти заново.

## Реализация в проекте

```js
const REFRESH_SECRET = 'laptop_rbac_refresh_2025';

const REFRESH_EXPIRES_IN = '7d';



// Хранилище refresh-токенов (in-memory Set)

const refreshTokens = new Set();



// При логине — добавляем в Set

refreshTokens.add(refreshToken);



// При обновлении — ротация

refreshTokens.delete(oldRefreshToken);

refreshTokens.add(newRefreshToken);
```

## Новый маршрут

| Маршрут           | Метод | Тело запроса              | Ответ                                           |
| ----------------- | ----- | ------------------------- | ----------------------------------------------- |
| /api/auth/refresh | POST  | { "refreshToken": "..." } | { "accessToken": "...", "refreshToken": "..." } |

---

# ⚛️ Практическое занятие 10 — Фронтенд на React

## Тема

Хранение токенов на фронтенде, Axios interceptors, автоматическое обновление токенов.

## Хранение токенов

Токены хранятся в localStorage браузера:

```js
localStorage.setItem('accessToken',  data.accessToken);

localStorage.setItem('refreshToken', data.refreshToken);
```

При каждом запросе Axios автоматически подставляет токен через interceptor.

## Axios interceptors

Файл: frontend/src/api/index.js

Request interceptor — подставляет access-токен в заголовок:

```js
apiClient.interceptors.request.use((config) => {

  const token = localStorage.getItem('accessToken');

  if (token) config.headers.Authorization = `Bearer ${token}`;

  return config;

});
```

Response interceptor — при ошибке 401 автоматически обновляет токен:

```js
apiClient.interceptors.response.use(response => response, async (error) => {

  if (error.response?.status === 401 && !original._retry) {

    // Обновляем токен через /api/auth/refresh

    // Повторяем исходный запрос с новым токеном

  }

});
```

## Структура фронтенда

| Файл                   | Назначение                                        |
| ---------------------- | ------------------------------------------------- |
| api/index.js           | Axios клиент + interceptors                       |
| context/AuthContext.js | Глобальный стейт: user, login, logout, register   |
| components/Navbar.js   | Навигация — меняется в зависимости от авторизации |
| pages/CatalogPage.js   | Список всех ноутбуков                             |
| pages/ProductPage.js   | Детальная страница + редактирование               |
| pages/LoginPage.js     | Форма входа                                       |
| pages/RegisterPage.js  | Форма регистрации                                 |
| pages/ProfilePage.js   | Профиль текущего пользователя                     |

## Страницы и маршруты React Router

| URL           | Страница     | Доступ                      |
| ------------- | ------------ | --------------------------- |
| /             | CatalogPage  | Авторизованные пользователи |
| /login        | LoginPage    | Все                         |
| /register     | RegisterPage | Все                         |
| /products/:id | ProductPage  | Авторизованные              |
| /profile      | ProfilePage  | Авторизованные              |

---

# 🛡 Практическое занятие 11 — RBAC (управление доступом на основе ролей)

## Тема

Role-Based Access Control. Разграничение прав пользователей через систему ролей.

## Теория

RBAC — модель контроля доступа, при которой права назначаются не напрямую пользователям, а через роли.
Каждая роль — набор разрешений для определённого вида задач.

## Роли в проекте

| Роль   | Описание                                       | Кто это               |
| ------ | ---------------------------------------------- | --------------------- |
| user   | Только просмотр товаров                        | Покупатель на сайте   |
| seller | user + создание и редактирование товаров       | Сотрудник магазина    |
| admin  | seller + удаление товаров + управление юзерами | Администратор системы |

## Таблица прав доступа

| Действие               | user | seller | admin |
| ---------------------- | ---- | ------ | ----- |
| Просмотр каталога      | ✅    | ✅      | ✅     |
| Просмотр товара по id  | ✅    | ✅      | ✅     |
| Создание товара        | ❌    | ✅      | ✅     |
| Редактирование товара  | ❌    | ✅      | ✅     |
| Удаление товара        | ❌    | ❌      | ✅     |
| Просмотр пользователей | ❌    | ❌      | ✅     |
| Редактирование юзеров  | ❌    | ❌      | ✅     |
| Блокировка юзеров      | ❌    | ❌      | ✅     |

## Реализация на бэкенде

Два middleware в файле backend/index.js:

### authMiddleware

```js
function authMiddleware(req, res, next) {

  const [scheme, token] = req.headers.authorization.split(' ');

  if (scheme !== 'Bearer' || !token) return res.status(401)...

  req.user = jwt.verify(token, ACCESS_SECRET); // { sub, role, ... }

  next();

}
```

### roleMiddleware

```js
function roleMiddleware(allowedRoles) {

  return (req, res, next) => {

    if (!allowedRoles.includes(req.user.role))

      return res.status(403).json({ error: 'Forbidden' });

    next();

  };

}
```

## Применение на маршрутах

```js
// Только seller и admin
app.post('/api/products', authMiddleware, roleMiddleware(['seller','admin']), ...)


// Только admin
app.get('/api/users', authMiddleware, roleMiddleware(['admin']), ...)
```

## Полная таблица API маршрутов

| Маршрут            | Метод  | Роль         | Описание                       |
| ------------------ | ------ | ------------ | ------------------------------ |
| /api/auth/register | POST   | Гость        | Регистрация пользователя       |
| /api/auth/login    | POST   | Гость        | Вход в систему                 |
| /api/auth/refresh  | POST   | Гость        | Обновление пары токенов        |
| /api/auth/me       | GET    | Все роли     | Получить текущего пользователя |
| /api/users         | GET    | admin        | Список всех пользователей      |
| /api/users/:id     | GET    | admin        | Пользователь по id             |
| /api/users/:id     | PUT    | admin        | Обновить данные пользователя   |
| /api/users/:id     | DELETE | admin        | Заблокировать пользователя     |
| /api/products      | GET    | все роли     | Список ноутбуков               |
| /api/products/:id  | GET    | все роли     | Ноутбук по id                  |
| /api/products      | POST   | seller/admin | Создать ноутбук                |
| /api/products/:id  | PUT    | seller/admin | Обновить ноутбук               |
| /api/products/:id  | DELETE | admin        | Удалить ноутбук                |

## Реализация на фронтенде

Файл: frontend/src/context/AuthContext.js

Хелперы ролей:

```js
const isUser   = !!user;

const isSeller = user?.role === 'seller' || role === 'admin';

const isAdmin  = user?.role === 'admin';
```

Использование:

```js
const { user, isSeller, isAdmin } = useAuth();

{isSeller && <button>+ Добавить ноутбук</button>}
{isAdmin  && <button>🗑 Удалить</button>}
{isAdmin  && <Link to='/admin/users'>Пользователи</Link>}
```

## Визуальные отличия ролей

| Роль   | Бейдж                      | Возможности               |
| ------ | -------------------------- | ------------------------- |
| user   | 👤 Пользователь (зелёный)  | Просмотр                  |
| seller | 🛍 Продавец (оранжевый)    | Добавление/редактирование |
| admin  | 👑 Администратор (красный) | Полный доступ             |

---

# 🎯 Итог: что реализовано в проекте

| Практика | Тема           | Что сделано                                                       |
| -------- | -------------- | ----------------------------------------------------------------- |
| 7        | bcrypt         | Хеширование паролей при регистрации, проверка при входе           |
| 8        | JWT            | Access-токен, authMiddleware, защищённые маршруты, /api/auth/me   |
| 9        | Refresh        | Refresh-токены, ротация, маршрут /api/auth/refresh                |
| 10       | React фронтенд | Axios interceptors, AuthContext, страницы                         |
| 11       | RBAC           | Роли user/seller/admin, roleMiddleware, управление пользователями |

---
