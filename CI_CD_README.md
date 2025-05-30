# Настройка CI/CD с помощью GitHub Actions для Todo приложения

## Цель работы
Настроить непрерывную интеграцию и доставку (CI/CD) для приложения, автоматизировать процесс сборки контейнеров, тестирования и деплоя.

## Теоретическая часть

### Основы CI/CD

CI/CD (Непрерывная интеграция и непрерывная доставка) - это набор практик, направленных на автоматизацию процесса разработки и выпуска программного обеспечения.

**Непрерывная интеграция (CI)** обеспечивает автоматическую проверку кода при каждом коммите в репозиторий, запуская сборку и тесты. Это помогает рано выявлять проблемы и поддерживать стабильность кодовой базы.

**Непрерывная доставка (CD)** автоматизирует процесс выпуска новых версий приложения, подготавливая их к развертыванию.

### GitHub Actions

GitHub Actions - это сервис CI/CD, встроенный в GitHub. Основные преимущества:
- Тесная интеграция с GitHub
- Простота настройки с использованием YAML-файлов
- Широкие возможности настройки рабочих процессов
- Поддержка различных окружений и триггеров

### Примеры GitHub Actions для Docker

GitHub Actions удобен для работы с Docker, позволяя автоматизировать:
- Сборку Docker-образов
- Тестирование приложений внутри контейнеров
- Загрузку образов в реестры контейнеров (Docker Hub, GitHub Container Registry)
- Развертывание контейнеров в различных окружениях

## Ход выполнения

### 1. Создание GitHub Actions workflow

В рамках работы были созданы два workflow:

1. **Основной CI/CD workflow** (`.github/workflows/ci-cd.yml`) - запускается при каждом push или pull request в ветку main.
2. **Релизный workflow** (`.github/workflows/release.yml`) - запускается при создании тега с версией (v*.*.*)

Основные этапы каждого workflow:
1. Настройка окружения и создание Docker-сети
2. Сборка и запуск контейнеров (база данных, бэкенд, фронтенд)
3. Тестирование работоспособности сервисов через HTTP-запросы
4. Загрузка образов в Docker Hub
5. Очистка ресурсов

### 2. Сборка образов и тестирование

Workflow последовательно выполняет:
- Сборку трех Docker-образов (БД, бэкенд, фронтенд)
- Запуск контейнеров с правильной конфигурацией сети
- Проверку работоспособности бэкенда через запросы к API (GET /api/health, GET /tasks)
- Проверку доступности фронтенда

### 3. Загрузка образов в Docker Hub

После успешного тестирования образы загружаются в Docker Hub с использованием секретов репозитория для авторизации:
```yaml
- name: Авторизация в Docker Hub
  uses: docker/login-action@v2
  with:
    username: ${{ secrets.DOCKER_USERNAME }}
    password: ${{ secrets.DOCKER_PASSWORD }}

- name: Отправка образов в Docker Hub
  run: |
    docker push $BACKEND_IMAGE
    docker push $FRONTEND_IMAGE
    docker push $DATABASE_IMAGE
```

### 4. Особенности релизного workflow

Релизный workflow имеет следующие отличия от основного:
- Запускается только при создании Git-тега с версией (v*.*.*)
- Создает образы с тегами версии и latest
- Автоматически создает Release в GitHub с описанием изменений
- Пример команды для создания релиза:
  ```
  git tag v1.0.0
  git push origin v1.0.0
  ```

## Настройка для использования

1. Добавьте секреты в настройках репозитория GitHub:
   - `DOCKER_USERNAME` - имя пользователя на Docker Hub
   - `DOCKER_PASSWORD` - пароль или токен для Docker Hub

2. Убедитесь, что в бэкенде есть эндпоинт `/api/health` для проверки работоспособности

3. Запустите workflow одним из способов:
   - Внесите изменения и выполните push в ветку main (основной workflow)
   - Создайте pull request в ветку main (основной workflow)
   - Создайте и опубликуйте тег версии (релизный workflow)

## Преимущества настроенного CI/CD

1. **Автоматизация** - весь процесс от сборки до тестирования и публикации образов выполняется автоматически
2. **Быстрая обратная связь** - разработчики быстро узнают о проблемах с их изменениями
3. **Стандартизация процесса** - каждый релиз проходит одинаковые шаги проверки
4. **Экономия времени** - ручные операции по сборке и тестированию минимизированы
5. **Версионирование** - отдельный workflow для создания релизов с тегами версий 