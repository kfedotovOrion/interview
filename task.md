# Ожидаемый результат

1.	Процедура установки переведена на русский язык
2.	Процедура установки утилиты понятна для пользователя незнакомого с кубами
3.	Процедура установки не содержит грамматических и логических ошибок
4.	Процедура установки корректна с точки зрения применения
5.	Желательный выходной формат документа: asciidoc

# Исходный текст

1.	Переключите SELinux в пермиссив-режим:

*Этапы инструкции, приведенные ниже, основаны на Kubernetes 1.33*

```
$ setenforce .
$ sed -i 's/^SELINUX=enforcing$/SELINUX=permissiv/' /etc/selinux/config
```

:::warning
-	Переключение SELinux в пермиссив-режим через команды `setenforce 0` и `sed ...` деактивирует его. Это требуется, чтобы контейнеры могли получить доступ к файл-системе хоста; например, это требуется для работы с некоторыми кластерными сетевыми плагинами. Это необходимо делать, пока поддержка SELinux не будет улучшена в **kubelet**.
-	Вы можете отсавить SELinux активным, если знаете как его конфигурировать, однако для него могут потребоваться настройки, неподдерживаемые **kubeadm**.
:::

2.	Добавьте Kubernetes yum-репозиторий. Параметр `exclude` в конфигурационном .repo файле репозитория блокирует пакеты, относящиеся к Kubernetes для обновлений через команду `yum update`. Апгрейд Kubernetes должен проводиться по специальной инструкции. 

:::info
Обратите внимание, что в этом репозитории пакеты только для Kubernetes 1.33. Для других минорных версий Kubernetes, нужно заменить минорную версию Kubernetes в URL на требующуюся (v1.xx), а так же убедиться, что у вас есть инструкция для соответсвующей версии Kubernetes.
:::

```
$ cat <<EOF tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.33/rpm/
enabled=0
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.33/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF
```

3.	Установите kubelet, kubeadm and kubectl:

```
$ dnf install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```

4.	(Опционально) Подключите сервис kubelet прежде, чем запустить kubeadm:

```
$ system enable --now kubelet
```
