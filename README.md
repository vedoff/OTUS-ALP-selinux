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
- Вариант №1

Разрешим в SELinux работу nginx на порту TCP 4881 c помощью
переключателей setsebool \
Найдем ошибку в логе \
vi /var/log/audit/audit.log \
Используем шаблон vi в командном режиме для поиска нужной строки 
- /4881