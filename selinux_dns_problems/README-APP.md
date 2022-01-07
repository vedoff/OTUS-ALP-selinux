# Обеспечение работоспособности приложения при включенном SELinux
Обеспечить работоспособность приложения при включенном selinux.
- развернуть приложенный стенд
https://github.com/mbfx/otus-linux-adm/tree/master/selinux_dns_problems;
- выяснить причину неработоспособности механизма обновления зоны (см. README);
- предложить решение (или решения) для данной проблемы;
- выбрать одно из решений для реализации, предварительно обосновав выбор;
- реализовать выбранное решение и продемонстрировать его работоспособность. 

Выполним клонирование репозитория: \
`git clone https://github.com/mbfx/otus-linux-adm.git` 

Заходим на хост Client \
`vagrant ssh client` 
![](https://github.com/vedoff/selinux/blob/main/pict/Screenshot%20from%202022-01-06%2016-28-13.png)

Попробуем внести изменения в зону

![](https://github.com/vedoff/selinux/blob/main/pict/Screenshot%20from%202022-01-06%2016-29-39.png)

Изменения внести не получилось. Смотрим логи SELinux 

`sudo -i` \
`cat /var/log/audit/audit.log | audit2why` 

На клиенте отсутствуют ошибки 

![](https://github.com/vedoff/selinux/blob/main/pict/Screenshot%20from%202022-01-06%2016-47-24.png)

Подключимся к серверу ns01 и проверим логи SELinux \
Проверяем лог \
`cat /var/log/audit/audit.log | audit2why` \
В логах мы видим, что ошибка в контексте безопасности. Вместо типа
named_t используется тип etc_t.

![](https://github.com/vedoff/selinux/blob/main/pict/Screenshot%20from%202022-01-06%2016-59-20.png)

Проверим данную проблему в каталоге /etc/named
![](https://github.com/vedoff/selinux/blob/main/pict/Screenshot%20from%202022-01-06%2022-10-26.png)

Проблема заключается в том, что конфигурационные файлы лежат в другом каталоге.
Посмотреть в каком каталоги должны лежать, файлы, чтобы на них
распространялись правильные политики SELinux можно с помощью команды:

`semanage fcontext -l | grep named` 

Изменим тип контекста безопасности для каталога /etc/named: \
`chcon -R -t named_zone_t /etc/named` \

![](https://github.com/vedoff/selinux/blob/main/pict/Screenshot%20from%202022-01-07%2012-37-56.png)

Попробуем снова внести изменения с клиента: \
`nsupdate -k /etc/named.zonetransfer.key`
![]()
