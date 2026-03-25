# 💻 LaptopShop — Магазин ноутбуков

## Документация по практическим занятиям 7–11

**Дисциплина:** Фронтенд и бэкенд разработка  
**Семестр:** 4-й, 2025/2026 уч. год  
**Стек:** Node.js • Express • React • JWT • bcrypt

---

## Структура проекта

Практические занятия 7–11 реализованы в одном монорепозитории. Бэкенд и фронтенд запускаются независимо в двух терминалах.
laptop-shop-rbac/
├── backend/
│ ├── index.js # Express-сервер, маршруты, middleware
│ └── package.json
├── frontend/
│ ├── public/
│ │ └── index.html
│ └── src/
│ ├── api/
│ │ └── index.js # Axios-клиент + interceptors
│ ├── context/
│ │ └── AuthContext.js # Глобальное состояние пользователя
│ ├── components/
│ │ └── Navbar.js # Шапка с бейджем роли
│ └── pages/
│ ├── CatalogPage.js # Каталог ноутбуков
│ ├── ProductPage.js # Детальная страница товара
│ ├── LoginPage.js # Страница входа
│ ├── RegisterPage.js # Регистрация с выбором роли
│ ├── ProfilePage.js # Профиль и права пользователя
│ └── AdminUsersPage.js # Управление пользователями (admin)
└── README.md

text

---

## Запуск проекта

Откройте два терминала и выполните:

**Терминал 1 — Бэкенд (порт 3001)**
```bash
cd backend
npm install
node index.js
Терминал 2 — Фронтенд (порт 3000)

bash
cd frontend
npm install
npm start
```
После запуска откройте в браузере: http://localhost:3000

Практическое занятие 7 — Хеширование паролей (bcrypt)
Теория
Хеширование паролей — преобразование пароля в хеш с помощью односторонней функции. При входе пароль снова хешируется и сравнивается с хранимым хешем.

bcrypt — алгоритм на основе Blowfish:

Встроенная соль — защита от радужных таблиц.

Параметр cost (rounds) — задаёт сложность вычислений (обычно 10–12).

Установка
bash
npm install bcrypt
Реализация в проекте (backend/index.js)
javascript
const bcrypt = require('bcrypt');

// Хеширование при регистрации
const passwordHash = await bcrypt.hash(password, 10);

// Проверка при входе
const isValid = await bcrypt.compare(password, user.passwordHash);
Маршруты
Маршрут	Метод	Описание
/api/auth/register	POST	Регистрация (хеширование пароля)
/api/auth/login	POST	Вход (сравнение хешей)
Сущность «Пользователь»
Поле	Тип	Описание
id	string	Уникальный идентификатор (nanoid)
email	string	Логин пользователя
first_name	string	Имя
last_name	string	Фамилия
passwordHash	string	Хеш пароля (bcrypt, 10 rounds)
Практическое занятие 8 — JWT-токены и защищённые маршруты
Теория
JWT (JSON Web Token) состоит из трёх частей:
Header.Payload.Signature.
Payload содержит id, email, role.
Токен подписывается секретным ключом и передаётся в заголовке Authorization: Bearer <token>.

Установка
bash ```
npm install```
jsonwebtoken
Реализация в проекте (backend/index.js)
javascript
const jwt = require('jsonwebtoken');
const ACCESS_SECRET = 'laptop_rbac_access_2025';
const ACCESS_EXPIRES_IN = '15m';

// Создание токена
```
const token = jwt.sign(
  { sub: user.id, email: user.email, role: user.role },
  ACCESS_SECRET,
  { expiresIn: ACCESS_EXPIRES_IN }
);
```
// Валидация токена (authMiddleware)
const payload = jwt.verify(token, ACCESS_SECRET);
req.user = payload; // { sub, email, role, iat, exp }
Маршруты
Маршрут	Метод	Доступ	Описание
/api/auth/register	POST	Публичный	Регистрация
/api/auth/login	POST	Публичный	Вход, выдача токена
/api/auth/me	GET	🔒 Токен	Текущий пользователь
/api/products	GET	🔒 Токен	Список ноутбуков
/api/products/:id	GET	🔒 Токен	Ноутбук по id
/api/products	POST	🔒 Токен	Создать ноутбук
/api/products/:id	PUT	🔒 Токен	Обновить ноутбук
/api/products/:id	DELETE	🔒 Токен	Удалить ноутбук
Сущность «Товар (Ноутбук)»
Поле	Тип	Описание
id	string	Уникальный идентификатор (nanoid)
title	string	Название
category	string	Категория (Apple, Игровые, Бизнес)
description	string	Описание характеристик
price	number	Цена в рублях
image	string	URL фотографии
Практическое занятие 9 — Refresh-токены
Теория
Access-токен — короткоживущий (15 минут), передаётся в каждом запросе.

Refresh-токен — долгоживущий (7 дней), используется только для получения нового access-токена.

Ротация токенов — при обновлении старый refresh-токен удаляется, выдаётся новая пара.

Реализация в проекте (backend/index.js)
javascript
const REFRESH_SECRET = 'laptop_rbac_refresh_2025';
const REFRESH_EXPIRES_IN = '7d';

// Хранилище refresh-токенов (in-memory)
const refreshTokens = new Set();

// При логине
refreshTokens.add(refreshToken);

// При обновлении (ротация)
refreshTokens.delete(oldRefreshToken);
refreshTokens.add(newRefreshToken);
Новый маршрут
Маршрут	Метод	Тело запроса	Ответ
/api/auth/refresh	POST	{ "refreshToken": "..." }	{ "accessToken": "...", "refreshToken": "..." }
Практическое занятие 10 — Фронтенд на React
Хранение токенов
javascript
localStorage.setItem('accessToken', data.accessToken);
localStorage.setItem('refreshToken', data.refreshToken);
Axios interceptors (frontend/src/api/index.js)
Request interceptor — добавляет access-токен в заголовок:

javascript
apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('accessToken');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});
Response interceptor — при ошибке 401 автоматически обновляет токен:

javascript
apiClient.interceptors.response.use(
  response => response,
  async (error) => {
    if (error.response?.status === 401 && !original._retry) {
      // Обновление токена через /api/auth/refresh
      // Повтор исходного запроса
    }
  }
);
Структура фронтенда
Файл	Назначение
api/index.js	Axios-клиент + interceptors
context/AuthContext.js	Глобальный стейт: user, login, logout, register
components/Navbar.js	Навигация в зависимости от авторизации
pages/CatalogPage.js	Список ноутбуков
pages/ProductPage.js	Детальная страница + редактирование
pages/LoginPage.js	Форма входа
pages/RegisterPage.js	Форма регистрации
pages/ProfilePage.js	Профиль текущего пользователя
Страницы и маршруты React Router
URL	Страница	Доступ
/	CatalogPage	Авторизованные
/login	LoginPage	Все
/register	RegisterPage	Все
/products/:id	ProductPage	Авторизованные
/profile	ProfilePage	Авторизованные
Практическое занятие 11 — RBAC (управление доступом на основе ролей)
Теория
RBAC — модель, в которой права назначаются через роли.

Роли в проекте
Роль	Описание
user	Только просмотр товаров
seller	user + создание и редактирование товаров
admin	seller + удаление товаров + управление юзерами
Таблица прав доступа
Действие	user	seller	admin
Просмотр каталога	✅	✅	✅
Просмотр товара	✅	✅	✅
Создание товара	❌	✅	✅
Редактирование товара	❌	✅	✅
Удаление товара	❌	❌	✅
Просмотр пользователей	❌	❌	✅
Редактирование юзеров	❌	❌	✅
Блокировка юзеров	❌	❌	✅
Реализация на бэкенде (backend/index.js)
1. authMiddleware — проверяет JWT

javascript
function authMiddleware(req, res, next) {
  const [scheme, token] = req.headers.authorization.split(' ');
  if (scheme !== 'Bearer' || !token) return res.status(401)...
  req.user = jwt.verify(token, ACCESS_SECRET);
  next();
}
2. roleMiddleware — проверяет роль

javascript
function roleMiddleware(allowedRoles) {
  return (req, res, next) => {
    if (!allowedRoles.includes(req.user.role))
      return res.status(403).json({ error: 'Forbidden' });
    next();
  };
}
Применение на маршрутах

javascript
// Только seller и admin
app.post('/api/products', authMiddleware, roleMiddleware(['seller','admin']), ...)

// Только admin
app.get('/api/users', authMiddleware, roleMiddleware(['admin']), ...)
Полная таблица API маршрутов
Маршрут	Метод	Роль	Описание
/api/auth/register	POST	Гость	Регистрация
/api/auth/login	POST	Гость	Вход
/api/auth/refresh	POST	Гость	Обновление пары токенов
/api/auth/me	GET	Все роли	Текущий пользователь
/api/users	GET	admin	Список пользователей
/api/users/:id	GET	admin	Пользователь по id
/api/users/:id	PUT	admin	Обновить пользователя
/api/users/:id	DELETE	admin	Заблокировать пользователя
/api/products	GET	Все роли	Список ноутбуков
/api/products/:id	GET	Все роли	Ноутбук по id
/api/products	POST	seller/admin	Создать ноутбук
/api/products/:id	PUT	seller/admin	Обновить ноутбук
/api/products/:id	DELETE	admin	Удалить ноутбук
Реализация на фронтенде (frontend/src/context/AuthContext.js)
Хелперы ролей доступны через хук useAuth():

javascript
const isUser = !!user;                // любой авторизованный
const isSeller = user?.role === 'seller' || user?.role === 'admin';
const isAdmin = user?.role === 'admin';
Использование в компонентах:

javascript
const { user, isSeller, isAdmin } = useAuth();

{isSeller && <button>+ Добавить ноутбук</button>}
{isAdmin && <button>🗑 Удалить</button>}
{isAdmin && <Link to='/admin/users'>Пользователи</Link>}
Визуальные отличия ролей
Роль	Бейдж в Navbar	Доступные функции
user	👤 Пользователь (зелёный)	Просмотр каталога и карточек товаров
seller	🛍 Продавец (оранжевый)	Добавление и редактирование товаров
admin	👑 Администратор (красный)	Всё + удаление товаров + управление пользователями
Итог: что реализовано в проекте
Практика	Тема	Что сделано
7	bcrypt	Хеширование паролей при регистрации, проверка при входе
8	JWT	Access-токен, authMiddleware, защищённые маршруты, /api/auth/me
9	Refresh	Refresh-токены, ротация, маршрут /api/auth/refresh
10	React фронтенд	Axios interceptors, AuthContext, страницы каталога, товара, профиля
11	RBAC	Роли user/seller/admin, roleMiddleware, управление пользователями
