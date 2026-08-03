CKA (Certified Kubernetes Administrator)
CKA (Certified Kubernetes Administrator)
Настраивать
Капсулы
Устранение неполадок в модулях
Пространства имен
Узлы
Услуги
ReplicaSets
Устранение неполадок в ReplicaSets
Развертывания
Устранение неполадок при развертывании
Планировщик
Привязка узла
Метки и селекторы
Селектор узлов
Загрязнения
Ограничения ресурсов
Мониторинг
Планировщик
Настраивать
Настройте кластер Kubernetes. Используйте один из следующих вариантов.

Minikube для локального бесплатного и простого кластера
Управляемый кластер (EKS, GKE, AKS)
Установить псевдонимы

alias k=kubectl
alias kd=kubectl delete
alias kds=kubectl describe
alias ke=kubectl edit
alias kr=kubectl run
alias kg=kubectl get
Капсулы
Выполните команду, чтобы просмотреть все поды в текущем пространстве имен.

kubectl get pods

Примечание: создайте псевдоним ( alias k=kubectl) и привыкните к нему.k get po

Запустите под с именем "nginx-test", используя образ "nginx".

k run nginx-test --image=nginx

Предположим, у вас есть Pod с именем "nginx-test". Как его удалить?

k delete po nginx-test

В каком пространстве имен etcdзапущен под? Перечислите поды в этом пространстве имен.

k get po -n kube-system

Допустим, вы не знаете, в каком пространстве имен находится этот под. В таком случае вы можете запустить команду k get po -A | grep etc, чтобы найти под и узнать, в каком пространстве имен он находится.

Вывести список подов из всех пространств имен.

k get po -A

Полная версия будет выглядеть так kubectl get pods --all-namespaces: .

Напишите YAML-файл, описывающий Pod с двумя контейнерами, и используйте этот YAML-файл для создания Pod (используйте любые образы по вашему выбору).

cat > pod.yaml <<EOL
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  containers:
  - image: alpine
    name: alpine
  - image: nginx-unprivileged
    name: nginx-unprivileged
EOL

k create -f pod.yaml
Если вы задаетесь вопросом: «Как я запомню, что все это написал?», не волнуйтесь, можете просто убежать kubectl run some_pod --image=redis -o yaml --dry-run=client > pod.yaml. Если же вы спрашиваете себя: «Как я должен запомнить эту длинную команду?», пора изменить свое отношение ;)

Создайте YAML-файл пода, не запуская его фактически, с помощью команды kubectl (используйте любой образ по вашему выбору).

k run some-pod -o yaml --image nginx-unprivileged --dry-run=client > pod.yaml

Как проверить корректность манифеста?

с --dry-runфлагом, который фактически не создаст файл, но проверит его, и таким образом вы сможете обнаружить любые синтаксические ошибки.

k create -f YAML_FILE --dry-run

Как проверить, какой образ использует тот или иной Pod?

k describe po <POD_NAME> | grep -i image

Как проверить, сколько контейнеров запущено в одном Pod?

k get po POD_NAMEи посмотрите число в столбце "ГОТОВО".

Вы также можете бежатьk describe po POD_NAME

Запустите Pod под названием "remo" с последним образом Redis и меткой 'year=2017'.

k run remo --image=redis:latest -l year=2017

Перечислите капсулы и их этикетки.

k get po --show-labels

Удалите Pod с именем "nm".

k delete po nm

Перечислите все поды с меткой "env=prod".

k get po -l env=prod

Чтобы их пересчитать:k get po -l env=prod --no-headers | wc -l

Создайте статический под с образом python, который будет выполнять команду.sleep 2017

Сначала перейдите в каталог, отслеживаемый kubelet для создания статического пода: cd /etc/kubernetes/manifests(вы можете проверить путь, прочитав конфигурационный файл kubelet)

Теперь создайте определение/манифест в этой директории.

k run some-pod --image=python --command sleep 2017 --restart=Never --dry-run=client -o yaml > static-pod.yaml


Опишите, как бы вы удалили статический Pod.

Найдите каталог статических подов (его можно найти staticPodPathв конфигурационном файле kubelet).

Перейдите в указанную директорию и удалите манифест/определение статического пода ( rm <STATIC_POD_PATH>/<POD_DEFINITION_FILE>).

Устранение неполадок в модулях
Вы пытаетесь запустить Pod, но видите статус "CrashLoopBackOff". Что это значит? Как определить проблему?

Запуск контейнера не удался (по разным причинам), и Kubernetes пытается запустить под снова после некоторой задержки (= времени BackOff).

Вот несколько причин, по которым это может не получиться:

Неправильная конфигурация — орфографическая ошибка, неподдерживаемое значение и т. д.
Ресурс недоступен — узлы не работают, фотоэлектрические панели не установлены и т. д.
Несколько способов отладки:

kubectl describe pod POD_NAME
Основное внимание следует уделить Stateсостоянию «Ожидание» (или «CrashLoopBackOff»), а также Last Stateтому, что произошло ранее (например, почему произошел сбой).
Бегатьkubectl logs mypod
Это должно обеспечить точный результат.
Для конкретного контейнера можно добавить-c CONTAINER_NAME
Если вы до сих пор не понимаете, почему это не удалось, попробуйтеkubectl get events

Что означает эта ошибка ImagePullBackOff?

Скорее всего, вы неправильно указали имя образа, который пытаетесь загрузить и запустить. Или, возможно, он отсутствует в реестре.

Вы можете подтвердить это с помощьюkubectl describe po POD_NAME

Как проверить, на каком узле запущен тот или иной Pod?

k get po POD_NAME -o wide

Выполните следующую команду: kubectl run ohno --image=sheris. Получилось? Почему нет? Исправьте это, не удаляя Pod и используя любой образ по вашему желанию.

Потому что такого изображения нет sheris. По крайней мере, пока :)

Чтобы исправить это, запустите программу kubectl edit ohnoи измените следующую строку - image: sherisна - image: redisили любое другое изображение по вашему выбору.

Вы пытаетесь запустить Pod, но он находится в состоянии "Ожидание". В чём может быть причина?

Одна из возможных причин заключается в том, что планировщик, который должен планировать размещение Pod-ов на узлах, не запущен. Чтобы проверить это, вы можете запустить kubectl get po -A | grep schedulerили проверить непосредственно в kube-systemпространстве имен.

Как просмотреть логи контейнера, работающего в поде?

k logs POD_NAME

Внутри Pod-контейнера под названием "some-pod" находятся два контейнера. Что произойдет, если вы запустите команду?kubectl logs some-pod

Это не сработает, потому что внутри Pod находятся два контейнера, и вам нужно указать только один из них.kubectl logs POD_NAME -c CONTAINER_NAME

Пространства имен
Перечислите все пространства имен.

k get ns

Создайте пространство имен с именем 'alle'.

k create ns alle

Проверьте, сколько пространств имен существует.

k get ns --no-headers | wc -l

Проверьте, сколько подов существует в пространстве имен "dev".

k get po -n dev

Создайте под с именем "kartos" в пространстве имен dev. Под должен использовать образ "redis".

Если пространство имен еще не существует:k create ns dev

k run kratos --image=redis -n dev

Вы ищете Pod с именем "atreus". Как проверить, в каком пространстве имен он работает?

k get po -A | grep atreus

Узлы
Выполните команду для просмотра всех узлов кластера.

kubectl get nodes

Примечание: создайте псевдоним ( alias k=kubectl) и привыкните к нему.k get no

Создайте список всех узлов в формате JSON и сохраните его в файле с именем "some_nodes.json".

k get nodes -o json > some_nodes.json

Проверьте, какие метки присвоены одному из ваших узлов в кластере.

k get no minikube --show-labels

Услуги
Проверьте, сколько служб запущено в текущем пространстве имен.

k get svc

Создайте внутренний сервис с именем "sevi", который будет предоставлять доступ к приложению "web" через порт 1991.

kubectl expose pod web --port=1991 --name=sevi

Как сослаться по имени на сервис с именем "app-service" в пределах одного пространства имен?

сервис приложений

Как проверить целевой порт (TargetPort) сервиса?

k describe svc <SERVICE_NAME>

Как проверить, какие конечные точки поддерживает данный сервис?

k describe svc <SERVICE_NAME>

Как сослаться по имени на сервис с именем "app-service" в другом пространстве имен, называемом "dev"?

app-service.dev.svc.cluster.local

Предположим, у вас запущено развертывание, и вам нужно создать сервис для предоставления доступа к подам. Вот что требуется/известно:
Название развертывания: jabulik
Целевой порт: 8080
Тип сервиса: NodePort
Селектор: jabulik-app
Порт: 8080

kubectl expose deployment jabulik --name=jabulik-service --target-port=8080 --type=NodePort --port=8080 --dry-run=client -o yaml -> svc.yaml

vi svc.yaml(убедитесь, что селектор установлен на jabulik-app)

k apply -f svc.yaml

ReplicaSets
Как проверить, сколько репликационных наборов определено в текущем пространстве имен?

k get rs

У вас настроен репликационный набор для запуска 3 подов. Вы удалили один из этих 3 подов. Что произойдет дальше? Сколько подов останется?

Теоретически, будет по-прежнему работать 3 пода, поскольку цель репликационного набора — это обеспечить их функционирование. Поэтому, если вы удалите один или несколько подов, будут запущены дополнительные поды, и таким образом всегда будет работать 3 пода.

Как проверить, какой образ контейнера использовался в составе репликационного набора, называемого "repli"?

k describe rs repli | grep -i image

Как проверить, сколько Pod-ов готово к работе в составе репликационного набора, называемого "repli"?

k describe rs repli | grep -i "Pods Status"

Как удалить набор реплик с именем "rori"?

k delete rs rori

Как изменить набор реплик под названием "rori", чтобы использовать другой образ?

k edis rs rori

Увеличьте размер репликационного набора под названием "rori", чтобы он мог запускать 5 модулей вместо 2.

k scale rs rori --replicas=5

Уменьшите размер набора реплик под названием "rori", чтобы запускать 1 Pod вместо 5.

k scale rs rori --replicas=1

Устранение неполадок в ReplicaSets
Исправьте следующее определение ReplicaSet.
apiVersion: apps/v1
kind: ReplicaCet
metadata:
  name: redis
  labels:
    app: redis
    tier: cache
spec:
  selector:
    matchLabels:
      tier: cache
  template:
    metadata:
      labels:
        tier: cachy
    spec:
      containers:
      - name: redis
        image: redis

Типом следует быть ReplicaSet, а не ReplicaCet :)


Исправьте следующее определение ReplicaSet.
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: redis
  labels:
    app: redis
    tier: cache
spec:
  selector:
    matchLabels:
      tier: cache
  template:
    metadata:
      labels:
        tier: cachy
    spec:
      containers:
      - name: redis
        image: redis

Селектор не соответствует метке (cache против cachy). Чтобы это исправить, замените cachy на cache.


Развертывания
Как вывести список всех развертываний в текущем пространстве имен?

k get deploy


Как проверить, какой образ используется в конкретном развертывании?

k describe deploy <DEPLOYMENT_NAME> | grep image


Создайте файл определения/манифеста развертывания с именем "dep" и 3 репликами, использующими образ "redis".

k create deploy dep -o yaml --image=redis --dry-run=client --replicas 3 > deployment.yaml 


Удалите развертывание `depdep`.

k delete deploy depdep


Создайте развертывание с именем "pluck", используя образ "redis", и убедитесь, что оно запускает 5 реплик.

kubectl create deployment pluck --image=redis --replicas=5


Создайте развертывание со следующими свойствами:
называется "блюфер"
используя изображение "python"
запускает 3 реплики
Все модули будут размещены на узле с меткой "blufer".

kubectl create deployment blufer --image=python --replicas=3 -o yaml --dry-run=client > deployment.yaml

Добавьте следующий раздел ( vi deployment.yaml):

spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedlingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: blufer
            operator: Exists
kubectl apply -f deployment.yaml

Устранение неполадок при развертывании
Исправьте следующий манифест развертывания.
apiVersion: apps/v1
kind: Deploy
metadata:
  creationTimestamp: null
  labels:
    app: dep
  name: dep
spec:
  replicas: 3
  selector:
    matchLabels:
      app: dep
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: dep
    spec:
      containers:
      - image: redis
        name: redis
        resources: {}
status: {}

Изменить kind: Deployнаkind: Deployment

Исправьте следующий манифест развертывания.
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: dep
  name: dep
spec:
  replicas: 3
  selector:
    matchLabels:
      app: depdep
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: dep
    spec:
      containers:
      - image: redis
        name: redis
        resources: {}
status: {}

Селектор не соответствует метке (dep против depdep). Чтобы это исправить, замените depdep на dep.

Планировщик
Как запланировать запуск пода на узле с именем "node1"?

k run some-pod --image=redix -o yaml --dry-run=client > pod.yaml

vi pod.yamlи добавить:

spec:
  nodeName: node1
k apply -f pod.yaml

Примечание: если в вашем кластере нет узла node1, Pod будет находиться в состоянии "Ожидание".

Привязка узла
Используя привязку к узлу, настройте Pod для планирования на узле, где ключом является «region», а значением — либо «asia», либо «emea».

vi pod.yaml

affinity:
  nodeAffinity:
    requiredDuringSchedlingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: region
          operator: In
          values:
          - asia
          - emea

Используя привязку к узлу, настройте Pod таким образом, чтобы он никогда не планировался на узле, где ключ — "region", а значение — "neverland".

vi pod.yaml

affinity:
  nodeAffinity:
    requiredDuringSchedlingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: region
          operator: NotIn
          values:
          - neverland

Метки и селекторы
Как вывести список всех Pod-ов с меткой "app=web"?

k get po -l app=web

Как вывести список всех объектов с меткой "env=staging"?

k get all -l env=staging

Как вывести список всех развертываний из среды "env=prod" и типа "type=web"?

k get deploy -l env=prod,type=web

Селектор узлов
Присвойте метку "hw=max" одному из узлов в вашем кластере.

kubectl label nodes some-node hw=max


Создайте и запустите Pod с именем `some-pod`, используя образ `redis`, и настройте его для использования селектора `hw=max`.

kubectl run some-pod --image=redis --dry-run=client -o yaml > pod.yaml

vi pod.yaml

spec:
  nodeSelector:
    hw: max

kubectl apply -f pod.yaml

Объясните, почему селекторы узлов могут быть ограничены.

Предположим, вы хотите запустить свой Pod на всех узлах, hwустановив значение либо на максимальное, либо на минимальное, а не только на максимальное. Это невозможно с помощью nodeSelectors, которые довольно упрощены, и именно здесь вам следует рассмотреть другой подход node affinity.

Загрязнения
Проверьте наличие меток на узле "master".

k describe no master | grep -i taints

Создайте метку на одном из узлов кластера с ключом "app", значением "web" и эффектом "NoSchedule". Убедитесь, что она применена.

k taint node minikube app=web:NoSchedule

k describe no minikube | grep -i taints

Вы применили метку k taint node minikube app=web:NoScheduleна единственном узле в вашем кластере, а затем выполнили команду kubectl run some-pod --image=redis. Что произойдет?

Под останется в статусе «Ожидание», поскольку единственный узел в кластере имеет метку «app=web».

Вы применили метку (taint) к k taint node minikube app=web:NoScheduleединственному узлу в вашем кластере, а затем выполнили команду kubectl run some-pod --image=redis, но под находится в состоянии ожидания. Как это исправить?

kubectl edit po some-podи добавьте следующее

  - effect: NoSchedule
    key: app
    operator: Equal
    value: web
Выйдите и сохраните. Теперь модуль должен находиться в состоянии "Работает".

Удалите существующую метку с одного из узлов в вашем кластере.

k taint node minikube app=web:NoSchedule-

Ограничения ресурсов
Проверьте, нет ли каких-либо ограничений на один из модулей в вашем кластере.

kubectl describe po <POD_NAME> | grep -i limits

Запустите под с именем "yay" с образом "python" и запросите ресурсы: 64 МБ памяти и 250 МБ ЦП.

kubectl run yay --image=python --dry-run=client -o yaml > pod.yaml

vi pod.yaml

spec:
  containers:
  - image: python
    imagePullPolicy: Always
    name: yay
    resources:
      requests:
        cpu: 250m
        memory: 64Mi
kubectl apply -f pod.yaml

Запустите под с именем "yay2" и образом "python". Убедитесь, что он запрашивает ресурсы в объеме 64 МБ памяти и 250 МБ ЦП, а ограничения составляют 128 МБ памяти и 500 МБ ЦП.

kubectl run yay2 --image=python --dry-run=client -o yaml > pod.yaml

vi pod.yaml

spec:
  containers:
  - image: python
    imagePullPolicy: Always
    name: yay2
    resources:
      limits:
        cpu: 500m
        memory: 128Mi
      requests:
        cpu: 250m
        memory: 64Mi
kubectl apply -f pod.yaml

Мониторинг
Разверните сервер метрик

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

С помощью metrics-server просмотрите следующее:
лучшие узлы в кластере по производительности
лучшие по производительности модули

верхние узлы:kubectl top nodes
верхние капсулы:kubectl top pods

Планировщик
Можно ли развернуть несколько планировщиков?

Да, это возможно. Вы можете запустить другой под с помощью команды, аналогичной следующей:

spec:
  containers:
  - command:
    - kube-scheduler
    - --address=127.0.0.1
    - --leader-elect=true
    - --scheduler-name=some-custom-scheduler
...

Предположим, у вас несколько планировщиков. Как узнать, какой планировщик использовался для конкретного пода?

Запустив программу, kubectl get eventsвы сможете увидеть, какой планировщик использовался.

Вы хотите запустить новый Pod и запланировать его запуск с помощью пользовательского планировщика. Как это сделать?

Добавьте в спецификацию Pod следующее:

spec:
  schedulerName: some-custom-scheduler
