# SELinux
Изучение SELinux на примере развертывания nginx на не страндартном порту. А также обеспечение работоспособности приложения при включенном SELinux.

# Задание
1. Запустить nginx на нестандартном порту 3-мя разными способами: 
   - переключатели setsebool; 
   - добавление нестандартного порта в имеющийся тип; 
   - формирование и установка модуля SELinux.
## Как это работает
### Развернуть стенд запустив команду. Исполюзуется Ansible role и шаблоны J2.
    vagrant up && ansible-playbook setselinux.yml && ansible-playbook play.yaml
Будет развернут стенд с системой CentOS 7.9 \
Проведена проверка статуса SELinux, если статус "Disabled" переводим его в статус Enforsing с помощью шаблона и применяемого параметра. \
Устанавливаем утилиты для работы с SELinux \
Устанавливаем NGINX конфигурируем его при попощи шаблонов указываем не стандартный порт для работы виртуального домена "порт 4881"
- получаем ошибку при поднятии сервиса на нестандартном порту. \
![](https://github.com/vedoff/selinux/blob/main/pict/Screenshot%20from%202022-01-05%2018-26-20.png)

### Исправляем
####  ==================================== Вариант №1 =======================================
#### Переключатели setsebool
  Убедимся, что конфигурация nginx верная и firewall отключен. Проблема в SELinux \
  ![Удастоверимся, что проблема в SELinux](https://github.com/vedoff/selinux/blob/main/pict/Screenshot%20from%202022-01-05%2018-55-00.png)

Разрешим в SELinux работу nginx на порту TCP 4881 c помощью
переключателей setsebool \
Найдем ошибку в логе \
`vi /var/log/audit/audit.log` \
Используем шаблон vi в командном режиме для поиска нужной строки 
- `/4881`
###
![](https://github.com/vedoff/selinux/blob/main/pict/Screenshot%20from%202022-01-05%2018-59-14.png)

Используем время для поиска нужной строки и передаем ее на вход утилиты audit2why

`grep 1641399794.463:518 /var/log/audit/audit.log | audit2why`
###
![](https://github.com/vedoff/selinux/blob/main/pict/Screenshot%20from%202022-01-05%2019-01-47.png)

На выходе получаем требуемые действия для того, что бы наш сервис запустился на не стандартном порту. \
А именно: \
`setsebool -P nis_enabled on` \
Запустим наш сервис nginx \
`setsebool -P nis_enabled on && systemctl restart nginx`
### Проверяем, что сервис запустился
`systemctl status nginx && ss -tunlp && sestatus`
###
![](https://github.com/vedoff/selinux/blob/main/pict/Screenshot%20from%202022-01-05%2019-58-05.png)

Проверка статуса nis_enabled: \
`getsebool -a | grep nis_enabled`
> nis_enabled --> off | on 

Отключение nis_enabled \
 `setsebool -P nis_enabled off`
 
#### =================================== Вариант №2 =======================================

#### Добавление нестандартного порта в имеющийся тип
- Поиск имеющегося типа, для http трафика: \
`semanage port -l | grep http `

![Проверка типа для http трафика](https://github.com/vedoff/selinux/blob/main/pict/Screenshot%20from%202022-01-05%2020-22-20.png)

Добавим порт в тип http_port_t: \
`semanage port -a -t http_port_t -p tcp 4881` \
Проверяем: \
`semanage port -l | grep http `

![Проверка, что порт добавлен](https://github.com/vedoff/selinux/blob/main/pict/Screenshot%20from%202022-01-05%2020-32-46.png)

Перезапускаем nginx и проверяем работу сервиса \
`systemctl restart nginx && systemctl status nginx && ss -tunlp && sestatus`

![Проверка запуска сервиса nginx](https://github.com/vedoff/selinux/blob/main/pict/Screenshot%20from%202022-01-05%2020-35-13.png)

Удалить нестандартный порт из имеющегося типа \
`semanage port -d -t http_port_t -p tcp 4881` \
Проверка что порт удален \
`semanage port -l | grep http_port_t` 

Проверка сервиса nginx \
`systemctl restart nginx` \
`systemctl status nginx`

![Провелка сервиса nginx](https://github.com/vedoff/selinux/blob/main/pict/Screenshot%20from%202022-01-05%2020-54-55.png)

####  ==================================== Вариант №3 =======================================
#### Формирование и установка модуля SELinux

Проверка запуска nginx \
`systemctl status nginx`

Просмотр лога \
`grep nginx /var/log/audit/audit.log`

Создание модуля запуска nginx используя утилиту audit2allow. Так же вывод подсказывает команду установки модуля.\
`grep nginx /var/log/audit/audit.log | audit2allow -M nginx` \
На выходе получим два файла модуля \
nginx.pp \
nginx.te 

Копируем файлы модуля в папку \
`cp nginx.* /etc/nginx/ && cd /etc/nginx`

Запускаем команду установки модуля \
`semodule -i nginx.pp`

![](https://github.com/vedoff/selinux/blob/main/pict/Screenshot%20from%202022-01-05%2021-51-30.png)

Проверяем \
`semodule -l | grep nginx` 

Проверяем запуск nginx \
`systemctl status nginx && systemctl restart nginx && systemctl status nginx`
![](https://github.com/vedoff/selinux/blob/main/pict/Screenshot%20from%202022-01-05%2021-49-19.png)
