# my-docker-site

Учебный проект: свой сайт в Docker containers.

## Шаг 0. Подготовка
- Docker Engine из официального apt-репозитория, docker работает без sudo (пользователь в группе docker).
- Docker Hub напрямую не отвечал (TLS handshake timeout), настроил зеркало mirror.gcr.io в /etc/docker/daemon.json.
- Git и SSH-ключ, репозиторий склонирован.

## Шаг 1. Первый container
- docker run -d -p 8080:80 --name web nginx
- Открыл http://192.168.1.76:8080 в браузере Windows: VM в режиме Bridged, поэтому Windows её видит.
- docker logs web: запрос с VM пришёл с 172.17.0.1, из браузера с 192.168.1.30.
- docker exec -it web bash: страница лежит в /usr/share/nginx/html.
- 8080:80 — слева порт на VM, справа порт внутри container.

## Шаг 2. Своя страница
- docker run -d -p 8080:80 --name web -v ~/my-docker-site/site:/usr/share/nginx/html nginx
- Правка видна сразу: -v подключает папку с VM в container без копирования, это один и тот же файл, а nginx читает его при каждом запросе.

## Шаг 3. Свой image
- Dockerfile: FROM nginx:alpine и COPY site/ /usr/share/nginx/html/
- my-site:v1 на порту 8080 и my-site:v2 на порту 8081 работают одновременно.
- Страница скопирована внутрь image, поэтому правка видна только после новой сборки.
- nginx:latest — 66,7 MB при скачивании, наш image на nginx:alpine — 26,3 MB. Alpine — минимальный Linux, а обычный nginx построен на Debian.

## Шаг 4. Связка из двух сервисов
- docker network create mynet; httpbin и web4 в этой сети: docker exec -it web4 curl http://httpbin/get работает по имени.
- С самой VM curl http://httpbin/get даёт «Could not resolve host»: VM не в сети mynet.
- httpbin2 и web5 в стандартной сети bridge: по имени ошибка, по IP работает. В стандартной сети нет DNS.
- После перезапуска VM адрес httpbin2 сменился с 172.17.0.4 на 172.17.0.2.
- По имени надёжнее: имя постоянное, а IP может измениться.

## Шаг 5. docker compose
- docker-compose.yml: web (свой image), api (httpbin), db (postgres:16), общая сеть app-net.
- docker compose up -d поднимает всё примерно за секунду, web находит api по имени.
- Пароль postgres лежит в .env, а .env в .gitignore, поэтому в git он не попадает.
- Данные базы в volume db-data: после down и up строка в таблице на месте, после down -v данных нет.
