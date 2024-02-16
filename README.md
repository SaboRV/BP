## Цель домашнего задания
- Научиться самостоятельно развернуть сервис NFS и подключить к нему клиента
## Описание домашнего задания
Настроить стенд Vagrant с двумя виртуальными машинами: backup_server и client. (Студент самостоятельно настраивает Vagrant)
Настроить удаленный бэкап каталога /etc c сервера client при помощи borgbackup. Резервные копии должны соответствовать следующим критериям:
директория для резервных копий /var/backup. Это должна быть отдельная точка монтирования. В данном случае для демонстрации размер не принципиален, достаточно будет и 2GB; (Студент самостоятельно настраивает)
репозиторий для резервных копий должен быть зашифрован ключом или паролем - на ваше усмотрение;
имя бэкапа должно содержать информацию о времени снятия бекапа;
глубина бекапа должна быть год, хранить можно по последней копии на конец месяца, кроме последних трех. Последние три месяца должны содержать копии на каждый день. Т.е. должна быть правильно настроена политика удаления старых бэкапов;
резервная копия снимается каждые 5 минут. Такой частый запуск в целях демонстрации;
написан скрипт для снятия резервных копий. Скрипт запускается из соответствующей Cron джобы, либо systemd timer-а - на ваше усмотрение;
настроено логирование процесса бекапа. Для упрощения можно весь вывод перенаправлять в logger с соответствующим тегом. Если настроите не в syslog, то обязательна ротация логов.

## Решение
Запускаем vagrant. Устанавливаем две ОС - ubuntu-server-2204-lts
- backup-server
- client-server

### Настройка backup-server
#### Устанавливаем на backup сервере borgbackup
apt update
apt install borgbackup
Добавляем новый диск размером 4Гб. Создаем и форматируем.
Создаем папку root@backup-server:~# mkdir /var/backup
Монтируем диск root@backup-server:~# mkdir /var/backup
root@backup-server:~# df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              390M  1.1M  389M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  122G  7.7G  108G   7% /
tmpfs                              2.0G     0  2.0G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/sda2                          2.0G  131M  1.7G   8% /boot
/dev/sdb1                          3.9G  3.5M  3.7G   1% /var/backup
/dev/sda1                          1.1G  6.1M  1.1G   1% /boot/efi
vagrant                             98G   47G   51G  49% /vagrant
tmpfs                              390M  4.0K  390M   1% /run/user/1000

Добавляем строчку в /etc/fstab:
/dev/sdb1 /var/backup ext4 defaults 1 2

Создаем пользователя borg:
useradd -m borg

Права на папку архивирования:
root@backup-server:~# chown borg:borg /var/backup/

На сервер backup создаем каталог ~/.ssh/authorized_keys в каталоге /home/borg
	# su - borg
	# mkdir .ssh
   	# touch .ssh/authorized_keys
        # chown -R borg:borg ~borg/.ssh

#### Устанавливаем на сервере Client borgbackup
apt update
apt install borgbackup

Для доступа к серверу backup-server необходимо сгенерировать SSH-ключ, так как регламентные задания выполняются от имени суперпользователя, то ключи должны быть созданы именно для него. Поэтому повысим себе права до root, выполните: 
sudo -i
ssh-keygen
После создания ключа берем содержимое id_rsa.pub 

##### На сервере backup-server выполняем команду:
echo 'command="/usr/bin/borg serve" ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCnuGn8HY05hslRb3RoqzBT0VXaSNSNTQBqgJtcAS3RvqyOv1dAClc9Hcq9cGzh/1tI/fFxei/VfS1k9fsgRh/6W06QuycTmrpn29peq90CICk0vIHuzFigwJr+IY8pNJooS6lzRMhkNJWzqr3i9gTUL+rncTTl2zDoKXHuuUC1bK7Moq5N50PdE3M9qisq9kbDx2TKfjtRCn5YUPdRcqG/7TPo7NgqTmmJBDAP/0WZfsw1YojDIvrB+7nG5HGmQ1VhZR5S8ue5yULDchb97585KyNU9xzjOLOv5TvOju+aSIXgqPJVc5Yq5CxF1kCQSbm+2yGg8LDadDbOFGbs2TC/sOVO5QAIlYIRtNXYJ/zrhEl8vWQ7ziK1lZGrEnwJh0+K1cMBlZyk6oyuOQD2GfCjvoOAe6wvAU1/a5m5qt1TU6HdCO0pf33ZI0Yz1h2sFfdC7Bvc9ulnvfGMQ+6zMrcwymuoecobusu3sQxp9oBseLEaznefHCHePbf2SA7n9OU=' >> ~borg/.ssh/authorized_keys

Возвращаемся на сервер Client.

Инициализируем репозиторий borg на backup сервере с client сервера:
root@server-client:~# borg init --encryption=repokey borg@192.168.56.160:/var/backup/

Запускаем для проверки создания бэкапа
root@server-client:~# borg create --stats --list borg@192.168.11.160:/var/backup/::"etc-{now:%Y-%m-%d_%H:%M:%S}" /etc

Смотрим, что у нас получилось

root@server-client:~# borg list borg@192.168.56.160:/var/backup
Enter passphrase for key ssh://borg@192.168.56.160/var/backup: 
Enter passphrase for key ssh://borg@192.168.56.160/var/backup: 
Enter passphrase for key ssh://borg@192.168.56.160/var/backup: 
etc-2024-02-15_17:49:31              Thu, 2024-02-15 22:49:35 [ebbf510827973f8cce1d855d09e493e28ab85ea6804b11221fabf375aac19419]
etc-2024-02-15_23:57:36              Thu, 2024-02-15 23:57:36 [c83721a89e0d282f32733d0c88950c5ca52a6769feeff499e3ef827841b00cb6]
etc-2024-02-16_15:43:06              Fri, 2024-02-16 15:43:07 [e8bc65db11c911c39bd61ff2d57b310d5512defc299ab62b124cb490f2235c3a]


Автоматизируем создание бэкапов с помощью systemd
Создаем сервис и таймер в каталоге /etc/systemd/system/

Сервис:

##### /etc/systemd/system/borg-backup.service
[Unit]
Description=Borg Backup

[Service]
Type=oneshot

# Парольная фраза
Environment="BORG_PASSPHRASE=borg"
# Репозиторий
Environment="REPO=borg@192.168.56.160:/var/backup/"
# Что бэкапим
Environment="BACKUP_TARGET=/etc"
# Создание бэкапа
ExecStart=/bin/borg create \
    --stats                \
    ${REPO}::etc-{now:%%Y-%%m-%%d_%%H:%%M:%%S} ${BACKUP_TARGET}

# Проверка бэкапа
ExecStart=/bin/borg check ${REPO}

# Очистка старых бэкапов
ExecStart=/bin/borg prune \
    --keep-daily  90      \
    --keep-monthly 12     \
    --keep-yearly  1       \
    ${REPO}

Таймер:

##### /etc/systemd/system/borg-backup.timer

[Unit]
Description=Borg Backup

[Timer]
OnUnitActiveSec=5m

[Install]
WantedBy=timers.target


#### Включаем и запускаем службу таймера
systemctl enable borg-backup.timer 
systemctl start borg-backup.timer
systemctl start borg-backup.service

Проверяем работу таймера

root@server-client:~# systemctl list-timers --all
NEXT                        LEFT          LAST                        PASSED       UNIT                         ACTIVATES                     
Fri 2024-02-16 20:51:37 +05 3min 16s left Fri 2024-02-16 20:46:24 +05 1min 56s ago borg-backup.timer            borg-backup.service
Fri 2024-02-16 21:13:02 +05 24min left    Fri 2024-02-16 12:07:01 +05 8h ago       fwupd-refresh.timer          fwupd-refresh.service
Sat 2024-02-17 00:00:00 +05 3h 11min left n/a                         n/a          dpkg-db-backup.timer         dpkg-db-backup.service
Sat 2024-02-17 00:00:00 +05 3h 11min left Fri 2024-02-16 00:00:02 +05 20h ago      logrotate.timer              logrotate.service
Sat 2024-02-17 02:44:46 +05 5h 56min left Fri 2024-02-16 17:04:14 +05 3h 44min ago motd-news.timer              motd-news.service
Sat 2024-02-17 07:10:13 +05 10h left      Fri 2024-02-16 19:48:14 +05 1h 0min ago  man-db.timer                 man-db.service
Sat 2024-02-17 20:00:40 +05 23h left      Fri 2024-02-16 20:00:40 +05 47min ago    systemd-tmpfiles-clean.timer systemd-tmpfiles-clean.service
Sun 2024-02-18 03:10:13 +05 1 day 6h left Thu 2024-02-15 20:42:14 +05 24h ago      e2scrub_all.timer            e2scrub_all.service
Mon 2024-02-19 01:38:58 +05 2 days left   Thu 2024-02-15 21:19:34 +05 23h ago      fstrim.timer                 fstrim.service
n/a                         n/a           n/a                         n/a          apport-autoreport.timer      apport-autoreport.service
n/a                         n/a           n/a                         n/a          snapd.snap-repair.timer      snapd.snap-repair.service
n/a                         n/a           n/a                         n/a          ua-timer.timer               ua-timer.service

12 timers listed.


Проверяем список бекапов:
root@server-client:~# borg list borg@192.168.56.160:/var/backup
Enter passphrase for key ssh://borg@192.168.56.160/var/backup: 
etc-2024-02-15_17:49:31              Thu, 2024-02-15 22:49:35 [ebbf510827973f8cce1d855d09e493e28ab85ea6804b11221fabf375aac19419]
etc-2024-02-15_23:57:36              Thu, 2024-02-15 23:57:36 [c83721a89e0d282f32733d0c88950c5ca52a6769feeff499e3ef827841b00cb6]
etc-2024-02-16_20:46:37              Fri, 2024-02-16 20:46:38 [7f0e4b2aaa214382a2ad18ed378cc8c193b4a2328759433f8a12b581962fd000]

Проверим восстановление файлов:

Удалим в каталоге /etc файл legal
root@server-client:~# rm /etc/legal 

Восстановим этот файл из вчерашнего архива:

root@server-client:/# sudo borg extract --list borg@192.168.56.160:/var/backup/::etc-2024-02-15_17:49:31 /etc/legal
Enter passphrase for key ssh://borg@192.168.56.160/var/backup: 
etc/legal

Файл восстановился.
























