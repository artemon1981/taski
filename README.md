# Taski


## Описание проекта.
Небольшой проект, в котором можно создавать собственные задачи и отслеживать их выполнение.


## Использованные технологии.
 - Python 3.9
 - Django 2.3
 - djangorestframework 3.12.4
 - Nginx 
 - gunicorn 20.1.0
 - Docker
 - Github CI/CD


# Концепция проекта
На данном проекте была осуществленна проработка приложения `Docker`, проект был помещен в `Docker-образы` для дальнейшего размещения на сервере, а также были созданы `Github-actions` для проверки и перезапуска собранных контейнеров. 


# Процедура разворачивания проекта:


## Установка проекта на локальный компьютер.

 - Клонируйте репозиторий
   ```
   git clone <адрес вашего репозитория>
   ```
 - Перейдите в директорию с клонированным репозиторием
   ```
   cd <название репозитория>
   ```
 - Установите виртуальное окружение
   ```
   python3 -m venv venv
   ```
 - Установите зависимости находясь в директории с текстовым файлом
   ```
   pip install -r requirements.txt
   ```

## Подключение сервера к аккаунту на GitHub

- Установите Git на сервер:
  ```
  sudo apt install git
  ```
- Находясь на сервере, сгенерируйте пару SSH-ключей командой
  ```
  ssh-keygen
  ```
- Сохраните открытый ключ в вашем аккаунте на GitHub:
  ```
  cat .ssh/id_rsa.pub
  ```
- Скопируйте ключ от символов ssh-rsa и добавьте его к вашему аккаунту на GitHub.
- Клонируйте проект с GitHub на сервер
  ```
  git clone git@github.com:Ваш_аккаунт/<Имя проекта>.git
  ```

## Запуск backend-части проекта на сервере.

- Установите пакетный менеджер и утилиту для создания виртуального окружения
  ```
  sudo apt install python3-pip python3-venv -y
  ```
- Находясь в директории с проектом, создайте и активируйте виртуальное окружение
  ```
  python3 -m venv venv
  ```
  ```
  source venv/bin/activate
  ```
- Установить зависимости находясь в директории с файлом .txt
  ```
  pip install -r requirements.txt
  ```
- Выполните все миграции
  ```
  python manage.py migrate
  ```
- Создайте суперпользователя
  ```
  python manage.py createsuperuser
  ```
- Отредактируйте файл settings.py
  ```
  ALLOWED_HOSTS = ['<внешний адрес вашего сервера>', '127.0.0.1', 'localhost']
  ```

## Запуск frontend-части проекта на сервере.

- Установите на сервер `Node.js` следующим образом:
```
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\
```
```
sudo apt-get install -y nodejs
```
- Установите зависимости frontend приложения выполнив команду 
  ```
  npm i
  ```
  из директории <ваш_проект>/frontend

## Установка и запуск Gunicorn

- При активированном виртуальном окружении установите пакет gunicorn
  ```
  pip install gunicorn==20.1.0
  ```
- Откройте файл _settings.py_ проекта и установите для константы `DEBUG` значение `False`
```
`DEBUG = False`
```
- В директории _/etc/systemd/system/_ создайте файл _gunicorn.service_
  ```
  sudo nano /etc/systemd/system/gunicorn.service
  ```
- Внести в файл следующие изменения:

      [Unit]
    
	    Description=gunicorn daemon
    
	    After=network.target
    
	    [Service]
    
	    User=yc-user
    
	    WorkingDirectory=/home/<имя пользователя в системе>/<имя проекта>/backend/
    
	    ExecStart=/home/<имя пользователя в системе>/<имя проекта>/venv/bin/gunicorn --bind 0.0.0.0:8080 kittygram_backend.wsgi
    
	    [Install]
    
	    WantedBy=multi-user.target

## Сбор статики для frontend приложения.

- Перейдите в директорию _/<имя_проекта>/frontend/_  и выполните команду
  ```
  npm run build
  ```
  
- В системную директорию сервера _/var/www/_ скопируйте содержимое папки _/frontend/build/_.

## Сбор статики для backend приложения.

- В файле _settings.py_ добавьте следующие настройки:

      STATIC_URL = 'static_backend'
	  STATIC_ROOT = BASE_DIR / 'static_backend'

- Активируйте виртуальное окружение, перейти в директорию с файлом _manage.py_ и выполните команду
  ```
  python manage.py collectstatic
  ``` 
- Скопировать директорию _static_backend/_ в директорию _/var/www/<имя_проекта>/_


## Установка и настройка Nginx

- На сервере из любой директории выполнить команду
  ```
  sudo apt install nginx -y
  ```
- Ограничьте порты выполнив по очереди команды
  ```
  sudo ufw allow 'Nginx Full
  ```
  ```
  sudo ufw allow OpenSSH
  ```
- Включить файервол
  ```
  sudo ufw enable
  ```
- Отредактируйте файл конфигурации веб-сервера
  ```
  sudo nano /etc/nginx/sites-enabled/default
  ```
  следующим образом:

      server {
    
	        listen 80;
	        server_name публичный_ip_вашего_удаленного_сервера;
    
	        location /api/ {
	            proxy_pass http://127.0.0.1:8080;
	        }
	        
		      location /admin/ {
			    proxy_pass http://127.0.0.1:8000;
				  }
			
	        location / {
	            root   /var/www/<имя_проекта>;
	            index  index.html index.htm;
	            try_files $uri /index.html;
	        }
      }
- Сохраните изменения и перезагрузите конфигурацию веб-сервера
  ```
  sudo nginx -t
  ```
  ```
  sudo systemctl reload nginx
  ```

## Добавление доменного имени.

- В файл _settings.py_ добавьте в список `ALLOWED_HOSTS` доменное имя: 

      ALLOWED_HOSTS = ['ip_адрес_вашего_сервера', '127.0.0.1', 'localhost', 'ваш-домен']
  
- Сохраните изменения и перезапустите gunicorn
  ```
  sudo systemctl restart gunicorn
  ```
- Внесите изменения в конфигурацию Nginx
  ```
  sudo nano /etc/nginx/sites-enabled/default
  ```
- Добавьте в строку `server_name` доменное имя:

		server {
		...
		    server_name <ваш-ip> <ваш-домен>;
		...
		}

- Проверьте конфигурацию
  ```
  sudo nginx -t
  ```
  и перезагрузите её командой
  ```
  sudo systemctl reload nginx
  ```

 ## Получение и настройка SSL-сертификата
 
 - Зайдите на сервер и последовательно выполните команды
  ```
  sudo apt install snapd
  ```
  ```
  sudo snap install core
  ```
  ```
  sudo snap refresh core
  ```
  ```
  sudo snap install --classic certbot
  ```
  ```
  sudo ln -s /snap/bin/certbot /usr/bin/certbot
  ```
	    
	   
	  
   
- Запустите certbot и получить SSL-сертификат:
  ```
  sudo certbot --nginx
  ```

- Перезагрузите конфигурацию Nginx
  ```
  sudo systemctl reload nginx
  ```
