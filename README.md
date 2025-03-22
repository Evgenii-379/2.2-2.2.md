# **Домашнее задание к занятию «Хранение в K8s. Часть 2»**-***Вуколов Евгений***
 
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
5. Продемонстрировать, что файл сохранился на локальном диске ноды. Удалить PV.  Продемонстрировать что произошло с файлом после удаления PV. Пояснить, почему.
5. Предоставить манифесты, а также скриншоты или вывод необходимых команд.
 
------
 
### Задание 2
 
**Что нужно сделать**
 
Создать Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.
 
1. Включить и настроить NFS-сервер на MicroK8S.
2. Создать Deployment приложения состоящего из multitool, и подключить к нему PV, созданный автоматически на сервере NFS.
3. Продемонстрировать возможность чтения и записи файла изнутри пода. 
4. Предоставить манифесты, а также скриншоты или вывод необходимых команд.
 
------
 
### Правила приёма работы
 
1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.



# **Решение**

## **Задание 1**

- Для первого задания создаю namespace netology-pvc:

- ![scrin](https://github.com/Evgenii-379/2.2-2.2.md/blob/main/Снимок%20экрана%202025-03-21%20231559.png)

- В этом namespace cоздал Deployment приложения, состоящего из контейнеров busybox и multitool, а также PV и PVC для подключения папки на локальной ноде, которая будет использована в поде:

- ![scrin](https://github.com/Evgenii-379/2.2-2.2.md/blob/main/Снимок%20экрана%202025-03-22%20111248.png)

- Демонстрация, что multitool может читать файл, в который busybox пишет каждые пять секунд в общей директории:

- ![scrin](https://github.com/Evgenii-379/2.2-2.2.md/blob/main/Снимок%20экрана%202025-03-22%20113807.png)

- Удаляю Deployment и PVC. Демонстрация, что после этого произошло с PV:

- ![scrin](https://github.com/Evgenii-379/2.2-2.2.md/blob/main/Снимок%20экрана%202025-03-22%20114905.png)

PV остается в системе, но его статус изменится на Released, что означает, что он больше не связан с PVC и не доступен для новых PVC, пока не будет вручную удален или очищен.
PV не удаляется автоматически из-за политики Retain, которая позволяет вручную управлять ресурсами после удаления PVC. Это полезно для сохранения важных данных и предотвращения случайного удаления. 
Чтобы использовать PV снова, необходимо либо удалить его и создать заново, либо вручную очистить данные на нем.

- ![scrin](https://github.com/Evgenii-379/2.2-2.2.md/blob/main/Снимок%20экрана%202025-03-22%20121451.png)

После удаления PV, файл в папке /mnt/data сохраняется. Это происходит потому, что PV просто ссылался на эту локальную папку на ноде, а не управлял ею напрямую. 
Удаление PV не приводит к удалению данных в папке, на которую он ссылался.

- Ссылки на манифесты : 

[busybox-multitool-pvc.yaml](https://github.com/Evgenii-379/2.2-2.2.md/blob/main/config.yaml/busybox-multitool-pvc.yaml)

[pvc.yaml](https://github.com/Evgenii-379/2.2-2.2.md/blob/main/config.yaml/pvc.yaml)

[pv.yaml](https://github.com/Evgenii-379/2.2-2.2.md/blob/main/config.yaml/pv.yaml)


## **Задание 2**

- Для этого задания создаю namespace netology-nfs, скачиваю необходимые пакеты для установки nfs и устанавливаю:

- ![scrin](https://github.com/Evgenii-379/2.2-2.2.md/blob/main/Снимок%20экрана%202025-03-22%20124239.png)

- Настройка NFS :

```
1. Установка пакета NFS-сервера :

sudo apt-get update
sudo apt-get install nfs-kernel-server

2. Создание директории для NFS :

sudo mkdir -p /var/snap/microk8s/common/nfs-storage
sudo chown nobody:nogroup /var/snap/microk8s/common/nfs-storage
sudo chmod 0777 /var/snap/microk8s/common/nfs-storage

3. Настройка экспорта :

sudo nano /etc/exports

- Добавляю строку в файл: 

/var/snap/microk8s/common/nfs-storage 158.160.1.229(rw,sync,no_subtree_check)
 
- Сохранить и закрыть

4. Перезапуск NFS-сервера :

sudo exportfs -ra

5. Запуск сервера NFS : 

sudo systemctl start nfs-server

6. Автозапуск : 

sudo systemctl enable nfs-server

7. Проверка состояния NFS-сервера :

sudo systemctl status nfs-server

8. Проверка доступности NFS :

showmount -e 158.160.1.229

```

- Создаю Deployment приложения состоящего из multitool, и подключаю к нему PV, созданный автоматически на сервере NFS:

- ![scrin](https://github.com/Evgenii-379/2.2-2.2.md/blob/main/Снимок%20экрана%202025-03-22%20144708.png)

- ![scrin](https://github.com/Evgenii-379/2.2-2.2.md/blob/main/Снимок%20экрана%202025-03-22%20143451.png)

- Командой вывожу название пода, затем название пода вставляю в команду для того чтобы продемонстрировать возможность чтения и записи файла изнутри пода:

- ![scrin](https://github.com/Evgenii-379/2.2-2.2.md/blob/main/Снимок%20экрана%202025-03-22%20144036.png)

- Ссылки на манифесты : 

[multitool-nfs.yaml](https://github.com/Evgenii-379/2.2-2.2.md/blob/main/config.yaml/multitool-nfs.yaml)

[nfs-sc.yaml](https://github.com/Evgenii-379/2.2-2.2.md/blob/main/config.yaml/nfs-sc.yaml)

[pvc-nfs.yaml](https://github.com/Evgenii-379/2.2-2.2.md/blob/main/config.yaml/pvc-nfs.yaml)












































