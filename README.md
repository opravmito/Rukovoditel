# Описание проекта

CRM Руководитель является бесплатным приложением - **конструктором баз данных** с открытым кодом, предназначенным для установки на собственный локальный сервер или интернет сервер с поддержкой PHP/MySQL. Основатель проекта Сергей Харчишин.

Проект разделен на две части: основную (бесплатную) и [дополнение](https://www.rukovoditel.net.ru/extension.php). В основной части вы найдете набор инструментов для конструирования и настройки вашего приложения.

В [Дополнение ](https://www.rukovoditel.net.ru/extension.php)входит набор отчетов и инструментов для более эффективного планирования и управления.

[Дополнение ](https://www.rukovoditel.net.ru/extension.php) является платным расширением проекта и не входит в данный образ.

# Ресурсы приложения

* **Официальный сайт:** [Rukovoditel.net.ru](https://www.rukovoditel.net.ru/)
* **Документация:** [Официальная документация CRM Руководитель](https://docs.rukovoditel.net.ru/)
* **Ответы на вопросы:** [Официальный форум](https://forum.rukovoditel.net.ru)
* **Группа в Telegram:** [Неофициальное сообщество CRM Руководитель](https://t.me/crm_rukovoditel)
* **Репозиторий GitHub** [Неофициальный CRM Руководитель в Docker](https://github.com/registriren/Rukovoditel)

# Поддерживаемые версии приложения в Dockerfile

* [3.4.2 latest](https://github.com/registriren/Rukovoditel/blob/master/3.4.2/Dockerfile)

# Как использовать образ

1. Создаем общую сеть для базы данных и приложения Руководитель.
2. Создаем контейнер базы данных .
3. Создаем контейнер приложения (веб-сервер) Руководитель.
4. Осуществляем первичную настройку веб-сервера Руководитель, создаём связь с базой данных.
5. Удаляем инсталлятор (каталог `install`) из тома на котором развернуто приложение Руководитель.
6. Организуем резервирование данных.

## Создание сети:

Прежде чем создавать какие-либо контейнеры, вам понадобится общая сеть для взаимодействия контейнеров веб-сервера и базы данных.

Создайте локальную Docker сеть:

```
docker network create RukovoditelNET
```

## Создание контейнера базы данных:

Для управления данными вам нужна база данных, а для хранения данных вам необходимо постоянное хранилище. В данном случае это MariaDB и Docker том. Вы можете выбрать другую базу данных, например MySQL, или хранить данные в папке хост-машины.

Создайте контейнер базы данных MariaDB и Docker том RukovoditelDB для хранения данных:

```
docker run -d -p 3306:3306 --rm --name mariadb --network RukovoditelNET -v RukovoditelDB:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=secret -e MYSQL_USER=ruser -e MYSQL_PASSWORD=secret -e MYSQL_DATABASE=rukovoditel mariadb
```

Значение переменных `MYSQL_ROOT_PASSWORD=secret, MYSQL_USER=ruser, MYSQL_PASSWORD=secret, MYSQL_DATABASE=rukovoditel` можно задать самостоятельно. В дальнейшем они вам понадобятся при первичной настройке веб-сервера.

## Создание контейнера веб-сервера Руководитель:

Наконец, когда сеть и база данных будут готовы, вы можете запустить контейнер приложения Руководитель.

Создайте контейнер Руководитель, том Docker RukovoditelWEB для хранения данных и подключите его к сети:

```
docker run -dit --rm --name rukovoditel --network RukovoditelNET -v RukovoditelWEB:/var/www/html -p 8008:80 registriren/rukovoditel
```
Значение порта `8008` необходимо задать самостоятельно с учетом функционирующих сервисов на вашем сервере.

## Запуск и первичная настройка веб-сервера Руководитель:

Введите в браузере доменное имя или IP-адрес вашего сервера, на котором развёрнуто приложение Руководитель, например `http://localhost:8080`

Заполните поля для подключения к базе данных используя сведения заданные в переменных `MYSQL_USER=ruser, MYSQL_PASSWORD=secret, MYSQL_DATABASE=rukovoditel`

Введите данные администратора. Готово! Вы можете начать конструировать свое приложение - базу данных.

## Удаление инсталлятора:

Удалите каталог `install` ПОСЛЕ установки и первичной настройки веб-сервера Руководитель:

```
docker exec rukovoditel /bin/rm -r /var/www/html/install
```

## Резервирование и восстановление данных:

### Создание дампа базы данных

Большинство обычных инструментов будут работать, хотя в некоторых случаях их использование может быть немного запутанным, чтобы обеспечить им доступ к серверу mysqld. Простой способ убедиться в этом - использовать docker exec и запустить инструмент из того же контейнера:

```
docker exec mariadb sh -c 'exec mysqldump -user USERNAME --password --lock-tables --databases DATABASENAME' > /your/path/on/your/host/rukovoditel.sql
```

### Восстановление данных из дампа базы данных

Для восстановления данных вы можете использовать команду docker exec с флагом -i

```
docker exec -i mariadb sh -c 'exec mysql -user USERNAME --password' < /your/path/on/your/host/rukovoditel.sql
```

# Использование docker-compose:

Установку веб-приложения Руководитель можно осуществить и с помощью docker-compose:

```
version: '3'
services:
        rukovoditel:
                image: registriren/rukovoditel:latest
                volumes:
                        - RukovoditelWEB:/var/www/html
                networks:
                        - RukovoditelNET
                ports:
                        - "8008:80"
        mariadb:
                image: mariadb
                volumes:
                        - RukovoditelDB:/var/lib/mysql
                networks:
                        - RukovoditelNET
                ports:
                        - "3306:3306"
                environment:
                        - MYSQL_ROOT_PASSWORD=secret
                        - MYSQL_USER=ruser
                        - MYSQL_PASSWORD=secret
                        - MYSQL_DATABASE=rukovoditel
volumes:
        RukovoditelWEB:
        RukovoditelDB:
networks:
        RukovoditelNET:
```

# License

[Rukovoditel](https://www.rukovoditel.net/download.php) is open source and released under the terms of the [GNU General Public License v2 (GPL).](https://www.gnu.org/licenses/old-licenses/gpl-2.0.html) [Quick GPL-2.0 Summary.](https://tldrlegal.com/license/gnu-general-public-license-v2)

Main version of [Rukovoditel](https://www.rukovoditel.net/download.php) is free and fully functional software without any restrictions. [Extension](https://www.rukovoditel.net/extension.php) is commercial product and it is not included in this image.

As with all Docker images, these likely also contain other software which may be under other licenses (such as Bash, etc from the base distribution, along with any direct or indirect dependencies of the primary software being contained).

As for any pre-built image usage, it is the image user's responsibility to ensure that any use of this image complies with any relevant licenses for all software contained within.
