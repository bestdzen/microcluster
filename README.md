Пример создания тестовой среды для разработки на WordPress
reverse proxy с балансировкой нагрузки,  в контейнерах Docker.

Запуск:
docker-compose --compatibility --env-file .env -f docker-compose.pub.yml up -d
Останов:
docker-compose --compatibility --env-file .env -f docker-compose.pub.yml down 

Образы:
Nginx (proxy):   1.23.1
Nginx (backend): 1.23.1
MariaDB:         10.6.11
WordPress:       6.1.1-php8.2-fpm
Redis:           5.0.7-alpine          
phpMyAdmin:      Latest

Opcache - их главная задача — единожды скомпилировать каждый PHP-скрипт 
и закэшировать получившиеся опкоды в общую память, чтобы их мог считать и
выполнить каждый рабочий процесс PHP

Redis — сетевое журналируемое хранилище данных типа "ключ" — "значение" с открытым исходным кодом.
При загрузке страницы необходимый SQL-запрос извлекается из памяти Redis; благодаря этому база данных не 
перегружается дублируемыми запросами. В результате страница загружается значительно быстрее, а влияние
сервера на ресурсы базы данных уменьшается. Если поступивший запрос не обнаружен в памяти Redis,
он извлекается из базы данных, после чего добавляется в кэш-память Redis.

                                             
                                         |----------------|
                                         | Nginx balancer |
                                         | ssl path-throw | 
                                         |----------------|
                                                 / \ Балансировка нагрузки на web серверы (round-robin)
                                  |--------------------------------|            
                                  | Nginx backend-1 Nginx backend-2| 
                                  |--------------------------------|
                                                / \                  \
                                               /   \                  \
                   Балансировка нагрузки для FastCGI (round-robin )   
           |-------------------------------------------------------|   |----------| 
           |------- wordpress (php-fpm) wordpress (php-fpm)--------|   |phpMyAdmin|
           |         Redis Object Cache    Redis Object Cache      |   |----------|   
           |--------php opcache --------php opcache  --------------|        |
                    Кеш Redis для Wordpress   |                             |
                              /               |                             |
                         |-------|        |---------|                       |
                         | Redis |        | MariaDB |-----------------------|
                         |-------|        |---------|
                         
                                                   
