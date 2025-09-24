# mathkallpos
Веб-сайт онлайн школа для подготовки к студенческим олимпиадам по математике с интеграцией искусственного интеллекта и возможности авторизоваться пользователям, чтобы отслеживать KPI.

MathKallos — Полная документация (до реализации)

Назначение: документация описывает архитектуру, структуру, API, процесс разработки и развёртывания веб-платформы MathKallos — онлайн-школы подготовки к студенческим олимпиадам по математике с AI-модулем генерации задач и системой авторизации/KPI.

Эта документация рассчитана на разработчиков и DevOps-инженеров. Описаны пошаговые инструкции по локальной разработке, запуску через Docker, описание API, структура проекта, миграции, тестирование, CI/CD и рекомендации по продакшену.

Содержание

Обзор проекта

Архитектура и компоненты

Технологический стек

Структура репозитория

Модели данных (схема БД)

API: спецификация, примеры запросов

Аутентификация и авторизация (JWT / OAuth2)

UI / Frontend: структура, маршруты, компоненты

Модуль AI: интеграция, примеры prompt'ов, безопасность

KPI/аналитика: что и как хранить/считать

Локальная разработка: шаги для backend и frontend

Docker / docker-compose — конфигурации и запуск

Базы данных и миграции

Тестирование и качество кода

CI/CD (пример GitHub Actions)

Продакшен-развёртывание: рекомендации (Nginx, TLS, scaling)

Безопасность, хранение секретов, GDPR

Мониторинг, логирование, трейсинг

Roadmap (этапы реализации)

FAQ и отладка

Приложения (сниппеты и конфигурации)

1. Обзор проекта

MathKallos — платформа для студентов технических вузов, целью которой является подготовка к олимпиадам по математике.

Основные возможности (MVP и далее):

Регистрация/вход пользователей; роли: student, teacher, admin.

Личный кабинет с прогрессом и статистикой, история попыток, KPI.

Курсы и теоретические разделы.

Тесты / задания: разные типы вопросов (строковые ответы, числовые, множественный выбор, LaTeX-формулы, задачи с вводом шага решения).

AI-модуль для генерации задач по заданной теме и создания экзаменов (пакетов задач).

Автоматическая проверка решений (где применимо) и модуль ручной проверки для развёрнутых решений.

Админ-панель для управления курсами, тестами, пользователями и отчетами.

2. Архитектура и компоненты

Frontend (React + Tailwind, опционально Next.js) — SPA/SSR клиент.

Backend (FastAPI) — REST API, авторизация JWT, бизнес-логика, интеграция с AI.

PostgreSQL — хранение пользователей, курсов, тестов, результатов и KPI.

AI-сервис — обращение к OpenAI API (или локальным моделям Hugging Face).

Хранилище файлов — S3/minio.

Контейнеризация — Docker + docker-compose.

Мониторинг — Sentry, Prometheus, Grafana.

3. Технологический стек

Frontend: React, TailwindCSS, Vite/Next.js, axios

Backend: Python 3.11+, FastAPI, SQLAlchemy/SQLModel, Alembic

DB: PostgreSQL 14+

Auth: JWT (PyJWT), OAuth2

AI: OpenAI API / HuggingFace

Контейнеризация: Docker, docker-compose

CI/CD: GitHub Actions

Тесты: pytest

4. Структура репозитория
mathkallos/
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── api/v1/
│   │   │   ├── auth.py
│   │   │   ├── users.py
│   │   │   ├── courses.py
│   │   │   ├── tests.py
│   │   │   └── ai.py
│   │   ├── models/
│   │   ├── schemas/
│   │   ├── services/
│   │   └── db/
│   ├── alembic/
│   └── Dockerfile
├── frontend/
│   ├── src/
│   │   ├── pages/
│   │   ├── components/
│   │   └── services/
│   └── Dockerfile
├── docker-compose.yml
└── README.md

5. Модели данных (схема БД)

users

id, email, password_hash, full_name, role, created_at

courses

id, title, slug, description, syllabus, created_by

sections

id, course_id, title, content, order

tests

id, course_id, title, questions (JSON), duration

questions

id, test_id, type, statement, options, answer_key, difficulty

attempts

id, user_id, test_id, answers, score, duration

ai_generated_tasks

id, user_id, topic, prompt, tasks (JSON)

analytics

id, user_id, metric_date, metric_type, value

6. API: спецификация

Auth:

POST /auth/register

POST /auth/login

Users:

GET /users/me

PUT /users/me

Courses:

GET /courses

GET /courses/{id}

Tests:

GET /tests/{course_id}

POST /tests/submit

AI:

POST /ai/generate

7. Аутентификация и авторизация

JWT: sub, role, exp.

Роли: student, teacher, admin.

Refresh-токены — опционально.

8. UI / Frontend

Основные страницы:

/auth/login, /auth/register

/dashboard — личный кабинет

/courses, /courses/[slug]

/tests/[testId]

/ai/generate

/admin

Компоненты: AuthForm, CourseCard, TestRunner, ProgressChart, AiGeneratorForm.

9. Модуль AI

Сервис ai_service.py — общение с OpenAI API.

Prompt: генерация задач по теме X в формате JSON.

Валидация результата на backend.

Ограничения: rate-limits, кеширование.

10. KPI / аналитика

Метрики: tests_taken, avg_score, time_spent_minutes, topics_mastered, streak_days.
Сохраняются в таблице analytics.
Отображаются графиками в личном кабинете.

11. Локальная разработка
git clone ...
cd mathkallos

# backend
cd backend
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload

# frontend
cd frontend
npm install
npm run dev

12. Docker / docker-compose
docker-compose up --build


Backend → http://localhost:8000

Frontend → http://localhost:3000

13. База данных и миграции

Alembic команды:

alembic revision --autogenerate -m "init"
alembic upgrade head

14. Тестирование

pytest --maxfail=1 -q

Pre-commit: black, isort, flake8

15. CI/CD (GitHub Actions)

Workflow для backend и frontend.

Шаги: checkout → install deps → pytest / npm test.

CD: билд Docker → push → deploy.

16. Продакшен-развёртывание

Backend: gunicorn+uvicorn.

Nginx — TLS termination.

Let's Encrypt для HTTPS.

Secrets в Vault/Cloud.

Scaling через Kubernetes.

17. Безопасность

Пароли: bcrypt/argon2.

CSRF при cookie-токенах.

Rate-limits.

GDPR: возможность удалить аккаунт.

18. Мониторинг

Логи: JSON, отправка в ELK/Datadog.

Метрики: Prometheus + Grafana.

Ошибки: Sentry.

19. Roadmap

MVP: регистрация, вход, курсы, тесты вручную.

Тестирование, роли, админка.

AI-модуль.

Аналитика, рекомендации, геймификация.

Масштабирование: мобильное приложение, LMS интеграция.

20. FAQ

Backend не стартует → проверить .env, миграции, логи.

Frontend не видит API → CORS.

AI возвращает мусор → логировать, валидировать JSON.

21. Приложения
Пример .env
DATABASE_URL=postgresql://postgres:postgres@db:5432/mathkallos
OPENAI_API_KEY=sk-...

Nginx конфиг
server {
    listen 80;
    server_name mathkallos.com;

    location /api/ {
        proxy_pass http://backend:8000/;
    }

    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri /index.html;
    }
}


curl -X POST http://localhost:8000/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"student@example.com","password":"Test1234"}'
