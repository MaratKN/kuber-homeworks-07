# Домашнее задание к занятию «Хранение в K8s. Часть 2»

### Цель задания

В тестовой среде Kubernetes нужно создать PV и продемострировать запись и хранение файлов.

------

### Чеклист готовности к домашнему заданию

1. Установленное K8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключенным GitHub-репозиторием.

------

### Дополнительные материалы для выполнения задания

1. [Инструкция по установке NFS в MicroK8S](https://microk8s.io/docs/nfs). 
2. [Описание Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/). 
3. [Описание динамического провижининга](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/). 
4. [Описание Multitool](https://github.com/wbitt/Network-MultiTool).

------

### Задание 1

**Что нужно сделать**

Создать Deployment приложения, использующего локальный PV, созданный вручную.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.
2. Создать PV и PVC для подключения папки на локальной ноде, которая будет использована в поде.
3. Продемонстрировать, что multitool может читать файл, в который busybox пишет каждые пять секунд в общей директории. 
4. Удалить Deployment и PVC. Продемонстрировать, что после этого произошло с PV. Пояснить, почему.
5. Продемонстрировать, что файл сохранился на локальном диске ноды. Удалить PV.  Продемонстрировать что произошло с файлом после удаления PV. Пояснить, почему.
5. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

Создадим Deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-test
  namespace: default
  labels:
    app: dep-test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: dep-test
  template:
    metadata:
      labels:
        app: dep-test
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ['sh', '-c', 'while true; do echo Hello world! >> /output/output.txt; sleep 5; done']
        volumeMounts:
        - name: pv-test
          mountPath: /output
      - name: multitool
        image: wbitt/network-multitool:latest
        ports:
        - containerPort: 80
        env:
        - name: HTTP_PORT
          value: "80"
        volumeMounts:
        - name: pv-test
          mountPath: /input
      volumes:
      - name: pv-test
        persistentVolumeClaim:
          claimName: pvc-test
```

Создадим PV

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-local
spec:
  storageClassName: host-path
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: /root/data/pv-test
  persistentVolumeReclaimPolicy: Retain
```

Создадим PVC
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-local
spec:
  storageClassName: host-path
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Продемонстрируем, что multitool может читать файл, в который busybox пишет каждые пять секунд в общей директории.

```
root@DebianNew:~/.kube# kubectl exec pods/deployment-test-9579474fb-pdsxb -c multitool -- cat /input/output.txt
```
![alt text](https://github.com/MaratKN/kuber-homeworks-07/blob/main/1.png)

Удаляем deploy и pvc


После удаления PVC, PV сменил статус с Bound на Released, так как больше не связан с PVC

![alt text](https://github.com/MaratKN/kuber-homeworks-07/blob/main/2.png)

Продемонстрируем, что файл сохранился на локальном диске ноды

![alt text](https://github.com/MaratKN/kuber-homeworks-07/blob/main/3.png)

Удалим PV. Файл сохранился на ноде, так как Reclaim Policy мы установили равным Retain

![alt text](https://github.com/MaratKN/kuber-homeworks-07/blob/main/4.png)

------

### Задание 2

**Что нужно сделать**

Создать Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.

1. Включить и настроить NFS-сервер на MicroK8S.
2. Создать Deployment приложения состоящего из multitool, и подключить к нему PV, созданный автоматически на сервере NFS.
3. Продемонстрировать возможность чтения и записи файла изнутри пода. 
4. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

Включим и настроим NFS-сервер на MicroK8S.

microk8s enable community

microk8s enable nfs

sudo apt update && sudo apt install nfs-kernel-server

Так и не осилил установку и настройку NFS-сервера. 

По вашей ссылке https://github.com/netology-code/kuber-homeworks/blob/main/2.2/2.2.md на инструкцию падает 404.

Пытался по вашему видео сделать, с какими-то падениями и костылями что-то установил, но...

```root@DebianNew:~/.kube# kubectl get storageclasses
NAME      PROVISIONER        RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
csi-s3    ru.yandex.s3.csi   Delete          Immediate           false                  5m2s
```

Да у вас даже лектор не с первого раза это всё поставил и сам долго мучался и вспоминал. 

Просьба помочь с установкой и настройкой NFS, с самого нуля, максимально подробно и по шагам.

Вводные данные, если нужны: у меня Кубер на тачке яндекса (Ubuntu), работаю с VirtualBox (Debian)

------

### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.