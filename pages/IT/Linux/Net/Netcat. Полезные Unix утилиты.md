# Полезные Unix утилиты. Netcat
Утилиту Netcat часто называют эдаким «Швейцарским армейским ножом», в хорошем смысле этого слова. Функционал netcat полезен в той-же степени, в какой полезна многофункциональность и сподручность зарекомендовавшего себя карманного Швейцарского армейского ножа. Некоторые из ее возможностей включают сканирование портов, передачу файлов, прослушивание портов и она может быть использована как бэкдор.

В 2006 году netcat получила 4-ое место в опросе «100 утилит сетевой безопасности», поэтому она — это определенно тот инструмент, который необходимо знать.

## Как пользоваться nc?

Начнем с нескольких простых примеров и далее будем их использовать как базовые.

Если Вы помните, я говорил, что netcat это Швейцарский армейский нож. Чем-бы этот нож был, если бы его нельзя было использовать как обычный нож? Вот почему netcat может использоваться вместо обычного telnet:

    $ nc www.google.com 80

В действительности он более удобный чем обычный telnet, потому что Вы можете завершить соединение в любое время, нажав Ctrl+C и он обрабатывает двоичные данные как обычные (никаких escape последовательностей, ничего).

Вы можете добавить параметр “-v” для более подробного вывода результатов действий, и параметр (-vv) для получения статистики о том, сколько байт было передано во время текущего сеанса соединения.

Netcat может быть использован в качестве сервера. Если Вы запустите его, как указано ниже, он будет слушать на порту 12345 (на всех интерфейсах):
```bash
$ nc -l -p 12345
```
Теперь если Вы подключитесь к порту 12345 этого хоста, все, что Вы набираете будет передано удаленной стороне, что говорит нам о том, что netcat можно использовать как чат сервер. Запустите на одном из компьютеров:

    # На компьютере A с IP 10.10.10.10
    $ nc -l -p 12345

И подключитесь к нему с другого:

    # На компьютере B
    $ nc 10.10.10.10 12345

Теперь обе стороны могут переговариваться!

Такой способ разговора, когда обе стороны могут разговаривать друг с другом делает возможным использование nc для операций ввода/вывода через сеть! К примеру, Вы можете послать целую директорию с одного компьютера на другой организовав tar конвейер через nc на первом компьютере, и перенаправив вывод в другой tar процесс на втором.

Предположим, Вы хотите переслать файлы из директории /data компьютера A с IP 192.168.1.10 на компьютер B (с любым IP). Это просто:

    # На компьютере A с IP 192.168.1.10
    $ tar -cf — /data | nc -l -p 6666
    # На компьютере B
    $ nc 192.168.1.10 6666 | tar -xf —

Не забудьте скомбинировать конвейер с Рipe Viewer, который был описан в предыдущей статье, что-бы посмотреть статистику того, как быстро происходит передача!

Одиночный файл может быть послан проще:

    # На компьютере A с IP 192.168.1.10
    $ cat file | nc -l -p 6666
    # На компьютере B
    $ nc 192.168.1.10 6666 > file

Вы даже можете скопировать и восстановить целый диск, с помощью nc:

    # На компьютере A с IP 192.168.1.10
    $ cat /dev/hdb | nc -l -p 6666
    # На компьютере B
    $ nc 192.168.1.10 6666 > /dev/hdb

Заметим: Опция “-l” не может быть использована совместно с “-p” на Mac компьютерах! Решение, — просто заменить “-l -p 6666? на “-l 6666?. Как здесь:

    # теперь nc слушает на порту 6666 для Mac компьютеров
    $ nc -l 6666

Незаурядное использование netcat — сканирование портов. Netcat не лучший инструмент для такой работы, но он с этим справляется (лучший, конечно-же nmap):
```bash
$ nc -v -n -z -w 1 192.168.1.2 1-1000
(UNKNOWN) [192.168.1.2] 445 (microsoft-ds) open
(UNKNOWN) [192.168.1.2] 139 (netbios-ssn) open
(UNKNOWN) [192.168.1.2] 111 (sunrpc) open
(UNKNOWN) [192.168.1.2] 80 (www) open
(UNKNOWN) [192.168.1.2] 25 (smtp) : Connection timed out
(UNKNOWN) [192.168.1.2] 22 (ssh) open
```
Параметр “-n” предотвращает от просмотра DNS, “-z” не ждет ответа от сервера, и “-w 1? задает таймаут для соединения в 1 секунду.

Другое нетривиальное использование netcat в роли прокси. И порт и хост могут быть перенаправлены. Посмотрите на этот пример:

    $ nc -l -p 12345 | nc www.google.com 80

Эта команда запускает nc на порту 1234 и перенаправляет все соединения на google.com:80. Если теперь Вы подключитесь к этому компьютеру на порту 12345 и сделаете запрос, Вы обнаружите, что в ответ не получаете никаких данных. Это правильно, потому-что мы не установили двунаправленный канал. Если Вы добавите второй канал, Вы получите Ваши данные на другом порту:

    $ nc -l -p 12345 | nc www.google.com 80 | nc -l -p 12346

После посылки запроса на порт 12345, получите Ваши данные ответа на порту 12346.

Вероятно самая мощная возможность netcat — запустить любой процесс как сервер:

    $ nc -l -p 12345 -e /bin/bash

Параметр “-e” пораждает выполнение ввода и вывода перенаправляемого через сетевой сокет. Теперь, если Вы подключитесь к хосту на порту 12345, Вы можете использовать bash:
```bash
$ nc localhost 12345
ls -las
total 4288
4 drwxr-xr-x 15 pkrumins users 4096 2009-02-17 07:47 .
4 drwxr-xr-x 4 pkrumins users 4096 2009-01-18 21:22 ..
8 -rw——- 1 pkrumins users 8192 2009-02-16 19:30 .bash_history
4 -rw-r—r— 1 pkrumins users 220 2009-01-18 21:04 .bash_logout
…
```
Последствия таковы, что nc это популярный инструмент хакера и с его помощью можно очень легко сделать бэкдор. На Linux сервере вы можете запустить /bin/bash а на Windows cmd.exe и иметь в своих руках полный контроль.

Это все, что я хотел сказать. Вам знакомы какие-либо другие полезные приемы работы с netcat, которые здесь не описаны?

Если у Вас Windows, скачайте порт Windoze с [securityfocus](http://www.securityfocus.com/tools/139).

Руководство по утилите может быть найдено в man nc.
