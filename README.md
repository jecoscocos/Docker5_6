# Docker5_6

# Todo Application

Веб-приложение для управления задачами, построенное с использованием React (frontend), FastAPI (backend) и PostgreSQL (database).

## Структура проекта

```
PRlab6/
├── frontend/          # React приложение
│   ├── src/          # Исходный код
│   ├── public/       # Статические файлы
│   ├── package.json  # Зависимости и скрипты
│   └── Dockerfile    # Конфигурация Docker для frontend
├── backend/          # FastAPI приложение
│   ├── src/         # Исходный код
│   ├── requirements.txt  # Python зависимости
│   ├── .env.example # Пример конфигурации окружения
│   └── Dockerfile   # Конфигурация Docker для backend
├── database/        # PostgreSQL
│   ├── init.sql     # Скрипт инициализации БД
│   └── Dockerfile   # Конфигурация Docker для БД
└── .github/        # GitHub Actions
    └── workflows/   # CI/CD конфигурация
```

## Настройка окружения

### 1. Настройка переменных окружения

1. Скопируйте файл `.env.example` в `.env` в директории backend:
```powershell
cd backend
Copy-Item .env.example .env
```

2. Отредактируйте `.env` файл, установив свои значения:
```env
# База данных
POSTGRES_DB=tododb
POSTGRES_USER=your_db_user
POSTGRES_PASSWORD=your_secure_password
POSTGRES_HOST=todo-db
POSTGRES_PORT=5432

# SMTP сервер (для Gmail)
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=465
EMAIL_USER=your_email@gmail.com
EMAIL_PASSWORD=your_app_password
EMAIL_FROM=your_email@gmail.com

# IMAP сервер
IMAP_HOST=imap.gmail.com
IMAP_PORT=993
IMAP_USER=your_email@gmail.com
IMAP_PASSWORD=your_app_password

# POP3 сервер
POP3_HOST=pop.gmail.com
POP3_PORT=995
POP3_USER=your_email@gmail.com
POP3_PASSWORD=your_app_password
```

### 2. Настройка GitHub Secrets (для CI/CD)

Для работы CI/CD необходимо добавить следующие секреты в ваш GitHub репозиторий (Settings → Secrets and variables → Actions → New repository secret):

1. Для Docker Hub:
   - `DOCKER_USERNAME` - имя пользователя в Docker Hub
   - `DOCKER_PASSWORD` - пароль или токен Docker Hub

2. Для базы данных:
   - `DB_USER` - имя пользователя базы данных
   - `DB_PASSWORD` - пароль базы данных
   - `DB_NAME` - имя базы данных

3. Для Email сервисов:
   - `EMAIL_HOST` - SMTP хост
   - `EMAIL_PORT` - SMTP порт
   - `EMAIL_USER` - email пользователя
   - `EMAIL_PASSWORD` - пароль приложения
   - `EMAIL_FROM` - email отправителя
   - `IMAP_HOST` - IMAP хост
   - `IMAP_PORT` - IMAP порт
   - `IMAP_USER` - IMAP пользователь
   - `IMAP_PASSWORD` - IMAP пароль
   - `POP3_HOST` - POP3 хост
   - `POP3_PORT` - POP3 порт
   - `POP3_USER` - POP3 пользователь
   - `POP3_PASSWORD` - POP3 пароль

## Запуск приложения

### Создание Docker сети

```powershell
docker network create todo-network
```

### Запуск контейнеров

```powershell
# Сборка и запуск базы данных
cd database
docker build -t todo-db .
docker run -d --name todo-db --network todo-network `
  -p 5432:5432 `
  -e POSTGRES_DB=your_db_name `
  -e POSTGRES_USER=your_db_user `
  -e POSTGRES_PASSWORD=your_db_password `
  todo-db

# Сборка и запуск backend
cd ../backend
docker build -t todo-backend .
docker run -d --name todo-backend --network todo-network `
  -p 8000:8000 `
  --env-file .env `
  todo-backend

# Сборка и запуск frontend
cd ../frontend
docker build -t todo-frontend .
docker run -d --name todo-frontend --network todo-network `
  -p 3000:3000 `
  -e REACT_APP_API_URL=http://localhost:8000 `
  todo-frontend
```

### Управление контейнерами

```powershell
# Просмотр запущенных контейнеров
docker ps

# Остановка контейнеров
docker stop todo-db todo-backend todo-frontend

# Удаление контейнеров
docker rm todo-db todo-backend todo-frontend

# Просмотр логов
docker logs -f todo-backend
```

## Проверка работы приложения

1. Frontend: http://localhost:3000
2. Backend API: 
   - Swagger UI: http://localhost:8000/docs
   - ReDoc: http://localhost:8000/redoc
3. База данных:
   - Host: localhost
   - Port: 5432
   - Database: tododb (или ваше значение из .env)
   - Username: (значение из .env)
   - Password: (значение из .env)

## CI/CD

Проект настроен для автоматической сборки и тестирования при пуше в ветку main. CI/CD пайплайн:
1. Собирает Docker образы
2. Запускает тесты
3. Публикует образы в Docker Hub (при успешных тестах)

## Полезные команды

### Остановка и удаление контейнеров
```powershell
# Остановка всех контейнеров
docker ps -aq | ForEach-Object { docker stop $_ }

# Удаление всех контейнеров
docker ps -aq | ForEach-Object { docker rm $_ }

# Остановка конкретного контейнера
docker stop todo-db
docker stop todo-backend
docker stop todo-frontend

# Удаление конкретного контейнера
docker rm todo-db
docker rm todo-backend
docker rm todo-frontend

# Принудительное удаление контейнера
docker rm -f todo-db
```

### Управление образами
```powershell
# Просмотр всех образов
docker images

# Удаление образа базы данных
docker images todo-db -q | ForEach-Object { docker rmi $_ -f }

# Удаление образа бэкенда
docker images todo-backend -q | ForEach-Object { docker rmi $_ -f }

# Удаление образа фронтенда
docker images todo-frontend -q | ForEach-Object { docker rmi $_ -f }

# Удаление всех образов
docker images -q | ForEach-Object { docker rmi $_ -f }
```

### Управление сетями
```powershell
# Просмотр всех сетей
docker network ls

# Создание новой сети
docker network create todo-network

# Подключение контейнера к сети
docker network connect todo-network todo-db
docker network connect todo-network todo-backend
docker network connect todo-network todo-frontend

# Отключение контейнера от сети
docker network disconnect todo-network todo-db

# Удаление сети
docker network rm todo-network
```

### Просмотр логов и отладка
```powershell
# Просмотр логов контейнера
docker logs todo-frontend
docker logs todo-backend
docker logs todo-db

# Просмотр логов в реальном времени
docker logs -f todo-frontend

# Просмотр последних 100 строк логов
docker logs --tail 100 todo-frontend

# Просмотр использования ресурсов
docker stats

# Просмотр детальной информации о контейнере
docker inspect todo-db
```

### Очистка системы
```powershell
# Удаление всех остановленных контейнеров
docker container prune

# Удаление всех неиспользуемых образов
docker image prune

# Удаление всех неиспользуемых сетей
docker network prune

# Полная очистка системы (контейнеры, образы, сети)
docker system prune

# Полная очистка системы с удалением всех образов
docker system prune -a
```

### Работа с базой данных
```powershell
# Подключение к PostgreSQL
docker exec -it todo-db psql -U your_db_user -d your_db_name

# Основные команды PostgreSQL
\dt                    # показать все таблицы
\d имя_таблицы        # показать структуру таблицы
SELECT * FROM tasks;   # показать все задачи
\q                    # выйти
```

### Пересборка всего проекта
```powershell
# 1. Остановка всех контейнеров
docker ps -aq | ForEach-Object { docker stop $_ }

# 2. Удаление всех контейнеров
docker ps -aq | ForEach-Object { docker rm $_ }

# 3. Удаление всех образов
docker images todo-db -q | ForEach-Object { docker rmi $_ -f }
docker images todo-backend -q | ForEach-Object { docker rmi $_ -f }
docker images todo-frontend -q | ForEach-Object { docker rmi $_ -f }

# 4. Создание сети
docker network create todo-network

# 5. Сборка и запуск базы данных
cd database
docker build -t todo-db .
docker run -d --name todo-db --network todo-network -p 5432:5432 -e POSTGRES_DB=your_db_name -e POSTGRES_USER=your_db_user -e POSTGRES_PASSWORD=your_db_password todo-db

# 6. Сборка и запуск бэкенда
cd ../backend
docker build -t todo-backend .
docker run -d --name todo-backend --network todo-network -p 8000:8000 --env-file .env todo-backend

# 7. Сборка и запуск фронтенда
cd ../frontend
docker build -t todo-frontend .
docker run -d --name todo-frontend --network todo-network -p 3000:3000 -e REACT_APP_API_URL=http://localhost:8000 todo-frontend
```

### Примечания для PowerShell
1. В PowerShell для переноса длинных команд используется обратный апостроф (`) вместо обратного слеша (\)
2. Для работы с переменными используется синтаксис `$env:VARIABLE_NAME`
3. Для циклов и обработки вывода используется `ForEach-Object` вместо pipe в bash
4. Для выполнения нескольких команд в одной строке используется точка с запятой (;)
5. Для переноса длинных команд на новую строку используется обратный апостроф (`)