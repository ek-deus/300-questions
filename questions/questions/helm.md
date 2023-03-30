Концепты Helm

https://rtfm.co.ua/helm-kubernetes-package-manager-obzor-nachalo-raboty/

Три основных концепта Helm:

    chart: пакет информации, необходимой для создания приложения к кластере Kubernetes
    config: информация о настройках и параметрах, которые использутся chart-ом для создания инстанса приложения и управления его релизами
    release: работающий инстанс chart-а, связанный с определённым config-ом

Компоненты Helm

Helm можно разделить на две основных части – клиент, и набор библиотек.

    Helm client: утилита командной строки (весь Helm написан на Go), отвечает за локальную разработку чартов, взаимодействие с репозиториями чартов, управление релизами, и взаимодействие с библиотекой Helm – отправка новых чартов для установки, управление релизами запущенных приложений
    Helm library: предоставляет логику работы Helm, отвечает за взаимодействие с API Kubernetes-кластера, управляет чартами и их конфигами, релизами, установкой чартов в кластер, их обновление и удаление

Helm Charts

Итак, chart, чарт – набор файлов, описывающий определённый набор ресурсов Kubernetes, с помощью которого можно выполнить запуск пода с одним сервисом, например веб-сервер, или полноценное приложение, включающее в себя и веб-сервер, и приложения фронтента, бекнда, баз данных, систем кеширования и т.д.
Структура файлов

Чарты организованы в виде каталогов и файлов, где имя каталога верхнего уровня, содержащего другие директории и файлы будет именем такого чарта.

Например:
tree example-chart/
example-chart/
|-- Chart.yaml
|-- charts
|-- templates
|   |-- NOTES.txt
|   |-- _helpers.tpl
|   |-- deployment.yaml
|   |-- hpa.yaml
|   |-- ingress.yaml
|   |-- service.yaml
|   |-- serviceaccount.yaml
|   `-- tests
|       `-- test-connection.yaml
`-- values.yaml

   

Тут:

    example-chart – каталог, имя чарта
        Chart.yaml – файл с метаданными чарта, описывающий что это за чарт, содержащий информацию о версиях, зависимостях, авторе чарта и так далее
        charts: чарт может содержать subcharts – они будут располагатьсяв этом каталоге
        templates – содержит файлы шаблонов, которые будут применены к кластеру для запуска приложения, используя шаблонизатор Go
            NOTES.txt – help-текст, который будет отображаться пользователям
            deployment.yaml – пример манифеста с ресурсом Kubernetes Deployment
            service.yaml – пример манифеста с ресурсом  Kubernetes Service
        values.yaml – содержит значения по-умолчанию для шаблонов – Services, Deployments, и т.д.

Хватит теории – вперёд, к практике!
Подготовка окружения

Используем Minikube, kubectl, и Helm v3.

    minikube v1.9
    Kubernetes v1.18
    kubectl v1.18.2
    helm-3.2.0

Minikube – кластер Kubernetes

Устанавливаем minikube:

sudo pacman -S minikube



Создаём кластер:
minikube start
  minikube v1.9.2 on Arch rolling
...
  Enabling addons: default-storageclass, storage-provisioner
  Done! kubectl is now configured to use "minikube"



Проверяем:
kubectl cluster-info
Kubernetes master is running at https://192.168.99.100:8443
KubeDNS is running at https://192.168.99.100:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy



Ноды кластера:
kubectl get nodes
NAME       STATUS   ROLES    AGE     VERSION
minikube   Ready    master   4m16s   v1.18.0
  Re-playCopy to ClipboardPauseFull View


Установка Helm

Arch Linux – из репозитория:
sudo pacman -S helm

    Re-playCopy to ClipboardPauseFull View

macOS – Homebrew:
brew install helm

    Re-playCopy to ClipboardPauseFull View

Debian, etc – с помощью Snap:
sudo apt install snapd
sudo snap install helm --classic

    Re-playCopy to ClipboardPauseFull View

И отличнейшная подсказка по helm help:

 
Создание Helm Chart

Далее – создадим свой чарт, обновим в нём шаблоны, и задеплоим какое-то приложение в Kubernretes.

Создаём новый чарт:
helm create example-chart
Creating example-chart


Его содержимое мы уже видели выше:
tree example-chart/
example-chart/
|-- Chart.yaml
|-- charts
|-- templates
|   |-- NOTES.txt
|   |-- _helpers.tpl
|   |-- deployment.yaml
|   |-- hpa.yaml
|   |-- ingress.yaml
|   |-- service.yaml
|   |-- serviceaccount.yaml
|   `-- tests
|       `-- test-connection.yaml
`-- values.yaml
3 directories, 10 files


Содержимое его деплоймента:
cat example-chart/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
name: {{ include "example-chart.fullname" . }}
labels:
{{- include "example-chart.labels" . | nindent 4 }}
spec:
{{- if not .Values.autoscaling.enabled }}
replicas: {{ .Values.replicaCount }}
{{- end }}
...


И Values:
cat example-chart/values.yaml
Default values for example-chart.
This is a YAML-formatted file.
Declare variables to be passed into your templates.
replicaCount: 1
...


Тут всё достаточно очевидно – для replicas: {{ .Values.replicaCount }} в templates/deployment.yaml будет использовано значение replicaCount: 1 из values.yaml.

Но мы эти шаблоны использовать не будем – а напишем свой велосипед чарт.

Удаляем их:
rm -rf example-chart/templates/*


Далее добавим свой ConfigMap.
Добавление template

Создадим файл example-chart/templates/configmap.yaml, а в нём – содержимое для файла index.html:
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-configmap
data:
  index.html: "Hello, World
chart linter

Перед установкой чарта имеет смысл выполнить проверку синтаксиса чарта – используем helm lint:
helm lint example-chart/
==> Linting example-chart/
[INFO] Chart.yaml: icon is recommended
1 chart(s) linted, 0 chart(s) failed


chart install

Ещё раз проверим – на какой кластер настроен наш kubectl:
kubectl config current-context
minikube


Для установки выполняем helm install, которому первым аргументом передаём имя релиза, потом опции, и путь к файлам чарта.

Перед выполнением реальных действий имеет смысл выполнить “тестовый прогон” – используем --dry-run, и добавим --debug для деталей:
helm install example-chart --dry-run --debug example-chart/
install.go:159: [debug] Original chart version: ""
install.go:176: [debug] CHART PATH: /home/setevoy/Work/RTFM/example-chart
NAME: example-chart
LAST DEPLOYED: Sun May  3 13:17:07 2020
NAMESPACE: default
STATUS: pending-install
REVISION: 1
TEST SUITE: None
USER-SUPPLIED VALUES:
{}
COMPUTED VALUES:
affinity: {}
autoscaling:
enabled: false
maxReplicas: 100
minReplicas: 1
targetCPUUtilizationPercentage: 80
fullnameOverride: ""
image:
pullPolicy: IfNotPresent
repository: nginx
tag: ""
...
---
Source: example-chart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
name: nginx-configmap
data:
index.html: "Hello, World"


Ошибок нет – устанавливаем его:
helm install example-chart example-chart/
NAME: example-chart
LAST DEPLOYED: Sun May  3 13:20:52 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None


Проверяем его статус:
helm get manifest example-chart
---
Source: example-chart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
name: nginx-configmap
data:
index.html: "Hello, World"


helm вывел информацию о ресурсах и файлах, которые были применены к релизу.

Проверим с kubectl:
kubectl describe cm nginx-configmap
Name:         nginx-configmap
Namespace:    default
Labels:       app.kubernetes.io/managed-by=Helm
Annotations:  meta.helm.sh/release-name: example-chart
meta.helm.sh/release-namespace: default
Data
====
index.html:
----
Hello, World


chart uninstall

Аналогично, для удаления используем helm uninstall и имя релиза:
helm uninstall example-chart
release "example-chart" uninstalled


Template переменные

Хорошо – всё работает, но сейчас все данные в нашем ConfigMap статичны.

Изменим это – включим в работу шаблонизатор.

Для helm достпен набор предопределённых значений, например – Release.Name, см. полный список в документации.

Редактируем шаблон нашего ConfigMap:
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  index.html: "Hello, World"

И добавим своих переменных – удаляем текущий файл values.yaml:
rm example-chart/values.yaml


И создаём его заново, но с одним полем – user: "Username":
user: "Username"

А затем используем переменную user в шаблоне:
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  index.html: "Hello, {{ .Values.user }}"

Тут через .Values мы указываем обращение к нашему values,yaml, из которого helm должен подтянуть значение переменной user.

Применяем:
helm install example-chart example-chart/
NAME: example-chart
LAST DEPLOYED: Sun May  3 13:35:49 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None


Проверяем:
kubectl get cm
NAME                      DATA   AGE
example-chart-configmap   1      31s


И содержимое:
kubectl describe cm example-chart-configmap
Name:         example-chart-configmap
Namespace:    default
Labels:       app.kubernetes.io/managed-by=Helm
Annotations:  meta.helm.sh/release-name: example-chart
meta.helm.sh/release-namespace: default
Data
====
index.html:
----
Hello, Username


chart upgrade

Что, если мы хотим изменить значение Username?

Можно удалить релиз, и задеплоить заново, передав новое значение либо через редактирование values.yaml, либо через --set:
helm uninstall example-chart
release "example-chart" uninstalled
helm install example-chart example-chart/ --set user=NewUser


Проверяем:
kubectl describe cm example-chart-configmap
...
Data
====
index.html:
----
Hello, NewUser


Другой вариант – вызвать helm upgrade, передать имя релиза и новое значение:
helm upgrade example-chart example-chart/ --set user=AnotherOneUser
Release "example-chart" has been upgraded. Happy Helming!
...


Результат:
kubectl describe cm example-chart-configmap
...
Data
====
index.html:
----
Hello, AnotherOneUser


helm package

Что бы передать наш чарт другим пользователем – можно его упаковать в tgz-файл.

Используем helm package, который создаст файл вида имя-чарта-версия-чарта.tgz.

Версию helm получит из метаданных чарта:
cat example-chart/Chart.yaml | grep  version:
version: 0.1.0


Упаковываем:
helm  package example-chart/
Successfully packaged chart and saved it to: /home/setevoy/Work/RTFM/example-chart-0.1.0.tgz


И содержимое архива:
tar tf example-chart-0.1.0.tgz
example-chart/Chart.yaml
example-chart/values.yaml
example-chart/templates/configmap.yaml
example-chart/.helmignore


Helm репозитории

Для передачи чартов используем helm-репозитории.

Раньше можно было запустить локальный репоизторий с помощью helm serve, но в Helm v3 его выпилили.

Для работы с репозиториями в V3 используем helm repo.

По-умолчанию в список репозиториев сразу добавляется репозиторий от Google:
helm repo list
NAME    URL
stable  https://kubernetes-charts.storage.googleapis.com/


Running local Helm repo

Для того, что бы создать свой репозиторий – достаточно выполнить helm package чарта, после чего сгенерировать файл index.yaml в каталоге, который хранит архив.

В качестве бекенда для репозиториев может быть что угодно – от Github Pages, до AWS S3, см. The Chart Repository Guide.

Тут пример локального репозитория.

Создаём каталог, переносим в него архив:
mkdir helm-local-repo
mv example-chart-0.1.0.tgz helm-local-repo/


Инициализируем репозиторий:
helm repo index helm-local-repo/


Содержимое:
tree helm-local-repo/
helm-local-repo/
|-- example-chart-0.1.0.tgz
`-- index.yaml


Что бы получить к нему доступ – запустим простой NGINX:
sudo docker run -ti -v $(pwd)/helm-local-repo/:/usr/share/nginx/html -p 80:80 nginx


Проверяем подключение:
curl localhost/index.yaml
apiVersion: v1
entries:
example-chart:
- apiVersion: v2
appVersion: 1.16.0
created: "2020-05-03T14:04:44.896115358+03:00"
description: A Helm chart for Kubernetes
digest: afa314247a03c4c85f339bda665659f3ab13a5e8656336e14ed37ed7f31b5352
name: example-chart
type: application
urls:
- example-chart-0.1.0.tgz
version: 0.1.0
generated: "2020-05-03T14:04:44.895678349+03:00"


Добавляем этот репозиторий в список зарегистрированных в локальном Helm:
helm repo add example-chart http://localhost
"example-chart" has been added to your repositories


Проверяем:
helm repo list
NAME            URL
stable          https://kubernetes-charts.storage.googleapis.com/
example-chart   http://localhost


Пробуем поиск чарта:
helm search repo example
NAME                            CHART VERSION   APP VERSION     DESCRIPTION
example-chart/example-chart     0.1.0           1.16.0          A Helm chart for Kubernetes


Устанавливаем его:
helm install example-chart-from-repo example-chart/example-chart
NAME: example-chart-from-repo
LAST DEPLOYED: Sun May  3 14:15:51 2020
NAMESPACE: default
STATUS: deployed


Проверяем в Kubernretes:
kubectl get cm
NAME                                DATA   AGE
example-chart-configmap             1      34m
example-chart-from-repo-configmap   1      22s


В целом на этом всё.
