# Задача 1.

1. Переведите следующий текст на русский язык.
2. Оформите структуру в формате asciidoc или markdown

Исходный текст:

Virtualization Terminology 

Cluster - A cluster is a set of physical hosts that are treated as a resource pool for virtual machines. Hosts in a cluster share the same network infrastructure and storage. They form a migration domain within which virtual machines can be moved from host to host.
Data Center - A data center is the highest level container for all physical and logical resources within a managed virtual environment. It is a collection of clusters, virtual machines, storage domains, and networks.
Events - Alerts, warnings, and other notices about activities help the administrator to monitor the performance and status of resources.
HA Services - The HA services include the ovirt-ha-agent service and the ovirt-ha-broker service. The HA services run on self-hosted engine nodes and manage the high availability of the Manager virtual machine.
High Availability - High availability means that a virtual machine is automatically restarted if its process is interrupted, either on its original host or another host in the cluster. Highly available environments involve a small amount of downtime, but have a much lower cost than fault tolerance, which maintains two copies of each resource so that one can replace the other immediately in the event of a failure.

# Задача 2

1. Текст должен быть изложен в научно-техническом стиле.
2. Улучшите читебельность и чистоту текста.
3. Исправьте любые обнаруженные ошибки (грамматические, логические, синтаксические).

Исходный текст:

## Настройка BIND как кэширующего DNS-сервера на CentOS


По умолчанию DNS-сервер BIND разрешает и кэширует успешные и неудачные запросы. Затем служба отвечает на запросы к тем же записям из своего кэша. Это значительно повышает скорость DNS-запросов.

### Предпосылки

-- IP-адрес сервера статический.

### Процедура

1. Установите пакеты bind и bind-utils:
```
$ apt install bind bind-utils
```
Если вы хотите запустить BIND в среде change-root, установите bind-chrootпакет:
```
$ apt install bind-chroot
```
NOTE:::Обратите внимание, что запуск BIND на хосте с SELinux в enforcing режиме, который используется по умолчанию, более безопасен:::

2. Отредактируйте /etc/named.conf файл и внесите следующие изменения в options заявление:
2.1. Обновите операторы listen-on и listen-on-v6, чтобы указать, какие интерфейсы IPv4 и IPv6 должен прослушивать BIND:
```
listen-on port 53 { 127.0.0.1; 192.0.2.1; };
listen-on-v6 port 53 { ::1; 2001:db8:1::1; };
``
2.2. Обновите allow-queryоператор, чтобы настроить, с каких IP-адресов и диапазонов клиенты могут запрашивать этот DNS-сервер:
``
allow-query { localhost; 192.0.2.0/24; 2001:db8:1::/64; };
```
2.3. Добавьте allow-recursionоператор, чтобы определить, с каких IP-адресов и диапазонов BIND принимает рекурсивные запросы:

allow-recursion { localhost; 192.0.2.0/24; 2001:db8:1::/64; };

3. Проверьте синтаксис файла /etc/named.conf:
```
$ named-checkconf
```
Если команда не выводит никаких результатов, синтаксис правильный.

4. Обновите firewalld правила, чтобы разрешить входящий DNS-трафик:
```
$ firewall-cmd permanent --add-service=dns
$ firewall-cmd reload
```
6. Запустите и добавьте в автозагрузку BIND:
```
$ systemctl start named
```

### Проверка

Используйте недавно настроенный DNS-сервер для разрешения домена:
```
$ dig @localhost www.example.org +noall
...
www.example.org.    86400    IN    A    198.51.100.34

;; Query time: 917 msec
...
```
Дополнительные ресурсы:

-- (Запись SOA в файлах зоны)[https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/managing_networking_infrastructure_services/setting-up-and-configuring-a-bind-dns-server#the-soa-record-in-zone-files]


# Задача 3

1. Сконвертируйте следующий текст из markdown в asciidoc. Несмотря на то, что asciidoc поддерживает синтаксис markdown, в этой задаче необходимо использовать строгий синтаксис asciidoc.
2. Исправьте любые обнаруженные ошибки (грамматические, логические, синтаксические).
3. Текст должен быть изложен в научно-техническом стиле.

Исходный текст:

## Создать статический модуль

note:::
Статический Pod можно настроить либо с помощью файла конфигурации, размещенного в файловой системе , либо с помощью файла конфигурации, размещенного в Интернете .:::

### Статический манифест Pod, размещенный в файловой системе

_Манифесты_ — это стандартные определения модулей в формате JSON или YAML в определённом каталоге. Используйте это `staticPodPath: <the directory>` поле в файле конфигурации kubelet , который периодически сканирует каталог и создаёт/удаляет статические модули по мере появления/исчезновения в нём файлов YAML/JSON. Обратите внимание, что при сканировании указанного каталога kubelet будет игнорировать файлы, начинающиеся с точек.

Например, вот как запустить простой веб-сервер как статический Pod:

1. Выберите узел, на котором вы хотите запустить статический модуль. В данном примере это my-node1.
```
$ ssh my-node1
```
2. Выберите каталог _/etc/kubernetes/manifests_ и поместите туда определение Pod веб-сервера, например /etc/kubernetes/manifests/static-web.yaml:
```
# Run this command on the node where kubelet is running
mkdir -p /etc/kubernetes/manifests/
cat <<EOF >/etc/kubernetes/manifests/static-web.yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-web
  labels:
    role: myrole
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
          protocol: TCP
EOF
```
3. Настройте Kubelet на этом узле, чтобы задать staticPodPath значение в файле конфигурации Kubelet.
note:::
Альтернативный и устаревший метод — настроить kubelet на этом узле для локального поиска статических манифестов Pod с помощью аргумента командной строки. Чтобы использовать устаревший подход, запустите kubelet с этим `--pod-manifest-path=/etc/kubernetes/manifests/` аргументом.

4. Перезапустите kubelet. В Fedora выполните:
```
# Run this command on the node where the kubelet is running
$ systemctl restart kubelet
```
  Подробнее см. по [ссылке](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)

![Концептуальная схема](https://kubernetes.io/images/docs/components-of-kubernetes.svg)
