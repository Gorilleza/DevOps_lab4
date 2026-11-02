# Лабораторная работа №4. Методы и средства оркестрации в многоконтейнерных системах

**Студент:** Ходжамухаммедов Эзиз  
**Группа:** 13  

---

## Цель работы
Изучение создания кластера и работы с Kubernetes.

---

## Задание 1. Подготовка окружения для локального кластера

Для работы был установлен локальный кластер Kubernetes с помощью утилиты `kind` (Kubernetes in Docker) и клиента `kubectl`.

**Создание кластера:**
```bash
kind create cluster --name my-cluster
```
**Проверка статуса кластера:**
```
kubectl get nodes
```
Кластер успешно запущен, нода my-cluster-control-plane имеет статус Ready.
<img width="567" height="323" alt="Снимок экрана — 2026-11-02 в 12 14 03" src="https://github.com/user-attachments/assets/dafdbbe4-79cc-479b-9472-6dfdd01bb9ff" />

---

Задание 2. Знакомство с ресурсами Kubernetes

В ходе выполнения задания были изучены основные ресурсы Kubernetes (Pod, ReplicaSet) и продемонстрирована работа с томами.

1. Создание Pod с монтированием тома

Был создан манифест pod-with-volume.yaml, который описывает под с одним контейнером Nginx и монтирует в него пустой том (emptyDir).
```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-with-volume
spec:
  containers:
  - name: my-container
    image: nginx:alpine
    volumeMounts:
    - name: my-volume
      mountPath: /usr/share/nginx/html
  volumes:
  - name: my-volume
    emptyDir: {}
```
Применение манифеста:
```
kubectl apply -f pod-with-volume.yaml
```
Под успешно запущен (статус Running).

2. Создание ReplicaSet

Был создан манифест replicaset.yaml для управления тремя репликами пода.
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: nginx:alpine
```
Применение манифеста:

```
kubectl apply -f replicaset.yaml
```
ReplicaSet успешно создал 3 пода (статус Running).

<img width="541" height="269" alt="Снимок экрана — 2026-11-02 в 12 16 45" src="https://github.com/user-attachments/assets/ff63adf5-b8a7-42df-acec-50b17c64a37d" />


---

**1. Что такое Kubernetes и зачем он нужен в работе с контейнерами?**
Kubernetes (K8s) — это платформа для оркестрации контейнеров. Она автоматизирует развертывание, масштабирование и управление контейнеризированными приложениями. Он нужен для обеспечения отказоустойчивости, балансировки нагрузки и автоматического восстановления контейнеров.

**2. Объясните, в чём отличие «запустить контейнеры вручную через Docker» от «управлять ими в Kubernetes».**
В Docker вы запускаете контейнеры на одной машине вручную (командой `docker run`). В Kubernetes вы описываете желаемое состояние (например, «хочу 3 копии приложения»), и система сама поддерживает это состояние: перезапускает упавшие поды, распределяет их по разным серверам (нодам) и масштабирует.

**3. Приведите 1–2 примера задач, где оркестрация экономит время.**
- Автоматическое восстановление: если контейнер упал, Kubernetes сам его перезапустит.
- Масштабирование: при росте нагрузки можно одной командой увеличить количество реплик с 3 до 10.

**4. Назовите основные компоненты кластера Kubernetes и опишите, за что каждый отвечает.**
- **Master Node (Control Plane):** управляет кластером (API Server, Scheduler, Controller Manager, etcd).
- **Worker Node:** запускает поды (Kubelet, Kube-proxy, Container Runtime).

**5. Как создать под с помощью YAML-манифеста? Перечислите обязательные поля.**
Обязательные поля: `apiVersion` (версия API, например `v1`), `kind` (тип ресурса, например `Pod`), `metadata` (имя, метки), `spec` (описание контейнеров).

**6. Приведите YAML-файл для простого пода. Что произойдёт, если пропустить обязательное поле?**
Пример:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
```
Если пропустить apiVersion или kind, Kubernetes выдаст ошибку валидации и не создаст ресурс.

**7. Что такое сервисы в Kubernetes и зачем они нужны? Разница между ClusterIP и Headless.**
Сервис (Service) — это абстракция, которая дает постоянный IP-адрес и DNS-имя для группы подов. ClusterIP — стандартный сервис, доступный только внутри кластера. Headless-сервис (clusterIP: None) не имеет IP, а возвращает DNS-записи напрямую для каждого пода (используется для StatefulSet).

**8. Чем отличаются сервисы NodePort и LoadBalancer?**
NodePort: открывает порт на каждой ноде кластера (диапазон 30000-32767). Внешний пользователь заходит на IP любой ноды и этот порт.
LoadBalancer: создает внешний облачный балансировщик нагрузки, который автоматически направляет трафик на ноды (доступен только в облачных провайдерах).

**9. Укажите, какие порты задействованы и какие ограничения есть у NodePort.**
Порты: 30000–32767. Ограничения: работает только на IP-адресах нод (если нода упала, доступ пропадает), не поддерживает SSL-терминацию на уровне балансировщика.

**10. Приведите пример, когда NodePort достаточно, а когда нужен LoadBalancer.**
NodePort достаточно для тестирования или внутренних сервисов. LoadBalancer нужен для публичного продакшн-приложения с высокими требованиями к доступности и SSL.

**11. Что такое Ingress и какую проблему он решает? Пример манифеста.**
Ingress — это правило маршрутизации HTTP/HTTPS трафика. Он решает проблему создания множества NodePort/LoadBalancer сервисов (один Ingress может направлять трафик на разные сервисы по путям).
Пример:
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```

**12. Сравните Deployment, StatefulSet и DaemonSet.**

Deployment: для stateless-приложений (веб-серверы). Поды создаются параллельно, не привязаны к узлам. При масштабировании до 3 реплик создаст 3 одинаковых пода.
StatefulSet: для stateful-приложений (базы данных). Поды создаются по порядку (0, 1, 2) и имеют уникальные имена и постоянные диски. При масштабировании до 3 добавит под с индексом 3.
DaemonSet: для агентов на каждой ноде (логирование, мониторинг). Создает по одному поду на каждой ноде. При масштабировании не увеличивает количество подов, а просто покрывает все ноды.

**13. В чём разница между Job и CronJob?**

Job: выполняет задачу один раз до успешного завершения (например, миграция БД).
CronJob: запускает Job по расписанию (например, бэкап каждую ночь).
Проверить статус: `kubectl get jobs` или `kubectl get cronjobs`. Логи: `kubectl logs <pod-name>`.

**14. Для чего нужны ConfigMap и Secret? Способы передачи данных в под.**
ConfigMap — для хранения неконфиденциальных данных (конфиги). Secret — для паролей, токенов, ключей.
Способы: 1) как переменные окружения (envFrom), 2) как файлы через монтирование томов (volumeMounts). Удобнее монтировать файлы для больших конфигов, а переменные — для коротких параметров.

**15. YAML для ConfigMap и Secret. Почему пароли нельзя хранить в ConfigMap?**
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  app.properties: |
    color=blue
---
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  password: cGFzc3dvcmQ= # base64
```
Пароли нельзя хранить в ConfigMap, потому что они не шифруются (хранятся в открытом виде) и доступны всем, у кого есть доступ к кластеру. Secret хотя бы кодирует их в base64 и имеет механизмы шифрования (если включено).

**16. 8–10 основных команд kubectl, сгруппированных по задачам.**

Посмотреть: `kubectl get pods`, `kubectl describe pod <name>`, `kubectl logs <pod>`, `kubectl get nodes`.
Создать/обновить: `kubectl apply -f file.yaml`, `kubectl create ...`.
Удалить: `kubectl delete pod <name>`, `kubectl delete -f file.yaml`.
Отладка: `kubectl exec -it <pod> -- bash`, `kubectl port-forward <pod> 8080:80`.

**17. Что такое пространства имён (namespaces) и зачем они используются? Примеры.**
Namespaces — это логическое разделение кластера на изолированные части. Примеры: dev, staging, prod (разные среды), team-a, team-b (разные команды).

**18. Как создать новое пространство имён и как выполнять команды только в нём?**
Создать: kubectl create namespace my-ns.
Выполнять команды: `kubectl get pods -n my-ns` или переключить контекст: `kubectl config set-context --current --namespace=my-ns`.

**19. Зачем нужны пробы: livenessProbe, readinessProbe и startupProbe?**
livenessProbe: проверяет, жив ли контейнер (если нет — перезапускает).
readinessProbe: проверяет, готов ли контейнер принимать трафик.
startupProbe: проверяет, запустилось ли приложение (дает время на медленный старт).

**20. Что такое Persistent Volume (PV) и Persistent Volume Claim (PVC)?**
PV — это физический кусок хранилища (диск). PVC — это запрос от приложения на выделение хранилища (например, «мне нужно 10 ГБ»). Kubernetes связывает PVC с подходящим PV.

**21. Как Kubernetes реагирует, если под или узел «упали»?**

Если под упал — ReplicaSet/Deployment создаст новый под.
Если узел упал — Controller Manager пересоздаст поды на других узлах (через 5 минут).
Если контейнер внутри пода упал — Kubelet перезапустит его.

**22. Как обновлять приложение без простоя? Что такое rolling update?**
Rolling update — это постепенное обновление подов (сначала создается новый под, затем удаляется старый). Параметры в Deployment: maxSurge (сколько новых подов можно создать) и maxUnavailable (сколько старых можно удалить одновременно).

**23. Как запустить обновление приложения и следить за прогрессом?**
Запустить: изменить образ в манифесте и выполнить `kubectl apply -f deployment.yaml`.
Следить: `kubectl rollout status deployment/my-app`.
