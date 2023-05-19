## Программа обучения Kubernetes

1.	Основные сущности. Изучение сущностей namespace, nod, pod. 
•	Теория:
Изучение спецификации. Операции с пространствами имён.
Изучение спецификации. Операции с подами. Жизненный цикл подов. Поиск и устранение проблем.
•	Практика:
Развёртывание первого приложения. Определение нахождения приложения в пространстве имен, в ноде и поде
2.	Управление ресурсами. 
Введение в управление ресурсами. Применяйте его для запросов и ограничений ЦП и памяти.
3.	ReplicaSets
Описание набора реплик ReplicaSets, их взаимосвязь с подами. Основные операции при работе с ReplicaSets.
4.	Deployment
Описание развертываний, их взаимосвязь с наборами реплик. Основные операции при работе с развертываниями, история развертываний, откат неудачных развертываний. Обзор стратегий развертывания.
5.	Service
Описание сервисов. Основные операции при работе с сервисами. Механизмы обнаружения. Сетевой взаимодействие в Kubernetes.
6.	Метки и аннотации
Использование меток для выборки объектов, связи объектов. Использование аннотаций для описания.
7.	Ingress
8.	Тома
Описание томов, их разновидности (emptyDir, hostPath и persistent volume claim). Статическое и динамическое выделение по заявкам.
9.	ConfigMaps
Описание конфигураций с помощью ConfigMaps. Установка и получение настроек через файлы и переменные окружения.

10.	Secrets
Использование Secrets для работы с конфиденциальной информацией. Установка и получение важных данных в открытом и закодированном виде посредством файлов и переменных окружения.
11.	Хранение приватных данных в Vault 

12.	жизненный цикл. 
Научимся вызвать hooks из приложений для того, чтобы разобраться, как Kubernetes запускает код, и посмотрим как закончить работу приложения изящно.
ДЗ и практика: работа с SIGTERM на примере Python, а также знакомство c liveness и readiness пробами

13.	Шаблоны: Helm и его аналоги (Jsonnet, Kustomize)
— Конфигурация Helm-пакета Prometheus-оператора
— Развёртывание пакета Prometheus-оператора
14.	Мониторинг компонентов кластера и приложений, работающих в нём
Научитесь мониторить компоненты кластера и приложений, работающих в нем. Построите свои собственные SLA/SLO.
15.	Cert manager
16.	Service mesh. Знакомство с Istio и Envoy
Научитесь использовать service mesh для повышения безопасности вашего кластера и визуализировать трейсы запросов с Jaeger.
17.	Kubernetes для непрерывной поставки (CI/CD). Интеграция с CI-сервисом
— Интеграция в GitLab (для чего это нужно и как работает)
— Примеры сложных пайплайнов
— Полезные технологии для CI/CD (kaliko и т. д.)
— Разбор готового конфига-примера
18.	Итоговый проект
Напишете полноценную инфраструктурную платформу на основе кластера Kubernetes. По ходу работы над проектом вы: настроите CI/CD-пайплайн в Gitlab CI, установите и настроите мониторинг в Grafana с SLO/SLA, настроите автоскейлинг подов, обеспечите безопасное хранение приватной информации ваших приложений в Vault, внедрите service mesh Istio и настроите mTLS для безопасного взаимодействия контейнеров, добавите open tracing на основе Jaeger

или

3 практические задачи:
Развёртывание простого веб-приложения в Kubernetes
Скачиваем код приложения из репозитория. А дальше:
— Используете конфиг-файлы deployment базы данных и приложения;
— Используете конфиг-файл сервиса для базы данных;
— Создадите подключение к развёрнутому приложению;
— Рассмотрите масштабирование и обеспечение качества.
Развёртывание приложения с базой данных, внешним key-value-хранилищем Redis и очередью сообщений
Усложнённая версия первого приложения с подключением бэкенда и баз данных. Здесь добавятся:
— Redis;
— RabbitMQ (очередь сообщений).
Мониторинг второго приложения
Реализуем финальные шаги и проводим мониторинг развёрнутого приложения:
— Настраиваем оператора перед установкой через менеджер пакетов Helm;
— Устанавливаем оператора (то есть развёртываем масштабируемую систему мониторинга, сбора метрик и их визуализации);
— Пишем конфиг-файлы для подключения приложения к системе мониторинга и убеждаемся в простоте интеграции приложений в Kubernetes.




DOCKER RUN
ПРОСТЫЕ ДЕЙСТВИЯ С КОНТЕЙНЕРАМИ
ЗАПУСК И ОСТАНОВКА КОНТЕЙНЕРОВ
ИНФОРМАЦИЯ О КОНТЕЙНЕРЕ
РАБОТА С КОНТЕЙНЕРАМИ
РАБОТА С REGISTRY
РАБОТА С ОБРАЗАМИ
РАБОТА С ТОМАМИ
СЕТИ
ЧИСТКА МУСОРА
DOCKER RUN
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
[ OPTIONS ]

-it — интерактивный режим. Перейти в контейнер и запустить внутри контейнера команду
-d — запустить контейнер в фоне (демоном) и вывести его ID
-p port_localhost:port_docker_image — порты из докера на локалхост
-e «TZ=Europe/Moscow» — указываем нашему контейнеру timezone
-h HOSTNAME — присвоить имя хоста контейнеру
— link <имя контейнера> — связать контейнеры с другим
-v /local/path:/container/path/ — прокидываем в контейнер докера директорию с локальной машины
--name CONTAINERNAME — присвоить имя нашему контейнеру
--restart=[no/on-failure/always/unless-stopped] — варианты перезапуска контейнера при крэше
Подробнее про опции запуска тут

# Для больше читабельности можно при переносе строки ставить символ \
docker run \ 
—restart=always -it \
ПРОСТЫЕ ДЕЙСТВИЯ С КОНТЕЙНЕРАМИ
Создание контейнера
docker create -t -i eon01/infinite --name <CONTAINERNAME or CONTAINERID>
Переименование контейнера
docker rename <OLD CONTAINERNAME> <NEW CONTAINERNAME>
Удаление контейнера
docker rename <OLD CONTAINERNAME> <NEW CONTAINERNAME>
Обновление контейнера
docker update --cpu-shares 512 -m 300M <CONTAINERNAME or CONTAINERID>
ЗАПУСК И ОСТАНОВКА КОНТЕЙНЕРОВ
Запуск остановленного контейнера
docker start <CONTAINERNAME>
Остановка
docker stop <CONTAINERNAME>
Перезагрузка
docker restart <CONTAINERNAME>
Пауза (приостановка всех процессов контейнера)
docker pause <CONTAINERNAME>
Снятие паузы
docker unpause <CONTAINERNAME>
Блокировка (до остановки контейнера)
docker wait <CONTAINERNAME>
Отправка SIGKILL (завершающего сигнала)
docker kill <CONTAINERNAME>
Отправка другого сигнала
docker kill -s HUP <CONTAINERNAME>
Подключение к существующему контейнеру
docker attach <CONTAINERNAME>
Выход через Ctrl+p+q

ИНФОРМАЦИЯ О КОНТЕЙНЕРЕ
Работающие контейнеры
# Вывести работающие контейнеры
docker ps

# Вывести все контейнеры
docker ps -a
Логи контейнера
docker logs <CONTAINERNAME or CONTAINERID>
Информация о контейнере
# Все параметры контейнера
docker inspect <CONTAINERNAME or CONTAINERID>

# Вывести величину конкретного параметра
docker inspect --format '{{ .NetworkSettings.IPAddress }}' $(docker ps -q)
События контейнера
docker events <CONTAINERNAME or CONTAINERID>
Публичные порты
docker port <CONTAINERNAME or CONTAINERID>
Выполняющиеся процессы
docker top <CONTAINERNAME or CONTAINERID>
Использование ресурсов
docker stats <CONTAINERNAME or CONTAINERID>
Изменения в файлах или директориях файловой системы контейнера
docker diff <CONTAINERNAME or CONTAINERID>
РАБОТА С КОНТЕЙНЕРАМИ
Зайти в уже запущенный контейнер.
(точнее выполнить команду внутри контейнера)

docker exec -it name_of_container /bin/bash
Запустить контейнер и открыть в нём bash
docker run -it -d --name my_container CONTAINER_ID /bin/bash
Копирование файлов внутрь контейнера.
docker cp some_files.conf docker_container:/home/docker/
При смене путей, можно копировать из контейнера.

РАБОТА С REGISTRY
Вход в реестр
# Вход на докерзаб
docker login

# Вход в WebUI локального registry
docker login localhost:8080
Выход из реестра
# Докерхаб
docker logout

# Локальный registry
docker logout localhost:8080
Поиск образа
# Простой поиск
docker search nginx

# Поиск с фильтрами
docker search nginx -- filter stars=3 --no-trunc busybox
Pull (выгрузка из реестра) образа
# Выгрузка из Докерхаба
docker pull nginx

# Из локального registry
docker pull eon01/nginx localhost:5000/myadmin/nginx
Push (загрузка в реестр) образа
# В докерхаб
docker push eon01/nginx

# В локальный registry
docker push eon01/nginx localhost:5000/myadmin/nginx
РАБОТА С ОБРАЗАМИ
Список образов
docker images
Создание образов
(файл опциями сборки образа), учитывая что мы находимся в папке где лежит этот файл. Через ключ -t назначаем имя нашему образу. Точка в конце означает что Dockerfile лежит в текущей директории.

docker build -t my_docker .
docker build .
docker build github.com/creack/docker-firefox
docker build - < Dockerfile
docker build - < context.tar.gz
docker build -t eon/infinite .
docker build -f myOtherDockerfile .
curl example.com/remote/Dockerfile | docker build -f - .
Удаление образа
docker rmi nginx
Загрузка репозитория в tar (из файла или стандартного ввода)
docker load < ubuntu.tar.gz
docker load --input ubuntu.tar
Сохранение образа в tar-архив
docker save busybox > ubuntu.tar
Просмотр истории образа
docker history
Создание образа из контейнера
docker commit nginx
Тегирование образа
docker tag nginx eon01/nginx
Push (загрузка в реестр) образа
docker push eon01/nginx
РАБОТА С ТОМАМИ
Немного теории:
Если монтируем пустой том с хоста, а в контейнере уже есть файлы, то они скопируются в том
Если монтируем с хоста том с файлами, то они окажутся в контейнере
Если мы монтируем не пустой том с хоста, а в контейнере по этому пути уже есть файлы, то они будут скрыты
Можно монтировать с хоста любые файлы. В том числе и служебные. Например сокет docker. Что бы получился docker-in-docker (dind)
Создание тома
docker volume create <VOLUME-NAME>
Монтируем с хоста в контейнер
docker run --mount source=<VOLUME-NAME>,target=/path/to/folder/in/container -d <IMAGE>
Монтируем с контейнера на хост
docker run --mount type=bind,source=/host/folder,target=/container/folder -d <IMAGE>
Посмотреть настройки тома
docker volumes inspect <VOLUME-NAME>
Вывести список всех томов с их названиями.
docker volume ls
Удаление volumes по названию
docker volume rm <VOLUME-NAME>
СЕТИ
Создание сети
docker network create -d overlay MyOverlayNetwork
docker network create -d bridge MyBridgeNetwork
docker network create -d overlay \
  --subnet=192.168.0.0/16 \
  --subnet=192.170.0.0/16 \
  --gateway=192.168.0.100 \
  --gateway=192.170.0.100 \
  --ip-range=192.168.1.0/24 \
  --aux-address="my-router=192.168.1.5" --aux-address="my-switch=192.168.1.6" \
  --aux-address="my-printer=192.170.1.5" --aux-address="my-nas=192.170.1.6" \
  MyOverlayNetwork
Удаление сети
docker network rm MyOverlayNetwork
Список сетей
docker network ls
Получение информации о сети
docker network inspect MyOverlayNetwork
Подключение работающего контейнера к сети
docker network connect MyOverlayNetwork nginx
Подключение контейнера к сети при его запуске
docker run -it -d --network=MyOverlayNetwork nginx
Отключение контейнера от сети
docker network disconnect MyOverlayNetwork nginx
ЧИСТКА МУСОРА
Показать образы и контейнеры
# Вывести все образы (images)
docker images -a

# Вывести все неиспользуемые образы
docker images -f dangling=true

# Выведет список запущенных контейнеров, если добавить ключ -a, выведет список всех контейнеров.
docker ps

Удаление образов (images)
# Для удаления используется команда docker rmi с добавлением ИД или тега, например:
# с ключом --force удалит контейнер и образ
docker rmi abb461727af5

# Удаление всех образов
docker rmi $(docker images -a -q)

# Удаление всех неиспользуемых образов
docker images prune
docker rmi $(docker images -f dangling=true -q)

# Удаление всех неиспользуемых (не связанных с контейнерами) образов:
# Если добавить к команде ключ -a, то произойдет удаление всех остановленных контейнеров и неиспользуемых образов.
docker system prune

# Удаление всех образов без тегов
docker rmi -f $(docker images | grep "^<none>" | awk "{print $3}")

# Удаление всех образов
docker rmi $(docker images -a -q)
Удаление контейнеров (containers)
# Для удаления контейнера, его необходимо сначала остановить командой ниже с указанием ID или названия контейнера.
docker stop CONTAINER_ID

# Для удаления контейнера используется команда docker rm с добавлением ИД или названия
docker rm CONTAINER_ID

# Удаление контейнера и его тома (volume)
docker rm -v CONTAINER_ID

# Удаление всех контейнеров со статусом exited
docker rm $(docker ps -a -f status=exited -q)

# Удаление всех остановленных контейнеров
docker container prune
docker rm `docker ps -a -q`

# Удаление контейнеров, остановленных более суток назад
docker container prune --filter "until=24h"

# Остановка и удаление всех контейнеров
docker stop $(docker ps -a -q) && docker rm $(docker ps -a -q)
Удаление томов (volumes)
#Вывести список всех томов с их названиями.
docker volume ls

# Удаление volumes по названию
docker rm <volume_name>

# Вывести список всех томов не связанных с контейнерами
docker volume ls -f dangling=true

# Удаление томов (volumes) несвязанных с контейнерами
docker volume prune
docker volume rm $(docker volume ls -f dangling=true -q)

# Удаление неиспользуемых (dangling) томов по фильтру
docker volume prune --filter "label!=keep"
Удаление сетей (networks)
# Вывести список всех сетей с их ИД и названиями. 
docker network ls

# Для удаления используется команда  с добавлением ИД или названия:
docker network rm NETWORK_ID

# Удалит все сети не используемые хотя бы одним контейнером.
docker network prune
Удаление всех неиспользуемых объектов
docker system prune

# По умолчанию для Docker 17.06.1+ тома не удаляются. Чтобы удалились и они тоже:
docker system prune --volumes
