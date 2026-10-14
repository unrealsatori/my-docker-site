# my-docker-site

Учебный проект: свой сайт в Docker containers.

## Шаг 0. Подготовка
- Поставил Docker Engine, docker работает без sudo (пользователь в группе docker).
- Docker Hub напрямую не отвечал, настроил зеркало mirror.gcr.io в /etc/docker/daemon.json.
- Настроил git и SSH-ключ, склонировал репозиторий.

## Шаг 1. Первый container
- `docker run -d -p 8080:80 --name web nginx`
- Открыл http://192.168.1.76:8080 в браузере Windows: VM в режиме Bridged, поэтому Windows её видит.
- `docker logs web` — видны запросы от curl и от браузера.
- `docker exec -it web bash` — страница лежит в /usr/share/nginx/html.
- 8080:80 — слева порт на VM, справа порт внутри container.

## Шаг 2. Своя страница
- `docker run -d -p 8080:80 --name web -v ~/my-docker-site/site:/usr/share/nginx/html nginx`
- Правка index.html видна сразу, потому что - Правка index.html видна сразу, потому что -v подключает папку с VM в container без копирования: это один и тот же файл, а nginx читает его при каждом запросе. Поэтому пересборка не нужна.
