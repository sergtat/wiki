# pv — маленькая, но очень полезная утилита
И так pv — это сокращенно от pipeviewer, то есть ни много не мало как просмотрщик пайпов. Про эффективность использования пайпов расказывать не буду, ни для кого это не секрет. Единственное, «но» в работе с ними — это то, что набрав команду и нажав Enter часто не хватает самой малости — знать сколько времени займет ее выполнение. Именно cкорость обработки данных и будет нам показывать pv.

Дальше вводная, допустим вы такой же как и я счасливый обладатель каких-нибудь полезных логов и в какой-то момент у вас дошли руки их заархивировать, например так
```js
% cat rt94-171-06 | gzip > rt94-171-06.gz
```
Есть мысли, сколько времени займет эта операция?

Тоже самое при помощи pv

    %pv rt94-171-06 | gzip > rt94-171-06.gz
    128MB 0:00:15 [ 9.1MB/s] [=====>.....................] 18% ETA 0:01:07

Наглядно видно, что через пайп за 15 секунд прошло 128Мб — это 18% от всего объема, операция займет еще минуту и 7 секунд.

Может показаться, что pv это такая замена для cat, но на самом деле ее возможности намного шире. Например, упаковываем весь каталог в сжатый архив

    %tar -czf - . | pv > out.tgz
    21.9MB 0:00:15 [1.47MB/s] [...<=>.....................]

Уже неплохо, но хочеться большего, чтобы показывалось время окончания работы. Для этого всего лишь надо при помощи ключа -s передать pv размер каталога в байтах

    %tar -czf - . | pv -s $(du -sb | grep -o '[0-9]*') > out.tgz
    44.3MB 0:00:27 [1.73MB/s] [>..........................] 0% ETA 13:36:22

У меня вся операция займет 13 с половиной часов. Хех, накопил =)

Можно так же составлять команды из несколько копий pv.

    %tar -cf - . | pv -cN tar -s $(du -sb | grep -o '[0-9]*') | gzip | pv -cN gzip > out.tgz
    tar: 97.1MB 0:00:08 [12.3MB/s] [>......................] 0% ETA 1:50:26
    gzip: 13.1MB 0:00:08 [1.6MB/s] [....<=>................]

Ключ -c нужен, чтобы несколько копий pv не выводили информацию друг поверх друга. Ключ -N дает имя шкале.

Ну и под конец забавный пример с одного англоязычного блога о Линуксе

    %pv /dev/urandom > /dev/null
    18MB 0:00:05 [ 3,6MB/s] [...<=>............................]

> Источник Habrahabr.ru

## Из комментариев:

Есть еще такое приспособление для похожих нужд: [www.theiling.de/projects/bar.html]()

Ну как вариант «навскидку», — прописать в .bashrc:
```sh
pvcp() { command /bin/cp $@ | pv ; }
pvmv() { command /bin/mv $@ | pv ; }
```
Ну и результат:
```sh
fish@bone:~$ pvcp /dev/zero /dev/null 
0B 0:00:03 [   0B/s ] [<=>.................................]
fish@bone:~$ 
```
Назвать можете как угодно (pvcp, vcp или даже заменить стандартный cp =)
P.S. Возможны проблемы с аргументами, хотя у меня всё отлично работает =)

