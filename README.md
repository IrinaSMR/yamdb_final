## CI и CD проекта api_yamdb
***

![Django-app workflow](https://github.com/IrinaSMR/yamdb_final/actions/workflows/yamdb_workflow.yml/badge.svg)
***

### Стек технологий:

- Python 3.7
- DRF (Django REST framework)
- Django
- Docker
- Gunicorn
- nginx
- PostgreSQL
- GIT

### Описание проекта:

REST API для сервиса YaMDb, который собирает отзывы пользователей на произведения. Произведения делятся на категории: «Книги», «Фильмы», «Музыка». Список категорий может быть расширен.

### Workflow:

- tests - проверка кода на соответствие стандарту PEP8 (с помощью пакета flake8) и запуск pytest (дальнейшие шаги выполнятся, только если push был в ветку master или main);
- build_and_push_to_docker_hub - сборка и доставка докер-образов на Docker Hub;
- deploy - автоматический деплой проекта на боевой сервер (выполняется копирование файлов из репозитория на сервер);
- send_message - отправка уведомления о состоянии workflow в Telegram.

***
### Подготовка для запуска workflow:

Клонируйте репозиторий, перейдите в него в командной строке:

```
git clone https://github.com/IrinaSMR/yamdb_final.git
cd yamdb_final
```

Создайте и активируйте виртуальное окружение, обновите pip:

```
python -m venv venv или python3 -m venv venv для Linux
source venv/Scripts/activate или source venv/bin/activate для Linux
python -m pip install --upgrade pip
```

Запустите автоматическое тестирование:

```
pytest
```

В репозитории на GitHub добавьте данные в Settings - Secrets - Actions secrets:

```
DOCKER_USERNAME - имя пользователя в DockerHub
DOCKER_PASSWORD - пароль пользователя в DockerHub
HOST - ip-адрес сервера
USER - имя пользователя
SSH_KEY - приватный ssh-ключ (публичный должен быть на сервере)
PASSPHRASE - кодовая фраза для ssh-ключа (если использовалась при создании)
TELEGRAM_TO - id своего телеграм-аккаунта (можно узнать у @userinfobot, команда /start)
TELEGRAM_TOKEN - токен бота (получить токен можно у @BotFather, /token, имя бота)
```

При внесении любых изменений в файлы проекта после коммита и пуша:

```
git add .
git commit -m '...'
git push
```
запускается набор блока команд jobs (см. файл yamdb_workflow.yaml), поскольку команда git push является триггером workflow проекта.
***

### Как развернуть проект локально:

Клонируйте репозиторий и перейдите в него в командной строке:

```
git clone https://github.com/IrinaSMR/yamdb_final.git
cd yamdb_final
```
Создайте файл .env командой touch .env и добавьте в него переменные окружения для работы с базой данных.
Шаблон наполнения файла .env:

```
DB_ENGINE=django.db.backends.postgresql
DB_NAME=postgres # имя базы данных
POSTGRES_USER=postgres # логин для подключения к базе данных
POSTGRES_PASSWORD=postgres1 # пароль для подключения к БД (установите свой)
DB_HOST=db # название сервиса (контейнера)
DB_PORT=5432 # порт для подключения к БД
```

В папке проекта создайте образ:

```
docker build -t username/imagename:version api_yamdb/.
```

Соберите контейнеры:

```
docker-compose -f infra/docker-compose.yaml up -d --build
```

или пересоберите:

```
docker-compose up -d --build
```

Выполните миграции:

```
(опционально) docker-compose exec web python manage.py makemigrations
docker-compose exec web python manage.py migrate
```

Загрузите тестовые данные в БД:

```
docker-compose exec web python manage.py loaddata fixtures.json
```

или

```
docker-compose -f infra/docker-compose.yaml exec web python manage.py loaddata fixtures.json
```
***

### Как развернуть проект на сервере:

Установите соединение с сервером:

```
ssh username@server_address
```

Проверьте статус nginx:

```
sudo service nginx status
```

Если nginx запущен, остановите его:

```
sudo systemctl stop nginx
```

Установите Docker и Docker-compose:

```
sudo apt install docker.io
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

Проверьте корректность установки Docker-compose:

```
sudo  docker-compose --version
```

В файле nginx/default.conf в строке server_name укажите IP виртуальной машины (сервера).
Скопируйте подготовленные файлы docker-compose.yaml и nginx/default.conf из проекта на сервер:

```
scp docker-compose.yaml <username>@<host>:/home/<username>/docker-compose.yaml
mkdir nginx
scp default.conf <username>@<host>:nginx/default.conf
```

### После успешного деплоя:

Соберите файлы статики:

```
docker-compose exec web python manage.py collectstatic --no-input
```

Выполните миграции:

```
(опционально) docker-compose exec web python manage.py makemigrations
docker-compose exec web python manage.py migrate --noinput
```

Создайте суперпользователя:

```
docker-compose exec web python manage.py createsuperuser
```

При необходимости наполните базу тестовыми данными из ../yamdb_final/infra/:

```
docker-compose exec web python manage.py loaddata fixtures.json
```
***

### Проект был запущен и доступен по адресу:

http://yammydb.hopto.org/

## Документация API YaMDb

Документация доступна по эндпойнту: http://localhost/redoc/

## Author
- IrinaSMR
