## Программа обучения Gitlab

1. Что такое CI/CD и какие проблемы решает
Историческая справка
Необходимость автоматизации релизов, тестирования, их повторяемость
Ускорение разработки продукта
Унификация и отдельные ресурсы для сборки продукта
2. Общий принцип работы CI/CD
Конвейерный метод разработки
Пайплайны, билды, артефакты
CI и CD (deployment и delivery)
3. Обзор CI-систем
TrananosCI, CircleCI
Github Actions
Jenkins, TeamCity
Gitlab CI
4. Обзор Gitlab, его установка и настройка
Из чего состоит Gitlab, какие у него возможности и компоненты
Как установить Gitlab
Основные настройки системы Gitlab
5. Ваш первый проект в Gitlab
Создаем свой проект в Gitlab
Best Practices (учетки пользователей, LDAP-авторизация и т.д.)
6. Gitlab Runner и его настройка
Задачи и возможности runner
Какие есть виды и для каких кейсов они нужны
Настройка runner под проект
7. Файл .gitlab-ci.yml
Для чего этот файл нужен, что из себя представляет
Синтаксис, основные подходы. CI Linter от Gitlab
10. Интеграция с Kubernetes
Авторизация в кластере для раннеров
Нативный метод интеграции Gitlab с Kubernetes 
Практика: пишем пайплайн по разворачиванию приложения в Kubernetes через Gitlab
8. Приемы работы с Gitlab CI. Best Practices построения пайплайн
Include, шаблонизация.
Работа с переменными.
Условия работы со stage'ами.
Зависимости и параллельность stage.
Работа с инцидентами. Rollback и динамические окружения.
Добавление в пайплайн возможности Rollback
11. GitOps
Push и Pull модель для CI/CD пайплайнов
ArgoCD
12. Безопасность в CI/CD
Секретные переменные
Проверка кода на безопасность

Итог:
Пишем настоящий production-ready CI/CD процесс в GitLab CI




Рассмотрим процесс установки и настройки веб-инструмента жизненного цикла DevOps на Linux Ubuntu Server на примере версий 18.04 и 20.04. За основу взята официальная инструкция с сайта GitLab. В нашей инструкции приведен пример установки как платной. так и бесплатной версий программы.

Подготовка системы
    Обновление пакетов
    Настройка времени
    Порты в брандмауэре
Установка
    Зависимые компоненты
    GitLab
    Настройка
Вход на портал
Настройка GitLab
Создание проекта и отправка на него файла из Linux
Использование https
Сброс пароля для пользователя root
Подготовка сервера

В качестве предварительный настроек, мы обновим список пакетов в репозиториях, настроим правильное время и откроем порты в брандмауэре.
1. Обновление списков пакетов

Выполняем команду:
```bash
apt update
```
    При желании обновить установленные пакеты, также можно выполнить:
```bash
    apt upgrade
```
2. Время

Установим часовой пояс:
```bash
timedatectl set-timezone Europe/Moscow
```
* данная команда задаст настройки для московского времени. Все файлы с временными зонами находятся в каталоге /usr/share/zoneinfo.

Для автоматической синхронизации времени ставим пакет:
```bash
apt install chrony
```
И разрешаем автозапуск сервиса:
```bash
systemctl enable chrony
```
3. Настройка брандмауэра

По умолчанию, в Ubuntu брандмауэр настроен на то, чтобы принимать любые пакеты. Но если у нас он настроен на блокировку, нужно добавить порты 80 и 443.
```bash
iptables -I INPUT -p tcp --dport 80 -j ACCEPT
```
```bash
iptables -I INPUT -p tcp --dport 443 -j ACCEPT 
```
И чтобы сохранить правила, устанавливаем iptables-persistent:
```bash
apt install iptables-persistent
```
... и выполняем команду: 
```bash
netfilter-persistent save
```
Установка GitLab

Установку выполним в два шага — установка необходимых компонентов и, собственно, установка GitLab.
1. Необходимые компоненты
```bash
apt install curl openssh-server ca-certificates
```
Для отправки уведомлений, установим также postfix:
```bash
apt install postfix
```
При запросе типа конфигурации, выбираем Internet Site (если уведомления должны отправляться наружу) или Local only (уведомления в пределах сервера):

Выбираем тип конфигурации для Postfix

* при получении других запросов во время установки postfix можно ответить по умолчанию, нажимая Enter.
2. Установка GitLab

Установим репозиторий.

а) для платной версии:
```bash
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash
```
б) для бесплатной:
```bash
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
```
    Если установка выполняется на неподдерживаемую скриптом систему, например, Ubuntu новой версии (на момент обновления инструкции 22) или Astra Linux, мы увидим сообщение об ошибке:

    Unfortunately, your operating system distribution and version are not supported by this script.

    You can override the OS detection by setting os= and dist= prior to running this script.
    You can find a list of supported OSes and distributions on our website: https://packages.gitlab.com/docs#os_distro_version

    For example, to force Ubuntu Trusty: os=ubuntu dist=trusty ./script.sh

    Please email support@packagecloud.io and let us know if you run into any issues.

    Тогда скачиваем скрипт настройки репозитория:
```bash
    wget https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh
```
    * платная.
```bash
    wget https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh
```
    * бесплатная.

    После открываем скачанный файл:
```bash
    nano script.deb.sh
```
    Находим строки:
```bash
      # remove whitespace from OS and dist name
      os="${os// /}"
      dist="${dist// /}"
```
    И задаем значения для os и dist:
```bash
      os="ubuntu"
      dist="focal"
```
    * в нашем примере предполагается, что наша система подобна системе Ubuntu 20.04 (Focal Fossa).

    Запускаем скрипт:
```bash
    bash ./script.deb.sh
```
После установки репозитория, переходим к установке GitLab.

а) платную версию:
```bash
apt install gitlab-ee
```
б) бесплатную:
```bash
apt install gitlab-ce
```
Если установка прошла успешно, мы должны увидеть:
```bash
It looks like GitLab has not been configured yet; skipping the upgrade script.

       *.                  *.
      ***                 ***
     *****               *****
    .******             *******
    ********            ********
   ,,,,,,,,,***********,,,,,,,,,
  ,,,,,,,,,,,*********,,,,,,,,,,,
  .,,,,,,,,,,,*******,,,,,,,,,,,,
      ,,,,,,,,,*****,,,,,,,,,.
         ,,,,,,,****,,,,,,
            .,,,***,,,,
                ,*,.
  

     _______ __  __          __
    / ____(_) /_/ /   ____ _/ /_
   / / __/ / __/ /   / __ `/ __ \
  / /_/ / / /_/ /___/ /_/ / /_/ /
  \____/_/\__/_____/\__,_/_.___/
  

Thank you for installing GitLab!

```
3. Конфигурируем веб-адрес

Для запуска и корректной работы портала мы должны задать external_url. Для этого открываем файл:
```bash
nano /etc/gitlab/gitlab.rb
```
Нам нужно только изменить параметр external_url:
```bash
external_url 'http://gitlab.ekdeus.ru'
```
* данная настройка говорит, что наш веб-инструмент будет отвечать на запросы, которые пришли на узел gitlab.ekdeus.ru — это значит, что данное имя должно быть зарегистрирована в DNS или прописано в локальный файл hosts.

Выполняем конфигурирование:
```bash
gitlab-ctl reconfigure
```
Данная операция займет какое-то время.
Вход в веб-интерфейс

Открываем браузер и вводим наш адрес, который мы указали в настройках в опции external_url — в данном примере, http://gitlab.ekdeus.ru. Мы должны увидеть страницу авторизации, на которой нас запросят пароль для администратора.

Посмотреть пароль, который был назначен пользователю после установки можно в файле /etc/gitlab/initial_root_password:
```bash
cat /etc/gitlab/initial_root_password | grep Password:
```
Вводим в качестве пользователя root и пароль, который посмотрели в файле:

Меняем пароль для администратора gitlab

Мы должны войти в систему.
Настройка GitLab

Приведем некоторые примеры настроек, которые могут оказаться полезными.
Русский интерфейс

По умолчанию, портал устанавливается с интерфейсом на английском. Для смены языка, кликаем по иконке в правом верхнем углу и выбираем Settings:

Переходим к настройкам в GitLab

В меню слева нажимаем по Preferences:

Кликаем по пункту меню Preferences

В подразделе Localization выбираем нужный нам язык и первый день недели:

Выбираем языковые настройки для интерфейса GitLab

Сохранияем настройки и перезапускаем страницу для применения нового языка.
Создание репозитория и подключение к нему

Попробуем создать проект и подключиться к нему из Linux. Также для теста мы создадим файл и закинем его в наш репозиторий.

В веб-интерфейсе GitLab создаем новый проект:

Кликаем по кнопке Новый проект

Задаем имя проекта, оставляем или редактируем URL, выбираем уровень доступа. После кликаем по кнопке Создать проект:

Заполняем поля для создания нового проекта в GitLab

* в данном примере мы создаем проект с названием Test, url до него будет http://gitlab.ekdeus.ru/root/test. Уровень доступа мы задаем «Приватный» — доступ к репозиторию будет только у авторизованного пользователя.

Для примера попробуем подключиться с компьютера Linux к нашему репозиторию и закинуть на него тестовый файл.

Для начала установим git на компьютер с Linux:

а) Если используем CentOS / Red Hat:
```bash
yum install git-core
```


б) Если используем Ubuntu / Debian:
```bash
apt install git
```
Создаем папку для тестового проекта:
```bash
mkdir -p /projects/test
```
Переходим в нее:
```bash
cd /projects/test
```
Создаем репозиторий:
```bash
git init
```
Создаем файл:
```bash
nano testfile.txt
```
Добавляем в него все файлы (то есть, наш единственный файл):
```bash
git add .
```
Делаем коммит:
```bash
git commit -m "Очередное изменение проекта" -a
```
Подключаемся к созданному репозиторию:
```bash
git remote add origin http://gitlab.ekdeus.ru/root/test.git
```
Заливаем в него закоммиченный файл:
```bash
git push origin master
```
Переходим на веб-страницу нашего проекта — мы должны увидеть наш файл:

В проекте на нашем GitLab появился файл, который мы закинули в репозиторий с Linux
Настройка SSL

В данном примере мы сконфигурируем наш сервер для возможности работы по https и получения сертификата от Let's Encrypt. Все настройки выполняются в конфигурационном файле:
```bash
nano /etc/gitlab/gitlab.rb
```
Меняем настройку:
```bash
external_url 'http://gitlab.ekdeus.ru'
```
* где gitlab.ekdeus.ru — url для нашего портала, который мы задали при первом конфигурировании.

на:
```bash
external_url 'https://gitlab.ekdeus.ru'
```
* мы просто добавили s к http.

Также настраиваем получение сертификата от Let's Encrypt:
```bash
letsencrypt['enable'] = true
```
Также задаем:
```bash
nginx['redirect_http_to_https'] = true
```
* опция нужна, чтобы автоматически перебрасывать запросы с http на https.

И задаем опции для автоматического обновления сертификата:
```bash
letsencrypt['auto_renew'] = true
letsencrypt['auto_renew_hour'] = "22"
letsencrypt['auto_renew_minute'] = "50"
letsencrypt['auto_renew_day_of_month'] = "*/7"
```
* где:

    auto_renew — разрешает автоматическое обновление.
    auto_renew_hour — время в часах, когда нужно запускать задание на обновление сертификата.
    auto_renew_minute — время в минутах, когда нужно запускать задание на обновление сертификата.
    auto_renew_day_of_month — день месяца. В данном примере, раз в 7 дней.

    Если же у нас есть купленный сертификат, то мы не трогаем опции letsencrypt. 

    Нам нужно раскомментировать данные опции:
```bash
    nginx['ssl_certificate'] = "/etc/gitlab/ssl/#{node['fqdn']}.crt"
    nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/#{node['fqdn']}.key"
```
    * обратите внимание, что подразумевается наличие файла с доменом+расширение в каталоге /etc/gitlab/ssl. Можно оставить как есть или прописать свой путь.

    Указанного каталога нет — создаем его командой:
```bash
    mkdir -p /etc/gitlab/ssl/
```
Применяем новую конфигурацию:
```bsah
gitlab-ctl reconfigure
```
В процессе переконфигурирования мы можем получить ошибку получения сертификата. Пробуем запустить команду:
```bash
gitlab-ctl renew-le-certs
```
* данная команда нам не нужна, если мы прописали путь до своего сертификата.
Сброс пароля root

 Если мы забыли пароль для пользователя root, можно его сбросить через командную строку.

Подключаемся к консоли управления gitlab с помощью команды:
```bash
gitlab-rails console -e production
```
Создаем переменную, которая будет вести на ссылку с учетной записью root (идентификатор 1):
```bash
user = User.where(id: 1).first
```
Задаем пароль для пользователя root дважды:
```bash
user.password = 'password123'
user.password_confirmation = 'password123'
```
где password123 — созданный для пользователя root новый пароль.

Созраняем изменения для пользователя:
```bash
user.save!
```
Готово.


#Установка в Docker

Создайте файл /root/docker-compose.yml

```yaml
version: '3.7'
services:
  gitlab:
   container_name: gitlab
   image: 'gitlab/gitlab-ce:latest'
   restart: always
   hostname: 'https://ek-gitlab'
   environment:
     GITLAB_OMNIBUS_CONFIG: |
       external_url 'https://ek-gitlab'
       # Add any other gitlab.rb configuration here, each on its own line
   ports:
     - '80:80'
     - '443:443'
     - '22:22'
   volumes:
     - '/opt/gitlab/config:/etc/gitlab'
     - '/opt/gitlab/logs:/var/log/gitlab'
     - '/opt/gitlab/data:/var/opt/gitlab'

  gitlab-runner:
   container_name: gitlab-runner
   image: gitlab/gitlab-runner:latest
   restart: always
   volumes:
     - '/opt/gitlab-runner/data:/home/gitlab_ci_multi_runner/data'
     - '/opt/gitlab-runner/config:/etc/gitlab-runner'
     - '/var/run/docker.sock:/var/run/docker.sock:rw'
   environment:
     - CI_SERVER_URL=https://https://ek-gitlab/ci
```
