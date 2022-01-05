# SELinux
Изучение SELinux на примере развертывания nginx на не страндартном порту. А также обеспечение работоспособности приложения при включенном SELinux.

# Задание
1. Запустить nginx на нестандартном порту 3-мя разными способами: 
   - переключатели setsebool; 
   - добавление нестандартного порта в имеющийся тип; 
   - формирование и установка модуля SELinux.
## Как это работает
### Развернуть стенд запустив команду
    vagrant up && ansible-playbook setselinux.yml && ansible-playbook play.yaml
Будет развернут стенд с системой CentOS 7.9 \
Проведена проверка статуса SELinux, если статус "Disabled" переводим его в статус Enforsing с помощью шаблона и применяемого параметра. \
Устанавливаем утилиты для работы с SELinux \
Устанавливаем NGINX конфигурируем его при попощи шаблонов указываем не стандартный порт для работы виртуального домена "порт 4881"
- получаем ошибку при поднятии сервиса на нестандартном порту. \
![](https://github.com/vedoff/selinux/blob/main/pict/Screenshot%20from%202022-01-05%2018-26-20.png)

### Исправляем
- Вариант №1 \
  Убедимся, что конфигурация nginx верная и firewall отключен. Проблема в SELinux \
  ![Удастоверимся, что проблема в SELinux](https://github.com/vedoff/selinux/blob/main/pict/Screenshot%20from%202022-01-05%2018-55-00.png)

Разрешим в SELinux работу nginx на порту TCP 4881 c помощью
переключателей setsebool \
Найдем ошибку в логе \
`vi /var/log/audit/audit.log` \
Используем шаблон vi в командном режиме для поиска нужной строки 
- `/4881`\
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
### Проверяем что сервис запустился
`systemctl status nginx && ss -tunlp`
###
![](https://github.com/vedoff/selinux/blob/main/pict/Screenshot%20from%202022-01-05%2019-03-47.png)

