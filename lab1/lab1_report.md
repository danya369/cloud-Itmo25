# Отчёт по лабораторной работе №1

#### Файлик с теорией и моими записями (если это важно), под названием lab1_theory.md, находится в этой же папке.


## Ход выполнения

### 1. Подготовка окружения
- Установлен **Docker** и **Docker Compose**.  
- Создан локальный проект с файловой структурой:  

```
.
├── docker-compose.yml
├── nginx
│   └── nginx.conf
├── site1
│   └── index.html
├── site2
│   └── index.html
└── ssl
    ├── cert.pem
    └── key.pem
```

---

### 2. Генерация сертификата
Для HTTPS был создан самоподписанный сертификат:  

```bash
openssl req -x509 -nodes -days 365   -newkey rsa:2048   -keyout ssl/key.pem   -out ssl/cert.pem   -subj "/CN=site1.local"
```

- `cert.pem` — публичный сертификат.  
- `key.pem` — приватный ключ.  

---

### 3. Конфигурация docker-compose.yml
Файл **docker-compose.yml** для запуска контейнера:  

```yaml
services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./site1:/usr/share/nginx/html/site1
      - ./site2:/usr/share/nginx/html/site2
      - ./ssl:/etc/nginx/ssl
```

---

### 4. Конфигурация nginx
Файл **nginx.conf**:  

```nginx
events {}

http {
    # site1: redirect HTTP -> HTTPS
    server {
        listen 80;
        server_name site1.local;
        return 301 https://$host$request_uri;
    }

    # site1: HTTPS
    server {
        listen 443 ssl;
        server_name site1.local;

        ssl_certificate     /etc/nginx/ssl/cert.pem;
        ssl_certificate_key /etc/nginx/ssl/key.pem;

        location / {
            root /usr/share/nginx/html/site1;
            index index.html;
        }

        location /images/ {
            alias /usr/share/nginx/html/site1/folder_images/;
        }
    }

    # site2: redirect HTTP -> HTTPS
    server {
        listen 80;
        server_name site2.local;
        return 301 https://$host$request_uri;
    }

    # site2: HTTPS
    server {
        listen 443 ssl;
        server_name site2.local;

        ssl_certificate     /etc/nginx/ssl/cert.pem;
        ssl_certificate_key /etc/nginx/ssl/key.pem;

        location / {
            root /usr/share/nginx/html/site2;
            index index.html;
        }
    }
}
```

---

### 5. Настройка hosts
Чтобы использовать локальные домены, в файл `/etc/hosts` были добавлены строки:  

```
127.0.0.1 site1.local site2.local
```

---

### 6. Запуск контейнера
Команда для запуска nginx:  
```bash
docker compose up -d
```

---

## Результат
- Настроены два сайта (`site1.local` и `site2.local`) с поддержкой HTTPS.  
- Работает редирект с HTTP → HTTPS.  
- Настроен alias для примера сопоставления путей.  
- Проект упакован в контейнер с помощью Docker Compose.  

