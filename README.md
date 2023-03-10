
## Балансировщик нагрузки NGINX для WordPress. Docker-compose

```
Были использованы следующие образы:
* Nginx (proxy): 1.23.1
* Nginx (2 backends): 1.23.1
* MariaDB: 10.6.11
* WordPress (2 app): 6.1.1-php8.2-fpm
* Redis: 5.0.7-alpine
* phpMyAdmin: Latest
```

**Redis** — сетевое журналируемое хранилище данных типа "ключ" — "значение" с открытым исходным кодом.
При загрузке страницы необходимый **`SQL`**-запрос извлекается из памяти **`Redis`**; благодаря этому база данных не 
перегружается дублируемыми запросами. В результате страница загружается значительно быстрее, а влияние
сервера на ресурсы базы данных уменьшается. Если поступивший запрос не обнаружен в памяти **`Redis`**,
он извлекается из базы данных, после чего добавляется в кэш-память **`Redis`**.

**Opcache** - его главная задача — единожды скомпилировать каждый **`PHP-скрипт`** 
и закешировать получившиеся опкоды в общую память, чтобы их мог считать и
выполнить каждый рабочий процесс **`PHP`**.

## Как работает балансировщик ?
Создается безопасный маршрут от браузера клиента, проходящий через балансировщик нагрузки, до внутреннего сервера. В **`Nginx`** использована директива **`HTTP`** для **`HTTP`** запросов и директива **`stream`** для **`SSL`** запросов. Директива **`stream`** фактически позволяет внутреннему серверу завершать входящие соединения и в то же время балансировать нагрузку. Настроен редирект с **`http://example.com`** (или  **`http://www.example.com`** ) на  **`https://example.com`**, c **`public ip`** на **`https://example.com`**. В  демонстрационных целях настроено использовани самоподписанного сертификата **`SSL`**. 

## Схема работы балансировщика
```mermaid
graph TD;
    Nginx[Nginx proxy balancer]-->Nginx-backend1;
    Nginx-->Nginx-backend2;
    Nginx-backend1[Nginx-backend1+FastCGI cache]-->WordPress-php-fpm-1[WordPress-php-fpm-1 + opcache];
    Nginx-backend2[Nginx-backend2+FastCGI cache]-->WordPress-php-fpm-1;
    Nginx-backend1-->WordPress-php-fpm-2[WordPress-php-fpm-2 + opcache];;
    Nginx-backend2-->WordPress-php-fpm-2;
    WordPress-php-fpm-1-->Redis;    
    WordPress-php-fpm-1-->MariaDB;
    WordPress-php-fpm-2-->Redis;    
    WordPress-php-fpm-2-->MariaDB;
    phpMyAdmin-->MariaDB;
```
## Основные настройки для проксирования 
1. Включена опция **`proxy_protocol`** на балансировщике.
2. Добавлена опция **`real_ip_header proxy_protocol`** и **`set_real_ip_from proxy`** на бэкэнд веб серверах.
3. Установлена опция **`proxy_protocol`** в директиве **`listen`** на бэкэндах.

## Директива set_real_ip_from *proxy* на бэкэндах
 Директива **`set_real_ip_from`** со значением **`proxy`**. proxy - это имя хоста прокси-сервиса, которое благодаря внутреннему механизму **`DNS`** **`Docker`** преобразуется в IP-адрес. Таким образом, мы можем гарантировать, что даже при перезапуске балансировщика нагрузки внутренние серверы получат правильный IP (**`docker`** делает это сам и правильно). Наконец, стоит упомянуть, что протокол прокси был разработан для создания цепочки обратных прокси без потери информации о клиенте.

## Порядок установки:

**Для корректной работы измените переменную окружения PUBLIC_IP в файле .env на IP адрес вашего компьютера:**
- Добавьте в файл **`hosts`** имя домена **`example.com`**, а также **`www.example.com`** и соответствующий ему IP адрес:
- <IP адрес вашего компьютера> **`example.com`**
- <IP адрес вашего компьютера> **`www.example.com`**

### Установка Docker на Ubuntu и Debian
```bash
$ sudo apt-get update
$ sudo apt install docker-ce -y
```

### Установка Docker-compose на Ubuntu
```bash
$ sudo curl -L "https://github.com/docker/compose/releases/download/2.15.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
```

### Запуск контейнеров с помощью docker-compose
```bash
$ git clone https://github.com/bestdzen/microcluster
$ cd microcluster
$ docker-compose --compatibility --env-file .env -f docker-compose.pub.yml up -d
```

### Заходим на сервис:
* ```https://example.com/wp-admin/```

## Учетные данные
* Login: ```wordpressadmin```
* Passwsord: ```wordpressadminsupersecret```


