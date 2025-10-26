# Алгоритм создания пользователя на стороне бэкенд-сервиса

После получения POST-запроса на `/api/v1/users/register` бэкенд выполняет следующие шаги:

---

## Шаг 1: Валидация входных данных

- Проверить, что все обязательные поля (`firstName`, `lastName`, `username`, `password`, `recaptchaToken`) присутствуют.
- Проверить типы и ограничения:
  - `firstName`, `lastName`: строка, 1–50 символов.
  - `username`: строка, 3–30 символов, только буквы, цифры, подчеркивание.
  - `password`: строка, ≥8 символов, содержит хотя бы одну заглавную, строчную букву, цифру и спецсимвол.
  - `recaptchaToken`: строка, не пустая.

→ Если ошибка → вернуть 400 Bad Request с детализированными сообщениями.

---

## Шаг 2: Проверка reCAPTCHA

- Отправить `recaptchaToken` на сервер Google reCAPTCHA v2/v3.
- Получить ответ: `success: true/false`.
- Если `success == false` → вернуть 422 Unprocessable Entity.

---

## Шаг 3: Проверка уникальности имени пользователя

- Выполнить запрос к БД: `SELECT id FROM users WHERE username = ?`
- Если запись найдена → вернуть 409 Conflict с сообщением «User with username '{username}' already exists».

---

## Шаг 4: Хеширование пароля

- Использовать алгоритм **bcrypt** с cost = 12.
- Сохранить хеш в переменной `hashedPassword`.

---

## Шаг 5: Создание записи в БД

- Сгенерировать UUID для `id`.
- Выполнить INSERT в таблицу `users`:
  ```sql
  INSERT INTO users (id, first_name, last_name, username, password_hash, created_at)
  VALUES (?, ?, ?, ?, ?, NOW())
