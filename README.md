## OTUS Administrator Linux. Professional ДЗ №8: Systemd

**Задание**

1. Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/sysconfig).
2. Из репозитория epel установить spawn-fcgi и переписать init-скрипт на unit-файл (имя service должно называться так же: spawn-fcgi).
3. Дополнить unit-файл httpd (он же apache) возможностью запустить несколько инстансов сервера с разными конфигурационными файлами.
4. \* Скачать демо-версию Atlassian Jira и переписать основной скрипт запуска на unit-файл.

**Решение**