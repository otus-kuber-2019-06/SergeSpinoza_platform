# SergeSpinoza_platform
SergeSpinoza Platform repository

## Оглавление
1. [Домашнее задание №1, Minikube](#домашнее-задание-1)
2. [Домашнее задание №2, RBAC](#домашнее-задание-2)
3. [Домашнее задание №3, Сети](#домашнее-задание-3)
4. [Домашнее задание №4, Volumes, Storages, StatefulSet](#домашнее-задание-4)
5. [Домашнее задание №5, Kubernetes-storage](#домашнее-задание-5)
6. [Домашнее задание №6, Kubernetes-debug](#домашнее-задание-6)
7. [Домашнее задание №7, Kubernetes-operators](#домашнее-задание-7)
10. [Домашнее задание №10, Kubernetes-templating](#домашнее-задание-10)
11. [Домашнее задание №11, Kubernetes-vault](#домашнее-задание-11)


<br><br>
## Домашнее задание 1
### Выполнено
- Развернут minikube ( https://kubernetes.io/docs/tasks/tools/install-minikube/ );
- Добавлен Dashboard (команда `minikube addons enable dashboard`, зайти в панель - команда `minikube dashboard`);
- Установлен k9s ( https://k9ss.io );
- Проверено, что контейнеры восстанавливаются после удаления;
- Создан Dockerfile, который запускает http сервер на 8000 порту под пользователем с uid 1001 (сделал на python); 
- Написан манифест web-pod.yaml;
- Выполнен деплой пода web и init контейнера;
- Опробован в работе kube-forwarder (https://kube-forwarder.pixelpoint.io ).


### Ответы на вопросы
1. `kube-apiserver` запускается и рестартится через Systemd Service операционной системы, а `coredns` - контролируется самим k8s через ReplicaSet; 

### Полезное
Команды: 
- `minikube start` - старт minikube;
- `minikube ssh` - войти на виртуальную машину minikube по ssh;
- `kubectl cluster-info` - проверка подключения к кластеру;
- `kubectl get pods -n kube-system` - показать все pods в namespace `kube-system`;
- `kubectl get cs` - показать состояние кластера;
- `kubectl get ...` - показать ресурсы;
- `kubectl describe ...` - показать детальную информацию о конктретном ресурсе;
- `kubectl logs ...` - показать логи контейнера в поде;
- `kubectl exec ...` - выполнить команду в контейнере в поде;
- `kubectl apply -f file.yaml` - применить манифест;  
- `kubectl get pod web -o yaml` - получить манифест уже запущенного pod;
- `kubectl port-forward --address 0.0.0.0 pod/web 8000:8000` - редирект порта (в примере 8000);


<br><br>
## Домашнее задание 2
### Выполнено
- TASK 01:
  - Создать Service Account bob, дать ему роль admin в рамках всего кластера;
  - Создать Service Account dave без доступа к кластеру;
- TASK 02:
  - Создать Namespace prometheus;
  - Создать Service Account carol в этом Namespace;
  - Дать всем Service Account в Namespace prometheus возможность делать get, list, watch в отношении Pods всего кластера;
- TASK 03:
  - Создать Namespace dev;
  - Создать Service Account jane в Namespace dev;
  - Дать jane роль admin в рамках Namespace dev;
  - Создать Service Account ken в Namespace dev;
  - Дать ken роль view в рамках Namespace dev.


### Полезное
Команды: 
- `kubectl auth can-i get/create ...` - проверка прав/возможности создания (пример `kubectl auth can-i get deployments --as system:serviceaccount:dev:ken -n dev`, `kubectl auth can-i create deployments --as system:serviceaccount:dev:ken -n dev`).


<br><br>
## Домашнее задание 3
### Выполнено
- Добавили к ранее созданному манифесту web-pod.yaml readinessProbe и livenessProbe;
- Создали манифест deployment с указанием стратегии обновления;
- Создали манифест сервиса с ClusterIP web-svc-cip;
- Переключили kube-proxy в режим `ipvs`; 
- Очистили правила iptables в minikube от мусора; 
- Настроили работу с MetalLB (к сожалению в macos доступ через браузер получить не удалось даже с прописанным маршрутом и драйвером hyperkit);
- Установили и настроили работу ingress-nginx; 

### Задание со * 
- Настроен LoadBalancer для CoreDNS (kube-dns) для 53 TCP и UDP портов с одинаковым внешним ip. Для проверки необходимо применить манифест `kubectl apply -f coredns-svc-lb.yaml` и проверить, выполнив команду из ssh minikube, например (c выводом):

```
# nslookup web-svc.default.svc.cluster.local 172.17.255.3
Server:    172.17.255.3
Address 1: 172.17.255.3 coredns-svc-lb-tcp.kube-system.svc.cluster.local

Name:      web-svc.default.svc.cluster.local
Address 1: 172.17.0.7 172-17-0-7.web-svc.default.svc.cluster.local
Address 2: 172.17.0.9 172-17-0-9.web-svc.default.svc.cluster.local
Address 3: 172.17.0.8 172-17-0-8.web-svc.default.svc.cluster.local
```
- Добавлен доступ к kubernetes-dashboard через настроенный ingress-proxy (манифест добавлен в файл `web-ingress.yaml`);
- Реализовано канареечное развертывание, манифесты `canary-namespaces.yaml` и `canary-ingress.yaml`. 

### Развертывание и проверка канареечного развертывания: 
- Применить манифест `kubectl apply -f canary-namespaces.yaml`, который создаст необходимые namespace;
- Развернуть приложение production `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/docs/examples/http-svc.yaml -n echo-production`; 
- Развернуть "тестовое" приложение `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/docs/examples/http-svc.yaml -n echo-canary`;
- Применить манифест `kubectl apply -f canary-ingress.yaml` для создания ingress для канареечного развертывания;
- Узнать ip minikube, выполнив команду `minikube ip`;
- Прописать в /etc/hosts следующую строчку: `<minikube_ip> echo.com`
- Проверить работу следующим образом: 
  - Через браузер зайти на сайт http://echo.com, обратить внимание, что строка `pod namespace:` имеет значение: `echo-production`; 
  - С помощью плагина ModHeader для браузера googleChome (ссылка https://chrome.google.com/webstore/detail/modheader/idgpnmonknjnojddfkpgkljpfnnfcklj?hl=ru) добавить заголовок `CanaryByHeader` со значением `DoCanary` и обновить страницу;
  - После этого строка `pod namespace:`должна изменить значение на: `echo-canary`.


### Ответы на вопросы
1. Почему следующая конфигурация валидна, но не имеет смысла?

```
livenessProbe:
  exec:
   command:
     - 'sh'
     - '-c'
     - 'ps aux | grep my_web_server_process'
```

**Ответ:** потому что данная проверка всегда будет отдавать процесс самой команды grep. Чтобы эта проверка работала - надо исключить из результатов grep - например так:

```
livenessProbe:
  exec:
   command:
     - 'sh'
     - '-c'
     - 'ps aux | grep -v grep | grep my_web_server_process'
```

2. Бывают ли ситуации, когда она все-таки имеет смысл?
**Ответ:** если исключать в выводе процесс самой команды grep - то это ситуации когда процесс только делает что-то внутри контейнера (допустим пишет что-то в файле на persistent volume) и не слушает никакой tcp/udp порт (т.е. проверить этот сервис по сети не возможно). 

3. Попробуйте разные варианты деплоя с крайними значениями maxSurge и maxUnavailable (оба 0, оба 100%, 0 и 100%)
**Ответ:** если maxSurge и maxUnavailable выставить в 0 - то получаем ошибку невозможности выполнения деплоя, вида 
```
The Deployment "web" is invalid: spec.strategy.rollingUpdate.maxUnavailable: Invalid value: intstr.IntOrString{Type:0, IntVal:0, StrVal:""}: may not be 0 when `maxSurge` is 0
```
При других вариантах все работает корректно, т.к. maxUnavailable задает максимальное количество недоступных подов при обновлении, а maxSurge - превышение количества подов при обновлении от заданного в `replicas`;


### Полезное
Команды: 
- `kubectl apply -f web-pod.yaml --force` - принудительное обновление;
- `kubectl --namespace kube-system edit configmap/kube-proxy` - редактирование конфига kube-proxy (для включения ipvs - выставить `mode` в `ipvs`); 
- `minikube ssh` - зайти в ssh minikube;
- `iptables --list -nv -t nat` - показать nat правила iptables;
- Внутри ssh minikube команда `toolbox` - запустить и зайти в контейнер с Fedora;
  - `dnf install -y ipvsadm && dnf clean all` - установка ipvsadm внутри контейнера Fedora;
  - `ipvsadm --list -n` - список правил ipvs;
- `kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.8.0/manifests/metallb.yaml` - установка MetalLb;
- `kubectl --namespace metallb-system logs controller-xxxxxxxxxxx-xxxxx` - логи контроллера MetalLB
- `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml` или `minikube addons enable ingress` - установка ingress-nginx;


<br><br>
## Домашнее задание 4
### Выполнено
- Установлен и запущен kind ( https://github.com/kubernetes-sigs/kind/ );
- Развернут StatefulSet с Minio;
- Развернут Headless Service.


### Задание со * 
- Учетные данные из StatefulSet вынесены в secret (манифест 01-minio-secret.yaml);
- Для кодирования учетных данных в base64 необходимо выполнить команды:  
  - `echo -n 'login' | base64`;
  - `echo -n 'password' | base64`
- Полученные значения записать в secret манифест. 


### Полезное
Команды: 
- `kind create cluster` - запуск кластера kind;
- `kind delete cluster` - удаление кластера kind
- `kubectl get statefulsets`
- `kubectl get pods`
- `kubectl get pvc`
- `kubectl get pv`
- `kubectl describe <resource> <resource_name>`


<br><br>
## Домашнее задание 5
## Kubernetes-storage
### Выполнено
- Подготовлен файл cluster.yaml с необходимой конфигурацией для kind;
- В директории `hw` созданы манифесты для создания объектов StorageClass, pvc и pod;
- Задание со * - развернут iscsi (добавлен диск lvm), в под добавлен volume iscsi, описана методика создания снапшотов и восстановления из них; 

### Как запустить: 
#### Предварительные действия: 
- Установить go версии > 1.13 (https://sourabhbajaj.com/mac-setup/Go/README.html)
- Установить kind (https://kind.sigs.k8s.io/docs/user/quick-start/)
- Перейти в директорию `kubernetes-storage/cluster` и создать кластер с помощью kind с кастомным конфигом: `kind create cluster --config cluster.yaml`
- Задать переменную окружения, для использования конфига kind: `export KUBECONFIG="$(kind get kubeconfig-path)"`
- Установить CSI Host Path Driver: 
  - `git clone https://github.com/kubernetes-csi/csi-driver-host-path.git`
  - `./csi-driver-host-path/deploy/kubernetes-1.15/deploy-hostpath.sh`

#### Основные действия: 
- Перейти в директорию `kubernetes-storage/hw` и последовательно выполнить: 
  - `kubectl apply -f 01-storageclass.yaml`
  - `kubectl apply -f 02-storage-pvc.yaml`
  - `kubectl apply -f 03-storage-pod.yaml`
- Проверить результат. 

#### Задание со * 
**Для развертывания iscsi необходимо выполнить команды на чистой машине с ubuntu 18.04:**
- `apt -y install targetcli-fb` - устанавливаем targetcli;
- Добавляем еще один диск (допустим /dev/sdb); 
- Произведем действия, для создания lvm 
```
# parted /dev/sdb
mklabel msdos
q
```

```
# parted -s /dev/sdb unit mib mkpart primary 1 100% set 1 lvm on
```

```
# pvcreate /dev/sdb1
# vgcreate vg0 /dev/sdb1
# lvcreate -l 10%FREE -n base vg0
# mkfs.ext4 /dev/vg0/base
```

**Для создания блочного устройства на отдельном диске:**
- `targetcli` - открывает targetcli
  - `/> ls` - просмотр иерархии;
  - `/> backstores/block create name=iscsi-disk dev=/dev/vg0/base` - создаем блочное устройство в бэксторе;

**Для создания блочного устройства в виде файла на существующем диске:**
- `mkdir /tmp/iscsi_disks`
- `targetcli` - открывает targetcli
  - `/> ls` - просмотр иерархии;
  - `/> backstores/fileio create iscsi_file /tmp/iscsi_disks/disk01.img 1G` - создаем блочное устройство в бэксторе;

**Далее:**
- `/iscsi create` - создаем таргет;
- `/iscsi` и `ls` - просмотрим название созданного таргета; 
- `iqn.2003-01.org.linux-iscsi.iscsi-1.x8664:sn.c0904cfa5297/tpg1/` - перейдем к созданному таргету к tpg1;
- `luns/ create /backstores/block/iscsi-disk` - создадим LUN;
- `set attribute authentication=0` - отключим авторизацию;
- `acls/` - перейдем в ACL;
- `create wwn=iqn.2019-09.com.example.srv01.initiator01` - указать iqn имя инициатора, который будет иметь право подключаться к таргету (это имя потом надо будет прописать в настройках воркер-нод кластера kubernetes, подробнее ниже);
- `cd /`, `ls`, `saveconfig` - сохраняем конфиг (по умолчанию сохраняется сюда: `/etc/rtslib-fb-target/saveconfig.json`);
- `exit`
(взято отсюда https://kifarunix.com/how-to-install-and-configure-iscsi-storage-server-on-ubuntu-18-04/)

**Далее разворачиваем кластер (я использовал kubespray). При создании машин кластера необходимо сразу при создании добавить второй интерфейс, который будет подключен к сети где находиться iscsi.** 
- `git clone https://github.com/kubernetes-sigs/kubespray.git`;
- `sudo pip install -r requirements.txt`;
- `cp -rfp inventory/sample inventory/mycluster`;
- отредактировать файл `inventory/mycluster/inventory.ini` согласно структуре кластера;
- `ansible-playbook -i inventory/mycluster/inventory.ini --become --become-user=root --user=spinoza --key-file=~/.ssh/id_rsa cluster.yml` - развернуть кластер; 

**Действия на воркер-нодах кластера kubernetes:**
- `apt -y install open-iscsi` - установим open-iscsi
- настроим конфиг `/etc/iscsi/initiatorname.iscsi`, внеся туда корректное имя, которое мы использовали ранее `iqn.2019-09.com.example.srv01.initiator01`
- добавим open-iscsi в автозагрузку и запустим:
  - `systemctl restart iscsid open-iscsi`
  - `systemctl enable iscsid open-iscsi`

**Проверка и снапшоты:**
- `kubectl apply -f kubernetes-storage/iscsi/01-iscsi-pod.yaml` - запустим под с использованием iscsi;
- зайдем в под `kubectl exec -it iscsi-pod -- /bin/bash`
- создадим файлик `echo "ISCSI TEST!" > /mnt/iscsi-test.txt`
- перейдем на машину с iscsi и создадим lvm снапшот: 
  - `lvcreate --snapshot --size 1G  --name ss-01 /dev/vg0/base`
- перейдем обратно в под и удалим данные - `rm -rf /mnt/iscsi-test.txt`
- удалим сам под `kubectl delete -f kubernetes-storage/iscsi/01-iscsi-pod.yaml`
- передем на машину с iscsi и исключим наш диск из iscsi: 
  - `targetcli`
  - `/> backstores/block delete iscsi-disk`
  - `exit`
  - восстановимся из снапшота `lvconvert --merge /dev/vg0/ss-01`
  - `targetcli`
  - `/> backstores/block create name=iscsi-disk dev=/dev/vg0/base`
  - `/> /iscsi/iqn.2003-01.org.linux-iscsi.iscsi-1.x8664:sn.c0904cfa5297/tpg1/`
  - `/> luns/ create /backstores/block/iscsi-disk`
  - `exit`
- снова запустим под `kubectl apply -f kubernetes-storage/iscsi/01-iscsi-pod.yaml`
- зайдем в под `kubectl exec -it iscsi-pod -- /bin/bash`
- проверим, что наш файл на месте `cat /mnt/iscsi-test.txt`


<br><br>
## Домашнее задание 6
## Kubernetes-debug
### Выполнено
- Ознакомились с работой `kubectl debug`, выявили проблему с агентом debug;
- Установили kube-iptables-tailer и необходимые для него сервис аккаунт, роль и биндинг;
- Установили тестовое приложение netperf-operator (оператор, который позволяет запускать тесты
пропускной способности сети между нодами кластера); 
- Добавили сетевую потилику Calico для эмуляции сетевой проблемы;
- Посмотрели логи calico на воркер-ноде `journalctl -k | grep calico-pack`;
- Запустили iptables-tailer для диагностики сетевой проблемы и добились того, чтобы iptables-tailer нормально запустился и писал лог;

### Выполнено задание со *
- Исправлена ошибка в сетевой политике, чтобы netperf заработал; 
- Исправлен манифест DaemonSet, чтобы показывались имена подов место ip адресов(?). 

### Как запустить: 
#### Предварительные действия: 
- Развернуть кластер (kind не подходит, нужен полноценный кластер минимум с 2 воркер нодами (минимум 2 ноды нужно для работы тестового приложения netperf-operator)); 
- Установить kubectl debug, согласно документации https://github.com/aylei/kubectl-debug
- Клонировать репозиторий netperf-operator - `git clone https://github.com/piontec/netperf-operator.git`
- Выполнить манифесты для запуска оператора в кластере: 
```
cd kubernetes-debug/netperf-operator
kubectl apply -f ./deploy/crd.yaml
kubectl apply -f ./deploy/rbac.yaml
kubectl apply -f ./deploy/operator.yaml

```

#### Основные действия: 
##### Задание 1 (проблема с контейнером-агентом):
При выполнении strace получаем ошибку вида: 

```
bash-5.0# strace -c -p1
strace: test_ptrace_get_syscall_info: PTRACE_TRACEME: Operation not permitted
strace: attach: ptrace(PTRACE_ATTACH, 1): Operation not permitted
```

Смотрим на наличие capabilities с использованием команды `docker inspect` (выполняем для запущенного debug контейнера на воркер-ноде):

```
            "CapAdd": null,
            "CapDrop": null,
``` 

Смотри добавляются ли необходимые capabilities в коде (по той ссылке что в подсказке): 
```
CapAdd:      strslice.StrSlice([]string{"SYS_PTRACE", "SYS_ADMIN"}),
```
- в коде все в порядке.

Смотрим версию образа агента: 

```
Normal  Pulling    16m   kubelet, node2     Pulling image "aylei/debug-agent:0.0.1"
```
- версия подозрительно старая. 

Смотри код в старой версии по ссылке https://github.com/aylei/kubectl-debug/blob/0.0.1/pkg/agent/runtime.go - а нужной строчки с capabilities там нет. 

Удаляем агента: 
`kubectl delete -f https://raw.githubusercontent.com/aylei/kubectl-debug/master/scripts/agent_daemonset.yml`

Устанавливаем последнюю версию агента, используя такой же манифест как и официальный, только с измененной версий образа (изменена на latest):
`kubectl apply -f kubernetes-debug/strace/01-agent_daemonset.yaml`

После этого еще раз проверяем strace и видим, что все успешно отрабатывает:

```
bash-5.0# strace -c -p1
strace: Process 1 attached
^Cstrace: Process 1 detached
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
  0.00    0.000000           0         2           poll
  0.00    0.000000           0         1           restart_syscall
------ ----------- ----------- --------- --------- ----------------
100.00    0.000000                     3           total
```

##### Задание 2 (диагностика сетевой проблемы):
Выполнить уже исправленные манифесты в следующей последовательности: 
- `kubectl apply -f kit/kit-clusterrole.yaml`;
- `kubectl apply -f kit/kit-serviceaccount.yaml`;
- `kubectl apply -f kit/kit-clusterrolebinding.yaml`;
- `kubectl apply -f kit/netperf-calico-policy.yaml`;
- `kubectl apply -f kit/iptables-tailer.yaml`;

Для запуска тестового приложения выполнить: 
- `kubectl apply -f netperf-operator/deploy/cr.yaml`;


##### Задание со `*`:
- Для исправления ошибки в сетевой политике необходимо исправить и применить манифест `kit/netperf-calico-policy.yaml`. Нужно строчки `selector: netperf-role == "netperf-client"` заменить на `selector: netperf-type in {"client", "server"}` (согласно документации https://docs.projectcalico.org/v3.7/reference/calicoctl/resources/globalnetworkpolicy#selector). И после этого запустить для проверки тест еще раз;
-  Возможно, судя по документации (https://github.com/box/kube-iptables-tailer), для того, чтобы отображалось имя пода, необходимо в DaemonSet изменить значение переменной окружения POD_IDENTIFIER с `label` на `name` (файл `kit/iptables-tailer.yaml`) и применить манифест, но я разницы не заметил. У меня в обоих случаях в поле `OBJECT` отображается имя пода, а в `MESSAGE` присутствует его ip адрес, делалось все на кластере GCE v1.14.  


<br><br>
## Домашнее задание 7
## Kubernetes operators
### Выполнено
- Создан CRD для MySQL:
  - Добавлена проверка наличия всех необходимых строчек в спецификации;
- Создали custom controller на python;
- Создали docker образ оператора mysql;
- Проверили работу оператора.

### Ответы на вопросы: 
- Вопрос: почему объект создался, хотя мы создали CR, до того, как запустили контроллер?
  - Ответ: Судя по документации (https://kopf.readthedocs.io/en/latest/walkthrough/starting/) оператор ищет уже созданный объект, т.е. по алгоритму работы - в начале применяется cr, а потом уже запускается оператор. 

### Как запустить: 
- Запустить minikube
- Установить python модули:
  - kopf (https://kopf.readthedocs.io/en/latest/install/)
  - kubernetes
  - jinja2
  - pyyaml

#### Ручной запуск (проверка работы оператора)
- В отдельной консоли запустить наш контроллер: `kopf run mysql-operator.py`
- В другом окне консоли применить манифест cr: `kubectl apply -f deploy/cr.yml`
- Записать что-нибуь в созданную базу данных, для этого выполним команды: 

```
export MYSQLPOD=$(kubectl get pods -l app=mysql-instance -o jsonpath="{.items[*].metadata.name}")

kubectl exec -it $MYSQLPOD -- mysql -u root -potuspassword -e "CREATE TABLE test ( id smallint unsigned not null auto_increment, name varchar(20) not null, constraint pk_example primary key (id) );" otus-database

kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "INSERT INTO test ( id, name ) VALUES ( null, 'some data' );" otus-database

kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "INSERT INTO test ( id, name ) VALUES ( null, 'some data-2' );" otus-database
```

- Проверить что данные записались в базу: 

```
kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "select * from test;" otus-database
```

- Для проверки, что работает восстановление из бекапа удалить mysql-instance `kubectl delete mysqls.otus.homework mysql-instance`

- Применить cr еще раз: `kubectl apply -f deploy/cr.yml`
- Проверить, что восстановление БД отработало:

```
export MYSQLPOD=$(kubectl get pods -l app=mysql-instance -o jsonpath="{.items[*].metadata.name}")

kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "select * from test;" otus-database
```

#### Со сборкой образа
- Собрать docker образ и запушить его на docker Hub (выполнять в директории `kubernetes-operators/build`):
  - `docker build -t s1spinoza/mysql-operator:v0.0.1 .` 
  - `docker push s1spinoza/mysql-operator:v0.0.1`
- Применить манифесты (находясь в директории `kubernetes-operators/deploy`): 
  - `kubectl apply -f service-account.yml`
  - `kubectl apply -f role.yml`
  - `kubectl apply -f role-binding.yml`
  - `kubectl apply -f deploy-operator.yml`
- Применить манифест cr: `kubectl apply -f cr.yml`
- Далее добавить в базу записи и проверить восстановление из бекапа (как расписано выше в `Ручной запуск`)


#### Необходимые выводы команд: 

```
# kubectl get jobs
NAME                         COMPLETIONS   DURATION   AGE
backup-mysql-instance-job    1/1           3s         6m43s
restore-mysql-instance-job   1/1           42s        5m57s
```

```
# export MYSQLPOD=$(kubectl get pods -l app=mysql-instance -o jsonpath="{.items[*].metadata.name}")
# kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "select * from test;" otus-database
mysql: [Warning] Using a password on the command line interface can be insecure.
+----+-------------+
| id | name        |
+----+-------------+
|  1 | some data   |
|  2 | some data-2 |
+----+-------------+
```

<br><br>
## Домашнее задание 10
## Kubernetes-templating
### Выполнено
- Использованы разные способы установки helm:
  - Helm 2 и tiller с правами cluster-admin
  - Helm 2 и tiller с правами, ограниченными namespace
  - Helm 2 с плагином helm-tiller (позволяет отказаться от использования tiller внутри кластера)
  - Helm 3


### Как запустить: 

#### Задание 1 (nginx и cert-manager)
- Установить helm2 согласно инструкции (https://github.com/helm/helm/blob/master/docs/install.md );
- Сроздать сервисный аккаунт для tiller, выполнив команду: 
  - `kubectl apply -f kubernetes-templating/tiller/v1/01-sa.yaml` 
  - `kubectl apply -f kubernetes-templating/tiller/v1/02-cluster-role-binding.yaml` 
- Установить tiller `helm init --service-account tiller`
- Создать release nginx-ingress используя Helm 2 и tiller с правами cluster-admin, выполнив команду: 

 ```
helm upgrade --install nginx-ingress stable/nginx-ingress --wait \
  --namespace=nginx-ingress \
  --version=1.11.1
 ```
- Установить tiller, с правами на определенный namespace. Для этого выполнить команду: 
  - `kubectl apply -f kubernetes-templating/tiller/v2/`
- Инициализировать helm в namespace cert-manager:
  - `helm init --tiller-namespace cert-manager --service-account tiller-cert-manager`
- Добавить репозиторий, в котором хранится актуальный helm chart cert-manager: 
  - `helm repo add jetstack https://charts.jetstack.io`
- Создать namespace `kubectl create namespace cert-manager`
- Выполнить дополнительную подготовку для установки cert-manager - `kubectl label namespace cert-manager certmanager.k8s.io/disable-validation="true"`
- Установить необходимые для cert-manager CRD - `kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v0.10.1/cert-manager.yaml`
- Проверить, что tiller в namespace cert-manager действительно НЕ обладает правами на управление объектами в других namespace (например, в nginx-ingress)(должны получить ошибку):

```
helm upgrade --install cert-manager jetstack/cert-manager --wait \
  --namespace=nginx-ingress \
  --version=0.10.1 \
  --tiller-namespace cert-manager
```

- Установим cert-manager в необходимый namespace: 

```
helm upgrade --install cert-manager jetstack/cert-manager --wait \
  --namespace=cert-manager \
  --version=0.10.1 \
  --tiller-namespace cert-manager \
  --atomic
```
- Получаем ошибку, из-за отсутствия необходимых прав у сервис-аккаунта tiller-cert-manager
- Установим тиллер не необходимым сервисным аккаунтом, выполнив команду `helm init --service-account tiller`
- Произведем теже самые действия с необходимыми правами (используя сервисный аккаунт tiller c правами cluster-admin):

```
helm upgrade --install cert-manager jetstack/cert-manager --wait \
  --namespace=cert-manager \
  --version=0.10.1 \
  --atomic
```

- Также для корректной работы cert-meneger необходимо создать ClusterIssuer
 (если необходимо генерировать сертификаты в любом неймспейсе) или Issuer (если нужно генерировать только в конкретном namespace). Для этого применим манифест `kubectl apply -f kubernetes-templating/cert-manager/01-clusterissuer.yaml`

**Ответы на вопросы:** 
- Новая ошибка должна подсказать вам, почему в данном случае установка tiller в namespace cert-manager и ограничение прав для него не имеет практического смысла
  - Ответ: cert-manager при установке пытается создать ресурс clusterroles (роль в кластере) в API группе rbac.authorization.k8s.io, т.е. ему для установки в любом случае придется дать права распространяющиеся не только на namespace где он установлен.   


#### Задание 2 (chartmuseum)
- Для запуска tiller локально необходимо установить плагин `helm plugin install https://github.com/rimusz/helm-tiller`
- Установим chartmuseum с помощью локального tiller, для этого выполним команду: 

```
helm tiller run \
  helm upgrade --install chartmuseum stable/chartmuseum --wait \
  --namespace=chartmuseum \
  --version=2.3.2 \
  -f kubernetes-templating/chartmuseum/values.yaml
```

- Проверить, что тиллер внутри кластера ничего не знает об установке `helm list`
- Проверить, что локальный tiller знает `helm tiller run helm list`
- Чтобы tiller внутри кластера видел то что установил tiller локальный, можно указать перед развертыванием чарта (перед `helm tiller`) - `export HELM_TILLER_STORAGE=configmap`, тогда информация о релизе будет храниться в секретах;
- Проверить успешную установку, chartmuseum должен быть доступен по url указанном в vaslues.yaml в ingress и иметь валидный ssl сертификат;

#### Задание 2 (chartmuseum) со *
- Для возможности работы нужно включить API в chartmuseum, для этого добавить в values.yaml 

```
env:
  open:
    DISABLE_API: false
```

- Добавить свой репозиторий в helm - `helm repo add chartmuseum https://motomap.info` и `helm repo update`
- Проверим чарт `helm lint`
- Запакуем чарт `helm package .`
- Отправим свой чарт в репо `curl -L --data-binary "@mysql-1.0.0.tgz" https://motomap.info/api/charts`
- Установить свой чарт из своего репо `helm install chartmuseum/mysql --name demo-mysql`


#### Задание 3 (harbor и helm3)
- Установить helm3 созласно документации https://github.com/helm/helm/releases
- Добавить репозиторий harbor `helm3 repo add harbor https://helm.goharbor.io` и `helm3 repo update`
- Создать namespace harbor - `kubectl create namespace harbor`
- Установить harbor с версией чарта 1.1.2: 

```
helm3 upgrade --install harbor harbor/harbor --wait \
--namespace=harbor \
--version=1.1.2 \
-f kubernetes-templating/harbor/values.yaml
```

- Проверить, что вход на harbor по адресу https://harbor.motomap.info работает и SSL сертификат валидный; 
- Посмотреть как helm3 хранит информацию о релизе - `kubectl get secrets -n harbor -l owner=helm`

#### Задание 3 (helmfile) со *
- Установить утилиту `helmfile`;
- Для развертывания нужно перейти в директорию `kubernetes-templating/helmfile` и выполнить команду `helmfile sync --concurrency=1`
- Для удаления выполнить команду `helmfile destroy` 


Полезные ссылки: 
helmfile - https://github.com/roboll/helmfile
helmfile templates - https://github.com/roboll/helmfile/blob/master/docs/writing-helmfile.md#release-template--conventional-directory-structure
helmfile templates - https://medium.com/@naseem_60378/helmfile-its-like-a-helm-for-your-helm-74a908581599


#### Задание 4 (свой чарт)
- Добавить зависимости `helm dep update kubernetes-templating/socks-shop`, указанные в файле `kubernetestemplating/socks-shop/requirements.yaml`
- Установить чарт:

```
helm upgrade --install socks-shop kubernetes-templating/socks-shop \
  --wait \
  --atomic
```


#### Задание 4 (свой чарт) со *
Добавил в requirements.yaml установку одной из mongoDB (carts-db) из community chart's. Чтобы ничего не сломалось - изменил некоторые параметры: 

```
helm upgrade --install socks-shop kubernetes-templating/socks-shop \
  --set frontend.service.NodePort=31234 \
  --set mongodb.service.name=carts-db \
  --set mongodb.usePassword=false \
  --set mongodb.persistence.enabled=false \
  --wait 
```

Вопрос - не нашел как в сабчарт передавать параметры не через `--set`, а через файл. Как это правильно делать? 



#### Проверка
- Делаем архивы чартов `helm package .`;
- Загружаем их в harbor через web интерфейс;
- Запускаем созданный скрипт kubernetes-templating/repo.sh
- Смотрим, что есть в репо:

```
# helm search templating
NAME                  CHART VERSION APP VERSION DESCRIPTION
templating/frontend   0.1.0         1.0         A Helm chart for Kubernetes
templating/socks-shop 0.1.0         1.0         A Helm chart for Kubernetes
```


#### Задание 6 (kubecfg)
- Установить kubecfg - `brew install kubecfg`
- Проверить что манифесты корректно создаются: `kubecfg show services.jsonnet`
- Установить манифесты: `kubecfg update services.jsonnet`


#### Задание 7 (Kustomize)
Документация: https://github.com/kubernetes-sigs/kustomize 
- Кастомизирована установка сервиса и деплоймента `queue-master` с разными параметрыми в разные namespace. Для установки выполнить команду (пример для установки в namespace socks-shop-prod): `kubectl apply -k kubernetes-templating/kustomize/overlays/socks-shop-prod`



### Полезные ключи helm
- `--wait` - ожидать успешного окончания установки;
- `--timeout` - считать установку неуспешной по истечении указанного времени;
- `--namespace` - установить chart в определенный namespace (будет создан, если не существует);
- `--version` - установить определенную версию chart;


<br><br>

## Домашнее задание 11
## Kubernetes-vault
### Выполнено
- Выполнена исталляция и настройка vault HA (с consul) на k8s;
- Завели в vault секреты;
- Настроили авторизацию через k8s;
- Cоздали CA на базе vault.


### Как запустить: 

#### Инсталляция hashicorp vault HA в k8s
- установить helm и tiller, а также создать сервисный аккаунт для tiller;
- перейти в директорию `kubernetes-vault`;
- склонировать репозиторий consul: `git clone https://github.com/hashicorp/consul-helm.git`;
- установить consul: `helm install --name=consul consul-helm`;
- установить vault: `git clone https://github.com/hashicorp/vault-helm.git`;
- установим vault: `helm install --name=vault vault-helm`;
- проверим статус установки vault `helm status vault`:

```
LAST DEPLOYED: Sun Nov  3 11:59:54 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME          DATA  AGE
vault-config  1     7m7s

==> v1/Pod(related)
NAME     READY  STATUS   RESTARTS  AGE
vault-0  0/1    Running  0         7m7s
vault-1  0/1    Running  0         7m7s
vault-2  0/1    Running  0         7m6s

==> v1/Service
NAME      TYPE       CLUSTER-IP   EXTERNAL-IP  PORT(S)            AGE
vault     ClusterIP  10.77.3.204  <none>       8200/TCP,8201/TCP  7m7s
vault-ui  ClusterIP  10.77.6.219  <none>       8200/TCP           7m7s

==> v1/ServiceAccount
NAME   SECRETS  AGE
vault  1        7m7s

==> v1/StatefulSet
NAME   READY  AGE
vault  0/3    7m7s

==> v1beta1/PodDisruptionBudget
NAME   MIN AVAILABLE  MAX UNAVAILABLE  ALLOWED DISRUPTIONS  AGE
vault  N/A            1                0                    7m7s

NOTES:

Thank you for installing HashiCorp Vault!

Now that you have deployed Vault, you should look over the docs on using
Vault with Kubernetes available here:

https://www.vaultproject.io/docs/


Your release is named vault. To learn more about the release, try:

  $ helm status vault
  $ helm get vault
```

- провести инициализацию серез любой под vault `kubectl exec -it vault-0 -- vault operator init --key-shares=1 --key-threshold=1`

```
Unseal Key 1: +3Rvi29dl/Y860jssTos7odFPShxZwP3VJAws3vOB2c=

Initial Root Token: s.aZ7PeufImmNIu1UEzFF6O7Pg
```

- распечатать каждый под: 

```
kubectl exec -it vault-0 -- vault operator unseal
kubectl exec -it vault-1 -- vault operator unseal
kubectl exec -it vault-2 -- vault operator unseal
```

- еще раз проверим статус установки vault `helm status vault`:

```
LAST DEPLOYED: Sun Nov  3 11:59:54 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME          DATA  AGE
vault-config  1     16m

==> v1/Pod(related)
NAME     READY  STATUS   RESTARTS  AGE
vault-0  1/1    Running  0         16m
vault-1  1/1    Running  0         16m
vault-2  1/1    Running  0         16m

==> v1/Service
NAME      TYPE       CLUSTER-IP   EXTERNAL-IP  PORT(S)            AGE
vault     ClusterIP  10.77.3.204  <none>       8200/TCP,8201/TCP  16m
vault-ui  ClusterIP  10.77.6.219  <none>       8200/TCP           16m

==> v1/ServiceAccount
NAME   SECRETS  AGE
vault  1        16m

==> v1/StatefulSet
NAME   READY  AGE
vault  3/3    16m

==> v1beta1/PodDisruptionBudget
NAME   MIN AVAILABLE  MAX UNAVAILABLE  ALLOWED DISRUPTIONS  AGE
vault  N/A            1                1                    16m

NOTES:

Thank you for installing HashiCorp Vault!

Now that you have deployed Vault, you should look over the docs on using
Vault with Kubernetes available here:

https://www.vaultproject.io/docs/


Your release is named vault. To learn more about the release, try:

  $ helm status vault
  $ helm get vault
```

- посмотрим список доступных авторизаций `kubectl exec -it vault-0 -- vault auth list` и получим ошибку;
- залогинимся в vault `kubectl exec -it vault-0 -- vault login`, введем root token. Вывод: 

```
Token (will be hidden):
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                s.aZ7PeufImmNIu1UEzFF6O7Pg
token_accessor       8QJzDxzsMJOc8ooLjYke0PxE
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
```

- повторно запросим список авторизаций `kubectl exec -it vault-0 -- vault auth list`, вывод: 

```
Path      Type     Accessor               Description
----      ----     --------               -----------
token/    token    auth_token_3520d4e2    token based credentials
```

- заведем секреты: 

```
kubectl exec -it vault-0 -- vault secrets enable --path=otus kv
kubectl exec -it vault-0 -- vault secrets list --detailed
kubectl exec -it vault-0 -- vault kv put otus/otus-ro/config username='otus' password='asajkjkahs'
kubectl exec -it vault-0 -- vault kv put otus/otus-rw/config username='otus' password='asajkjkahs'
kubectl exec -it vault-0 -- vault read otus/otus-ro/config
kubectl exec -it vault-0 -- vault kv get otus/otus-rw/config
```

Вывод чтения секрета: 

```
Key                 Value
---                 -----
refresh_interval    768h
password            asajkjkahs
username            otus

====== Data ======
Key         Value
---         -----
password    asajkjkahs
username    otus
```

- включим авторизацию через k8s:

```
kubectl exec -it vault-0 -- vault auth enable kubernetes
kubectl exec -it vault-0 -- vault auth list
```

Вывод обновленного списка авторизаций: 

```
Path           Type          Accessor                    Description
----           ----          --------                    -----------
kubernetes/    kubernetes    auth_kubernetes_c5c0ec97    n/a
token/         token         auth_token_3520d4e2         token based credentials
```

- создадим Service Account vault-auth и применим ClusterRoleBinding

```
kubectl create serviceaccount vault-auth
kubectl apply --filename vault-auth-service-account.yml
```

- подготовим переменные для записи в конфиг кубер авторизации:

```
export VAULT_SA_NAME=$(kubectl get sa vault-auth -o jsonpath="{.secrets[*]['name']}")
export SA_JWT_TOKEN=$(kubectl get secret $VAULT_SA_NAME -o jsonpath="{.data.token}" | base64 --decode; echo)
export SA_CA_CRT=$(kubectl get secret $VAULT_SA_NAME -o jsonpath="{.data['ca\.crt']}" | base64 --decode; echo)
export K8S_HOST=$(more ~/.kube/config | grep server |awk '/http/ {print $NF}')

### alternative way
export K8S_HOST=$(kubectl cluster-info | grep 'Kubernetes master' | awk '/https/ {print $NF}' | gsed 's/\x1b\[[0-9;]*m//g' )
```

- Ответ на вопрос, что делает конструкция `sed 's/\x1b\[[0-9;]*m//g'` - она убирает ANSI коды цветового оформления из вывода (для macos нужно использовать команду gsed);

- запишем конфиг в vault:

```
kubectl exec -it vault-0 -- vault write auth/kubernetes/config \
  token_reviewer_jwt="$SA_JWT_TOKEN" \
  kubernetes_host="$K8S_HOST" \
  kubernetes_ca_cert="$SA_CA_CRT"
```

- создадим политку и роль в vault: 

```
kubectl cp otus-policy.hcl vault-0:./tmp
kubectl exec -it vault-0 -- vault policy write otus-policy /tmp/otus-policy.hcl

kubectl exec -it vault-0 -- vault write auth/kubernetes/role/otus \
  bound_service_account_names=vault-auth \
  bound_service_account_namespaces=default policies=otus-policy ttl=24h
```

- создадим под с привязанным сервис аккоунтом и установим туда curl и jq: 

```
kubectl run --generator=run-pod/v1 tmp --rm -i --tty --serviceaccount=vault-auth --image alpine:3.7
apk add curl jq
```

- залогинимся и получим клиентский токен: 

```
VAULT_ADDR=http://vault:8200

KUBE_TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)

curl --request POST --data '{"jwt": "'$KUBE_TOKEN'", "role": "otus"}' $VAULT_ADDR/v1/auth/kubernetes/login | jq

TOKEN=$(curl -k -s --request POST --data '{"jwt": "'$KUBE_TOKEN'", "role": "test"}' $VAULT_ADDR/v1/auth/kubernetes/login | jq '.auth.client_token' | awk -F\" '{print $2}')
```

- используя свой клиентский токен проверм чтение: 

```
curl --header "X-Vault-Token:s.aZ7PeufImmNIu1UEzFF6O7Pg" $VAULT_ADDR/v1/otus/otus-ro/config
curl --header "X-Vault-Token:s.aZ7PeufImmNIu1UEzFF6O7Pg" $VAULT_ADDR/v1/otus/otus-rw/config
```

- и проверим запись: 

```
curl --request POST --data '{"bar": "baz"}' --header "X-Vault-Token:s.aZ7PeufImmNIu1UEzFF6O7Pg" $VAULT_ADDR/v1/otus/otus-ro/config
curl --request POST --data '{"bar": "baz"}' --header "X-Vault-Token:s.aZ7PeufImmNIu1UEzFF6O7Pg" $VAULT_ADDR/v1/otus/otus-rw/config
curl --request POST --data '{"bar": "baz"}' --header "X-Vault-Token:s.aZ7PeufImmNIu1UEzFF6O7Pg" $VAULT_ADDR/v1/otus/otus-rw/config1
```

- Разобраться с ошибками при записи: 
  - Почему мы смогли записать otus-rw/config1 но не смогли otusrw/config - не удалось получить ошибку


#### Тест использования vault, развертывание nginx
- перейти в директорию `cd vault-guides/identity/vault-agent-k8s-demo` и выполнить команды: 

```
# Create a ConfigMap, example-vault-agent-config
$ kubectl create configmap example-vault-agent-config --from-file=./configs-k8s/

# View the created ConfigMap
$ kubectl get configmap example-vault-agent-config -o yaml

# Finally, create vault-agent-example Pod
$ kubectl apply -f example-k8s-spec.yml --record

```

- проверим index.html в поде `vault-agent-example` (в контейнере `nginx-container`), что в него записались секреты;

#### Cоздадим CA на базе vault
- включим pki сикретс 

```
kubectl exec -it vault-0 -- vault secrets enable pki
kubectl exec -it vault-0 -- vault secrets tune -max-lease-ttl=87600h pki
kubectl exec -it vault-0 -- vault write -field=certificate pki/root/generate/internal \
  common_name="exmaple.ru" ttl=87600h > CA_cert.crt
```

- пропишем урлы для ca и отозванных сертификатов

```
kubectl exec -it vault-0 -- vault write pki/config/urls \
  issuing_certificates="http://vault:8200/v1/pki/ca" \
  crl_distribution_points="http://vault:8200/v1/pki/crl"
```

- создадим промежуточный сертификат

```
kubectl exec -it vault-0 -- vault secrets enable --path=pki_int pki
kubectl exec -it vault-0 -- vault secrets tune -max-lease-ttl=87600h pki_int
kubectl exec -it vault-0 -- vault write -format=json \
pki_int/intermediate/generate/internal \
common_name="example.ru Intermediate Authority" | jq -r '.data.csr' > pki_intermediate.csr
```

- пропишем промежуточный сертификат в vault

```
kubectl cp pki_intermediate.csr vault-0:/tmp

kubectl exec -it vault-0 -- vault write -format=json pki/root/sign-intermediate \
csr=@/tmp/pki_intermediate.csr \
format=pem_bundle ttl="43800h" | jq -r '.data.certificate' > intermediate.cert.pem

kubectl cp intermediate.cert.pem vault-0:/tmp

kubectl exec -it vault-0 -- vault write pki_int/intermediate/set-signed certificate=@/tmp/intermediate.cert.pem
```

- создадим роль для выдачи с ертификатов: 

```
kubectl exec -it vault-0 -- vault write pki_int/roles/example-dot-ru \
allowed_domains="example.ru" allow_subdomains=true max_ttl="720h"
```

- cоздадим и отзовем сертификат: 

```
kubectl exec -it vault-0 -- vault write pki_int/issue/example-dot-ru common_name="gitlab.example.ru" ttl="24h"
kubectl exec -it vault-0 -- vault write pki_int/revoke serial_number="6c:15:0c:26:9d:4c:52:ca:af:27:82:08:0e:a1:87:9a:65:cc:ed:ff"
```

Вывод консоли при выдаче сертификата: 

```
Key                 Value
---                 -----
ca_chain            [-----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIURPz2VD/gznUCeQQ63NBdL6/VxWgwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhtYXBsZS5ydTAeFw0xOTExMTAxNzA0MDhaFw0yNDEx
MDgxNzA0MzhaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAM6m2wfvUAdS
6tKL0HP8hC2a/K26yjYe6pNneIXoI+6BjXnSVAFuH1x5v8NBn9sgG183t2+JZWRs
4F05O9JtRTXxj6WA9gscsRMk3b3nNSo+ba27BhrHBTOf3Osmza4YA4K6lKQ1fGob
SEza0yEMLGJ/mrr8ZS0hmDSIgsJbZhqfr1vcH03J7Mf+d9JZoxU5fBy5zynyOo5X
ersaGvmhObtdWJG2hCDHYHkPEm5Vc3iNuF2Y2f3vrSkjpW2zADkdIoTm/Lse/CSt
qYpMX7YgVJW3ZHTE5v8bHEnP5TF0H1jKpr7QaBmX06rPWZXnFtK9Q+Yvl3KiPNPn
Hal/bYjwvgECAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQUCw9PjI3SxXt3qmfK3+SeXVzT+Z4wHwYDVR0jBBgwFoAU
T7KV0Bm/aN7mJ6MY4ZrRB4KqbykwNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
mmOh4THQiPmGO6nJmPsYBJVHL7O3GGru8RhrMK2yKmNaCjLtxcNEpumNCHu966TQ
e8h9FtGVpkbK++kYx5sb0LDE4VyI4WvJhxPwUJXaLRBAxmUAdSTvo8rmzHcV5Sq3
nXWcWB6yiIQUyKZkwoiHG70SVQyeWeEzOAxM3+p68indkJS0UhxeL7JRCk1Due3x
rT+U0A0eXqs4E2XeHTbANFy5zjOuuxVd7+Wnd3iQ14N8eaCd75f3hjLOa0lgFWSa
eqjNpoLr9lccLsmmDbQwheioDMz01Z4yr694s4BFpYZcngel0oSXbxth2OxgaWAV
yPGY0ZKqOrFQuUs8LTqyPA==
-----END CERTIFICATE-----]
certificate         -----BEGIN CERTIFICATE-----
MIIDZzCCAk+gAwIBAgIUbBUMJp1MUsqvJ4IIDqGHmmXM7f8wDQYJKoZIhvcNAQEL
BQAwLDEqMCgGA1UEAxMhZXhhbXBsZS5ydSBJbnRlcm1lZGlhdGUgQXV0aG9yaXR5
MB4XDTE5MTExMDE3MDgxMVoXDTE5MTExMTE3MDg0MVowHDEaMBgGA1UEAxMRZ2l0
bGFiLmV4YW1wbGUucnUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDj
RdnQLaAU+d4QezrCpt5v+84epgKqAOeteHDW9mfWI2PeURuRJkMFHEc1NPV4AcIk
qsdCHALXKbW9/ZNPO2TB2DwbLaL7ejSnJVrex2QFrzZoMDff27Twk2nOhpnmt6l4
xllI4l5IZ/PuFD6EtEiG7PkBXdsdf/6MsRvFIHv8sY+U0FvYR5xIA4ViFLNXv+Bf
UnW6VArbEZOKNm4QxqsypBL3zoydgDpoKwrnxbhC+D2EYC2Ffk26iIeSVqNHo45D
Bjfpmxw/bMpFBzAZ2IuptfrarzGhX/D0f/J/3RMle1aDNz6ysq4p3HHRdZLPVryK
0EofwwFxZQhDMyw6LE/ZAgMBAAGjgZAwgY0wDgYDVR0PAQH/BAQDAgOoMB0GA1Ud
JQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDAjAdBgNVHQ4EFgQU979knxz5FwSMY5lM
WJwlOcO/QsYwHwYDVR0jBBgwFoAUCw9PjI3SxXt3qmfK3+SeXVzT+Z4wHAYDVR0R
BBUwE4IRZ2l0bGFiLmV4YW1wbGUucnUwDQYJKoZIhvcNAQELBQADggEBACHF4cqY
Enqwp8bJSpO885SuQVpXg/PjBJ6JLd4241mr87AxhuJION369ulYBRQC/SW35SVs
rbPtNSgEG1AZUsob/jd6Iwpjw8MKejwdK+nbWb+gvOKoFBQIolfdC0DzFIaBAGQZ
7d13cpbbPNuAgBNngzZ3wI12ENq1KgstNp2xRpuzNpNkqYE4Jdby8ldPzRUEZW7/
z8PfXcilNQ6tUS0yjAhQs01m1i6Qmt22tsoGWBwoX/kELmu17vyVeEaTQn3RDbpL
o1il9uVhOjjjvBv59uWFy2v+aEh9qZFeU5W7ubV9vR4Bdc9iekLQdUy6i9xXzlwW
BtwNMyym1oKdRuA=
-----END CERTIFICATE-----
expiration          1573492121
issuing_ca          -----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIURPz2VD/gznUCeQQ63NBdL6/VxWgwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhtYXBsZS5ydTAeFw0xOTExMTAxNzA0MDhaFw0yNDEx
MDgxNzA0MzhaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAM6m2wfvUAdS
6tKL0HP8hC2a/K26yjYe6pNneIXoI+6BjXnSVAFuH1x5v8NBn9sgG183t2+JZWRs
4F05O9JtRTXxj6WA9gscsRMk3b3nNSo+ba27BhrHBTOf3Osmza4YA4K6lKQ1fGob
SEza0yEMLGJ/mrr8ZS0hmDSIgsJbZhqfr1vcH03J7Mf+d9JZoxU5fBy5zynyOo5X
ersaGvmhObtdWJG2hCDHYHkPEm5Vc3iNuF2Y2f3vrSkjpW2zADkdIoTm/Lse/CSt
qYpMX7YgVJW3ZHTE5v8bHEnP5TF0H1jKpr7QaBmX06rPWZXnFtK9Q+Yvl3KiPNPn
Hal/bYjwvgECAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQUCw9PjI3SxXt3qmfK3+SeXVzT+Z4wHwYDVR0jBBgwFoAU
T7KV0Bm/aN7mJ6MY4ZrRB4KqbykwNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
mmOh4THQiPmGO6nJmPsYBJVHL7O3GGru8RhrMK2yKmNaCjLtxcNEpumNCHu966TQ
e8h9FtGVpkbK++kYx5sb0LDE4VyI4WvJhxPwUJXaLRBAxmUAdSTvo8rmzHcV5Sq3
nXWcWB6yiIQUyKZkwoiHG70SVQyeWeEzOAxM3+p68indkJS0UhxeL7JRCk1Due3x
rT+U0A0eXqs4E2XeHTbANFy5zjOuuxVd7+Wnd3iQ14N8eaCd75f3hjLOa0lgFWSa
eqjNpoLr9lccLsmmDbQwheioDMz01Z4yr694s4BFpYZcngel0oSXbxth2OxgaWAV
yPGY0ZKqOrFQuUs8LTqyPA==
-----END CERTIFICATE-----
private_key         -----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA40XZ0C2gFPneEHs6wqbeb/vOHqYCqgDnrXhw1vZn1iNj3lEb
kSZDBRxHNTT1eAHCJKrHQhwC1ym1vf2TTztkwdg8Gy2i+3o0pyVa3sdkBa82aDA3
39u08JNpzoaZ5repeMZZSOJeSGfz7hQ+hLRIhuz5AV3bHX/+jLEbxSB7/LGPlNBb
2EecSAOFYhSzV7/gX1J1ulQK2xGTijZuEMarMqQS986MnYA6aCsK58W4Qvg9hGAt
hX5NuoiHklajR6OOQwY36ZscP2zKRQcwGdiLqbX62q8xoV/w9H/yf90TJXtWgzc+
srKuKdxx0XWSz1a8itBKH8MBcWUIQzMsOixP2QIDAQABAoIBACClegJDa4lX0yQ+
71PisHUZkKQqaJuPAbiTYnIedw/1iXT35aPWAS6Mv1XPQ6t3ZTHrLjA64dWScj7W
XAC3oWOO8iNdTNoe1c1kukbiEWYXoxMYSg5n+vfL1RkLkLPpkfh9VXn4ul5gQFPk
qI5bb0eiZqphlwYHysLe9gQ9BFJp0c/6yNjQzvz9oLTI0GfGox8mNrg5ftvaJLK8
ZlDloVr3ymRTp75+S9YGVS23aJMRYXVk00Z6hTcM6lVRuNxl+qxyezl77Kw/1hBp
dzxOOx0fXGUM/i7udT1AK7NxfnigDVi9JsVNVdvQc6xDLXLYDPQX+QLvC5SOONrH
HrfdruECgYEA8SxNNd0vAnCDZ6Ds2NORDyyBQupGyPPyHcuFdwuEXP4g++0M+OC9
7kYubPgfAQtKtPgVljAjVjN44Oh6kuiKAyOtJCNVEpItyoeXV8qHMb/ElQo1M4Ik
NHJPfOQKwgjclsy83bcfvE9rbiH0Ly0zyb3ysFqOhujd+zYBtmzKMZ0CgYEA8T7H
+tzdwmuGfN3GvYMRGPDdHOH+YVs2SZzxARntdGxINMne/6MkPVSrw9e0d7wck9bn
GVZk7ZYfcKaYSvFuvSWgHZ6YGogx1CtPxCNxIQrjfN0AkS/N0L+bvT845KkkYPLx
ZB/0wWrilqynd890XxS9e3O7Sk2F3tmGssw68G0CgYEAnNff2sDeuqprevB4N8bX
ltOtuNPddwDXG7NpN/NggI2w68XNYund+2De/nUazLYIPsr8VvE1efD9kt7+IB5k
6we/qTnlMK+qYgVuUmTfKWZ6tSavVLE1VHpm4WT47hmPQ+8ggNyAIhpQVo50XF38
SR5j/3bVLD2zZ5VG5dm4YS0CgYEAgk8vJkp3XrVGB9yjpWpOqfIw/ZD1HxFt2YV0
iOvAX8q9lgYU9nDg+l/qB/dT+/kYVqMWYZFRIyScBvV1/cU709+nBVjNQEeg4sIi
bAfY68g96QxXahUwTzmwniCwUpMqm1OfID5CrtdVXZ4VN5pPeaxyTWTOHeySCzXk
lF/M1mECgYAWLF81xKRlE8PoyefUmcZjNaVEJw7to3La9A4jqaV5yZ88H36AyosV
6EiL1UwkES/AOOw0PWnEYu6xscuWW4xJ5VqI0SKwZ/YQL/27OviXGjhaDnIflqdx
nnYGoPRuZ5cPtJYF5VVRhELFEY9UvqRGMCJnC1eFTKTEf5or1zbDzA==
-----END RSA PRIVATE KEY-----
private_key_type    rsa
serial_number       6c:15:0c:26:9d:4c:52:ca:af:27:82:08:0e:a1:87:9a:65:cc:ed:ff
```

Вывод консоли при отзыве сертификата: 

```
Key                        Value
---                        -----
revocation_time            1573405777
revocation_time_rfc3339    2019-11-10T17:09:37.074167903Z
```

