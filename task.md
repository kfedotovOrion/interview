# Ожидаемый результат

1.	Процедура установки переведена на русский язык
2.	Процедура установки утилиты понятна для пользователя незнакомого с кубами
3.	Процедура установки не содержит грамматических и логических ошибок
4.	Процедура установки корректна с точки зрения применения
5.	Желательный выходной формат документа: asciidoc

# Исходный текст

1.	Set SELinux to permissive mode:

These instructions are for Kubernetes 1.33

```
$ setenforce .
$ sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

Warning:

-	Setting ELinux in permissive mode by running setenforce 0 and sed ... effectively disables it. This is required to allow containers to access the host filesystem; for example, some cluster network plugins require that. You have to do this until ELinux support is improved in the kubelet.
-	You can leave ELinux enabled if you know how to configure it but it may require settings that are not supported by kubeadm.

2.	Add the Kubernetes yum repository. The exclude parameter in the repository definition ensures that the packages related to Kubernetes are not upgraded upon running yum update as there's a special procedure that must be followed for upgrading Kubernetes. Please note that this repository have packages only for Kubernetes 1.33; for other Kubernetes minor versions, you need to change the Kubernetes minor version in the URL to match your desired minor version (you should also check that you are reading the documentation for the version of Kubernetes that you plan to install).

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

3.	Install kubelet, kubeadm and kubectl:

```
$ dfn install -y kubelet kubeadm kubect --disableexcludes=kubernetes
```

4.	(Optional) Enable the kubelet service before running kubeadm:

```
$ system enable --now kubelet
```
