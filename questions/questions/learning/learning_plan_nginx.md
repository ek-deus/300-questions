### Главное, что нужно понять: Nginx работает по асинхронной событийной модели (event-driven).
Есть один Master-процесс (читает конфиг, управляет воркерами, пишет логи) и несколько Worker-процессов (непосредственно обрабатывают запросы от клиентов). Воркеры не блокируют друг друга.
Иерархия блоков конфигурации
Конфиг Nginx похож на матрешку. Директивы наследуются сверху вниз:
main -> events -> http -> server -> location

Эталонный nginx.conf с построчным разбором

```conf
# ================= MAIN CONTEXT (Глобальные настройки) =================
user nginx;                           # От какого пользователя работать (безопасность)
worker_processes auto;                # Кол-во воркеров. 'auto' = кол-ву ядер CPU. 
                                      # Каждый воркер однопоточный.
pid /run/nginx.pid;                   # Где хранить PID мастер-процесса
error_log /var/log/nginx/error.log warn; # Лог ошибок и уровень логирования (debug, info, notice, warn, error, crit)

# ================= EVENTS CONTEXT (Настройки сетевого ядра) =================
events {
    worker_connections 1024;          # Максимум одновременных подключений на ОДИН воркер.
                                      # Формула макс. подключений: worker_processes * worker_connections.
    use epoll;                        # Метод опроса событий. epoll (Linux) - самый эффективный.
    multi_accept on;                  # Разрешить воркеру принимать сразу несколько соединений за раз.
}

# ================= HTTP CONTEXT (Настройки HTTP-сервера) =================
http {
    include /etc/nginx/mime.types;    # Подключаем файл с соответствиями расширений файлов и MIME-типов
    default_type application/octet-stream; # MIME-тип по умолчанию, если не найден в mime.types

    # --- Оптимизация передачи файлов ---
    sendfile on;                      # Передача файлов напрямую из ядра в сеть (минуя буферы user-space). Ускорение.
    tcp_nopush on;                    # Отправлять заголовки HTTP в одном пакете (работает только с sendfile).
    tcp_nodelay on;                   # Отключить алгоритм Нэгла (важно для keepalive соединений).

    # --- Таймауты и Keepalive ---
    keepalive_timeout 65;             # Сколько секунд держать открытым соединение с клиентом после ответа.
    keepalive_requests 100;           # Сколько запросов можно сделать через одно keepalive соединение.

    # --- Логирование ---
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
    access_log /var/log/nginx/access.log main; # Путь к логу и формат

    # --- Сжатие (Gzip) ---
    gzip on;                          # Включить сжатие
    gzip_vary on;                     # Добавлять заголовок Vary: Accept-Encoding (важно для кэшей/CDN)
    gzip_proxied any;                 # Сжимать ответы для проксируемых запросов
    gzip_comp_level 6;                # Уровень сжатия (1-9). 6 - баланс между CPU и размером.
    gzip_types text/plain text/css application/json application/javascript text/xml; # Что сжимать

    # --- Upstream (Бэкенды для балансировки) ---
    upstream backend_api {
        # Метод балансировки (по умолчанию round-robin)
        # least_conn;                # Альтернатива: отдавать запрос тому, у кого меньше активных соединений
        # ip_hash;                   # Альтернатива: привязка сессии к IP клиента
        
        server 10.0.0.2:8080 weight=5 max_fails=3 fail_timeout=30s; # weight - вес, max_fails - кол-во ошибок для признания мертвым
        server 10.0.0.3:8080 backup; # backup - используется только если упали основные
        server 10.0.0.4:8080 down;   # down - сервер выключен из ротации
    }

    # ================= SERVER CONTEXT (Виртуальные хосты) =================
    server {
        listen 80 default_server;     # Слушать порт 80. default_server - если не подошел ни один server_name
        listen 443 ssl http2;         # Слушать 443 с SSL и поддержкой HTTP/2
        
        server_name example.com www.example.com; # Доменные имена, на которые реагирует этот блок

        # SSL сертификаты
        ssl_certificate /etc/nginx/ssl/cert.pem;
        ssl_certificate_key /etc/nginx/ssl/key.pem;
        ssl_protocols TLSv1.2 TLSv1.3; # Только безопасные протоколы

        root /var/www/html;           # Корневая директория для статических файлов
        index index.html index.htm;   # Файлы, которые отдаются, если запрошена директория

        # --- LOCATION CONTEXT (Правила маршрутизации) ---
        
        # 1. Точное совпадение (Приоритет самый высокий)
        location = /favicon.ico {
            log_not_found off;
            access_log off;
        }

        # 2. Префиксное совпадение с приоритетом regex (Приоритет выше обычного префикса)
        location ^~ /images/ {
            # Если URI начинается с /images/, то regex ниже НЕ проверяются
            expires 30d;
        }

        # 3. Проксирование на бэкенд (Upstream)
        location /api/ {
            # ВАЖНО: слеш в конце proxy_pass!
            proxy_pass http://backend_api/; 
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # 4. Отдача статики
        location / {
            try_files $uri $uri/ /index.html; # Если файл не найден, отдать index.html (нужно для SPA)
        }
    }
}
```

