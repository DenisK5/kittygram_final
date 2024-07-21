# Kittygram

[![Main Kittygram workflow](https://github.com/denisk5/kittygram_final/actions/workflows/main.yml/badge.svg)](https://github.com/denisk5/kittygram_final/actions/workflows/main.yml)
[![Website](https://img.shields.io/website?url=https%3A%2F%2Fkitty01.zapto.org%2F&label=kitty01.zapto.org&link=https%3A%2F%2Fkitty01.zapto.org%2F)](https://kitty01.zapto.org/)

## Описание
- Проект Kittygram позволяет пользователям делиться  фотографиями котиков и просматривать фотографии котиков других пользователей.
- При загрузке фото котика, пользователь должен ввести имя котика, год его рождения. Также можно указать цвет котика и его достижения.
- Добавлять и просматривать фотографии котиков могут только зарегистрированные и авторизованные пользователи.
- Только авторы могут изменять фотографии своих котиков и описание.

## Различия между продакшн и обычной версиями

### Обычная версия (Development)

- **Цель**: Используется для разработки и тестирования. Подходит для работы на локальной машине разработчика.
- **Конфигурация**:
  - `DEBUG=True` — включает режим отладки, который позволяет выводить подробные сообщения об ошибках и перезагружать сервер при изменении кода.
  - `ALLOWED_HOSTS` ограничены только локальными адресами (`127.0.0.1`, `localhost`).
  - Не настроена SSL-сертификация.

### Продакшн версия

- **Цель**: Используется для развертывания на сервере в боевой среде. Подходит для публичного доступа и эксплуатации.
- **Конфигурация**:
  - `DEBUG=False` — отключает режим отладки для повышения безопасности и производительности.
  - `ALLOWED_HOSTS` включает все публичные и частные адреса, где приложение будет доступно.
  - Настроена SSL-сертификация для безопасного HTTPS-соединения.

**Когда использовать**:
- **Обычная версия**: во время разработки и тестирования на локальной машине.
- **Продакшн версия**: при развертывании приложения на удаленном сервере для использования конечными пользователями.

## Локальное развертывание

Клонировать репозиторий:
```shell
git clone <https or SSH URL>
```
Перейти в каталог проекта:
```shell
cd kittygram_final
```

Создать .env файл со следующим содержанием:
```shell
# DB
POSTGRES_USER=<user>
POSTGRES_PASSWORD=<password>
POSTGRES_DB=<db name>
DB_HOST=db
DB_PORT=5432

# Django settings
SECRET_KEY=<django secret key>
DEBUG=False
ALLOWED_HOSTS=127.0.0.1;localhost;<your_host;xxx.xxx.xxx.xxx>
```
Развернуть приложение:
```shell
sudo docker compose -f docker-compose.dev.yml up
```
После успешного запуска, проект доступен на локальном IP `127.0.0.1:9000`.

## Развертывание на удаленном сервере
Для развертывания на удаленном сервере необходимо клонировать репозиторий на локальную машину. Подготовить и загрузить образы на Docker Hub.

Клонировать репозиторий:
```shell
git clone <https or SSH URL>
```
Перейти в каталог проекта:
```shell
cd kittygram_final
```

Создать docker images образы:
```shell
sudo docker build -t <username>/kittygram_backend backend/
sudo docker build -t <username>/kittygram_frontend frontend/
sudo docker build -t <username>/kittygram_gateway nginx/
```
Загрузить образы на Docker Hub:
```shell
sudo docker push <username>/kittygram_backend
sudo docker push <username>/kittygram_frontend
sudo docker push <username>/kittygram_gateway
```
Создать .env файл со следующим содержанием:
```shell
# DB
POSTGRES_USER=<user>
POSTGRES_PASSWORD=<password>
POSTGRES_DB=<db name>
DB_HOST=db
DB_PORT=5432

# Django settings
SECRET_KEY=<django secret key>
DEBUG=False
ALLOWED_HOSTS=127.0.0.1;localhost;<your_host;xxx.xxx.xxx.xxx>
```

Зайти на сервер:
```shell
ssh -i путь_до_файла_с_SSH_ключом/название_файла_закрытого_SSH-ключа login@ip
```
Создать директорию проекта и перейти в неё:
```shell
mkdir kittygram
```
```shell
cd kittygram
```
Перенести на удаленный сервер файлы `.env` `docker-compose.production.yml`:
```shell
scp .env docker-compose.production.yml <user@server-address>:/home/<user name>/kittygram
```
Выполнить сборку приложений:
```shell
sudo docker compose -f docker-compose.production.yml up -d
```
Выполнить миграции:
```shell
sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
```
Собрать статику:
```shell
sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
```
Скопировать статику:
```shell
sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. web/backend_static/static
```
Настроить шлюз для перенаправления запросов на `9000` порт, который слушает контейнер `kittygram_gateway`
```shell
sudo  nano /etc/nginx/sites-enabled/default
```
Пример конфигурации Nginx:
```shell
server {
    server_name xxx.xxx.xxx.xxx your_host;


    location / {
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:9000;
    }

    location /media/ {
        root /var/www/kittygram/;
    }


    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/your_host/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your_host/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

}

server {
    if ($host = your_host) {
        return 301 https://$host$request_uri;
    }



    listen 80;
    server_name xxx.xxx.xxx.xxx your_host;
    return 404;

}

```

Перезапустить nginx:
```shell
sudo systemctl restart nginx.service
```

## Технологии
- Python
- Docker
- PostgreSQL
- Gunicorn
- Nginx
- React

## Автор проекта

Проект разработан студентом 81 когорты Денисом Кондрашовым. 

[![DenisK5](https://img.shields.io/badge/GitHub-DenisK5-blue)](https://github.com/DenisK5)