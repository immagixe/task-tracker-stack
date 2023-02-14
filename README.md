# TaskTracker

Многопользовательский планировщик задач. Пользователи могут использовать его в качестве TODO листа. Источником вдохновения для проекта является Trello.

## Использованные технологии

- Java, Kotlin
- Maven
- Spring Boot, Spring Security, Spring Session, Spring AMQP, Spring Scheduler, Spring Mail
- PostgreSQL, Spring Data JPA
- Redis (хранение сессий)
- Docker - контейнеры, образы, volumes, написание Dockerfile, Docker Compose
- Брокер сообщений RabbitMQ - очереди
- CI/CD, GitHub Actions

При разработке проекта использовался микросервисный подход. Проект включает в себя 3 микросервиса:

### Backend

Spring Boot (Java) приложение, реализующее REST API для работы с пользователями, сессиями, создаваемыми при авторизации, и задачами.

Работа с пользователями:

- Регистрация
- Авторизация
- Logout

Работа с задачами:

- Создание, редактирование (каждая задача имеет заголовок и описание)
- Пометка задачи как сделанной
- Удаление

### Рассыльщик писем

Spring Boot (Kotlin) приложение с двумя модулями - Spring Mail и Spring AMQP.

С помощью Spring AMQP приложение подключается к RabbitMQ и создает очередь сообщений, после чего подписывается на сообщения, поступающие от планировщика и бэкенд-сервиса.

Для каждого полученного сообщения, содержимое которого десериализуется в экземпляр модели, используется модуль Spring Mail, чтобы отправить письмо на электронный адрес пользователя.

### Планировщик

Spring Boot (Kotlin) приложение с двумя модулями - Spring Scheduler и Spring AMQP.

Задача сервиса - раз в сутки итерировать всех пользователей, формировать для них отчёты о задачах и изменениях в них за сутки, а также формировать email для отправки. Сформированные сообщения отправляются в RabbitMQ очередь.

## CI/CD
В рамках проекта с помощью GitHub Actions была автоматизирована сборка docker образов, загрузка образов в публичный репозиторий https://hub.docker.com/, а также деплой (docker compose) образов на арендованный удаленный сервер. Деплой (CD) происходит автоматически после выполнения процесса сборки и отправки образов в публичный репозиторий (CI).

## Инструкция по запуску проекта на локальном компьютере

- Установить Docker на локальную машину — https://docs.docker.com/get-docker/
- Склонировать репозиторий task-tracker-stack

```bash
git clone https://github.com/immagixe/task-tracker-stack.git
```

- В .env файл вставить свои данные для настройки отправки сообщений по протоколу SMTP. Можно использовать SMTP-сервис Яндекса https://yandex.ru/support/mail/mail-clients/others.html#smtpsetting

```bash
# Образец сконфигурированного .env файла
SPRING_MAIL_PROTOCOL=smtps
SPRING_MAIL_HOST=smtp.yandex.ru
SPRING_MAIL_PORT=465
SPRING_MAIL_USERNAME=username
SPRING_MAIL_PASSWORD=password
SENDER_ADDRESS=username@yandex.ru
```
- Запустить приложение

```bash
docker-compose up
```

- Страница с документацией будет доступна по ссылке: http://localhost:8080/swagger-ui/index.html
