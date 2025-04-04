# Ubuntu questions and answers

## Running a script as a service
Let's say you have a script which normally you run using `./myscript.sh` *(contains java run command)*. You want to make it as a service, so it runs automatically after machine restart and etc. 

You need to [write systemd service file](https://patrakov.blogspot.com/2011/01/writing-systemd-service-files.html).

Simplest script looks like this:

```bash
[Unit]
Description=Virtual Distributed Ethernet

[Service]
ExecStart=/usr/bin/YOUR_SCRIPT

[Install]
WantedBy=multi-user.target
```

Also you need to execute daemon-reload after creating new service.

```bash
systemctl daemon-reload
```

## Schedule a nightly reboot

Edit crontab:

```bash
crontab -e
```

The first time you might have to choose your preferred editor (like nano).  
Generate your cron with [crontab generator](https://crontab.cronhub.io/) and add restart command right after it:

```bash
0 4 * * * /sbin/shutdown -r +5
```

at the bottom. Explanation:

```bash
m      h    dom        mon   dow       command
minute hour dayOfMonth Month dayOfWeek commandToRun
```

so it would announce the reboot every day at 4:00am, then reboot 5 minutes later (at 4:05am).

Ctrl+X, Y, Enter should get you out of crontab (if using nano)

!!! note
    you might have to run `crontab -e` as **root**, because shutdown needs **root**. `crontab -e` opens a file in `/tmp` instead of the actual crontab so that it can check your new crontab for errors. If there are no errors, then your actual crontab will be updated.

## Removing old or unused docker images

Docker 1.13: [PR 26108](https://github.com/docker/docker/pull/26108) and [commit 86de7c0](https://github.com/docker/docker/commit/86de7c000f5d854051369754ad1769194e8dd5e1) introduce a few new commands to help facilitate visualizing how much space the docker daemon data is taking on disk and allowing for easily cleaning up "unneeded" excess.

[docker system prune](https://docs.docker.com/engine/reference/commandline/system_prune/) will delete all dangling data (containers, networks, and images). You can remove all unused volumes with the `--volumes` option and remove all unused images (not just dangling) with the `-a` option.

You also have:
- [docker container prune](https://docs.docker.com/engine/reference/commandline/container_prune/)
- [docker image prune](https://docs.docker.com/engine/reference/commandline/image_prune/)
- [docker network prune](https://docs.docker.com/engine/reference/commandline/network_prune/)
- [docker volume prune](https://docs.docker.com/engine/reference/commandline/volume_prune/)

For unused images, use `docker image prune -a` (for removing dangling and ununsed images).
Warning: 'unused' means "images not referenced by any container": be careful before using `-a`.

As illustrated in [A L](https://stackoverflow.com/users/1207596/a-l)'s [answer](https://stackoverflow.com/a/50405599/6309), `docker system prune --all` will remove all unused images not just dangling ones... which can be a bit too much.

## Cleaning your server

Анализируем занятое дисковое пространство
Для анализа занимаемого пространства на диске можно использовать команду:

```bash
df -h
```

Данная команда выведет заполненность каждого раздела на диске. Нас интересует корневой раздел (/), именно его и стоит учитывать во время анализа занимаемого пространства.

Для более продвинутого анализа пространства можно использовать утилиту **ncdu**, у которой есть интерфейс, напоминающий чем-то утилиту [WinDirStat](https://windirstat.net/), которая позволит вам гулять по директориям вашего сервера и смотреть файлы, которые занимают больше всего места.

!!! warning
    Будьте осторожны при ручном удалении файлов!

### Installation

!!! note
    Если у вас нет команды sudo или вы находитесь под учеткой root, писать ее не нужно

``bash
sudo apt install ncdu
```

### Usage

!!! note
    Данная команда выполнит сканирование в директории /. Если у вас нет команды sudo или вы находитесь под учеткой root, писать ее не нужно

``bash
sudo ncdu /
```

После непродолжительного сканирования утилита выведет список файлов и директорий, занимающих пространство на вашем диске, в порядке убывания.

Пространство проанализировали, что дальше? 

Самая жирная директория обычно это /var, и для ее очистки есть несколько путей:

a) Очистка старых логов (/var/log)

В том случае, если у вас накопилось много логов, которые вам не нужны, вы можете выполнить команду удаления логов, встроенную в дистрибутив:

```bash
sudo journalctl --vacuum-time=2d
```

Вместо 2d вы можете указать любую величину, чтобы оставить логи, накопившиеся за последние <дней>d

b) Очистка файлов Docker (/var/lib/docker)

Если вы используете Docker и у вас накопилось большое кол-во ненужных образов (например, от отработки CI/CD пайплайнов), необходимо провести очистку Docker Engine, с помощью следующих команд:

- Удаление неиспользуемых образов

```bash
sudo docker image prune
```

- Удаление неиспользуемых контейнеров

```bash
sudo docker container prune
```

- Удаление неиспользуемых сетей

```bash
sudo docker network prune
```

- Удаление неиспользуемых томов

```bash
sudo docker volume prune
```

- Удаление всех неиспользуемых объектов в Docker Engine

```bash
sudo docker system prune -a
```

c) Очистка файлов Pterodactyl (/var/lib/pterodactyl)

Для владельцев игровых проектов

Файлы панели Pterodactyl крайне не рекомендуется удалять вручную, поскольку бэкенд панели "не заметит" удаление бэкапа, и оставит его висеть в панели, хотя он уже был удален.

Сделали очистку, но места все равно не хватает?

Удалите неиспользуемые пакеты (зависимости) с помощью вашего пакетного менеджера:

```bash
sudo apt autoremove
```