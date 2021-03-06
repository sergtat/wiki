# Повышаем отказоустойчивость системы на nodejs.
Это не исчерпывающее руководство к действию, просто я делюсь своим опытом, если вы профи в nodejs можете дописать в коментах свои рекомендации, на которые я с удовольствием сошлюсь в статье.

## 1. Nodejs — однопоточный

Это будет немного непривычно, потому что если у вас 4 ядра, то запущенная нода будет нагружать только одно. Т.е. чтоб загрузить все 4 ядра вам надо запустить 4 инстанса (копии) nodejs. Теперь перед этими 4-ма инстанами нужно установить балансировщик, который будет распределять нагрузку. Лучше не просто балансировщик а проксирующий сервер (с возможностью балансировки):

- Это позволит часть ответов вашей ноды положить в кеш прокси-сервера и отдавать его даже без обращения к самой ноде.
- Это позволит раздавать Вам статический контент специально предназначеным для этого софтом а не “самопальной” частью кода на JavaScript.
- Это даст возможность отдать клиенту какой-то ответ, когда с нодой что-то пошло не так.
- Вам не придется существенно менять что-либо в системе когда у вас появится еще одна нода этого же сервиса.

## 2. Что ставить перед нодой?

Тут, есть множество вариантов, например:

- Запустить еще один инстанс ноды и задействовать модуль [cluster](https://nodejs.org/api/cluster.html)
- Использовать проксисервер-балансировщик:
    - [HAProxy](http://www.haproxy.org/)
    - [Varnish](https://www.varnish-cache.org/)
    - [nginx](http://nginx.org/)

Не ставьте apache, он тоже все умеет, но не очень оптимален с точки зрения задействованных ресурсов.

Мы выбрали nginx.

## 3. Инстанс под нагрузкой, который работает продолжительное время начинает «тормозить»

Это связано не только с особенностью ядра, которое понижает приоритет процессу, работающему слишком долго, это и часть проблем со сборкой мусора в самой ноде, это и часть проблем допущенных в Javascript-коде. И мы не нашли ничего лучшего чем с определенной периодичностью ребутить инстансы. Периодичность перезагрузки (в зависимости от разных причин) от 1 раза в час до 1 раза в сутки.

Чтоб не было приостановки работы сервиса, нужно запустить 2 инстанса (копии) сервиса. Во время ребута инстанса в балансировщике нужно снимать с него нагрузку.

Мы это делаем путем внесения правок в конфиг и релоада nginx до и после перезагрузки инстанса.

## 4. Нода под нагрузкой требует увеличения ограничения nofile limit (LimitNOFILE)

Во многих дистрибутивах, по умолчанию там стоят очень скромные цифры. Я рекомендую ставить больше 16000 (у мнея стоит 131070). Это можно задать командой `ulimit -n 131070`, или подправить `/etc/security/limits.conf`

Мы описываем сервер в стандарте `systemd`, там ограничение задается переменной `LimitNOFILE` и выглядит это приблизительно так (файл `/usr/lib/systemd/system/nodejs1936.service`):

```bash
[Unit]
Description=Nodejs instance 1936
After=syslog.target network.target

[Service]
PIDFile=/var/run/nodejs1936.pid
Environment=NODE_ENV=production
Environment=NODE_APP_INSTANCE=1936
WorkingDirectory=/var/www/myNodejsApp
# node_program supervisor
ExecStart=/usr/bin/supervisor --harmony -- /var/www/myNodejsApp/index.js -p 1936
User=node
Group=node
LimitNOFILE=131070
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

## 5. У инстанса ноды память ограничена и это ограничение не очень большое

У nodejs по умолчанию установлен лимит на максимальный размер памяти, которую может «отъедать» каждый инстанс. [Цитирую Faq по v8](https://github.com/nodejs/node-v0.x-archive/wiki/FAQ#what-is-the-memory-limit-on-a-node-process):

"Currently, by default v8 has a memory limit of 512MB on 32-bit systems, and 1.4GB on 64-bit systems."

Но есть способ это поменять с помощью ключа `--max-old-space-size`, память указываем в M, например чтоб увеличить до 4G пишем `--max-old-space-size=4096`

Также можно влиять на размер стека ключем `--stack-size`, например `--stack-size=512`

Увеличение памяти может быть полезно при написании периодически запускающегося процесса, который спроектирован так, чтоб максимально использовать RAM для работы с данными. Ну например, скрипт рассылки писем, анализатор логов и т.д.

## 6. Код создан в виде большого связанного моноблока

Не далайте “три в одном” или “десять в одном” — это может работать, но любая нештатная ситуация завалит весь проект. Наоборот — делите все на 3, 5, 10 независимых сервисов. Чем проще и легче сервис тем стабильнее его работа. “Вылетание” одного сервиса приведет к вылетанию части функционала но не всего проекта.

Применяйте декомпозицию, делите сложные сервисы на несколько простых. Взаимодействуйте между сервисами по REST-протоколу. Это и будет технологичным фундаментом для роста вашего проекта. “Распухший” сервис всегда можно легко переселить на более производительный сервер без изменения архитектуры приложения.

Тут есть и обратная сторона медали, упрощая сами сервисы мы усложняем связи между ними и это тоже может быть проблемой, когда приложение насчитывает десятки а то и сотни сервисов, между ними могут быть сотни а то и тысячи связей. Это повышает отказоустойчивость системы но также повышает сложность разработки, деплоя проекта а также порог вхождения разработчика в проект.

## 7. Используйте софт, который может перезапустить упавший процесс ноды

Как бы грамотно не был написан код все равно придет тот момент. когда он упадет, после непредусмотренной вами ошибки. Для того чтоб решить эту стандартную проблему написано множество cli утилит, которые следят за процессом ноды и в случае необходимости его перегружают:

- [forever](https://github.com/foreverjs/forever)
- [supervisor](https://github.com/petruisfan/node-supervisor)
- [pm2](https://github.com/Unitech/pm2)
- [nodemon](https://github.com/remy/nodemon)

Мы используем `supervisor`.

Эту же функциональность можно организовать с помощью настроек сервиса в `Systemd` ([недавно была статья](http://habrahabr.ru/post/268583/)).

## 8. Самостоятельно собираем свежие релизы ноды

Не ждем когда нода обновиться в вашем дистрибутиве. Это будет крайне неоперативно. Я рекомендую научиться собирать пакеты для своего дистрибутива Linux самостоятельно, тем более что в этом нету ничего сложного.

Не забудьте пересобрать `node_modules`, при смене версии ноды (вернее мажорной или минорной версии), проблемы могут возникать с модулями, часть кода которого написана на с.

## 9. Не бойтесь использовать ECMAScript 2015 (ES6)

Сейчас у нас в на production серверах установлена `nodejs 5.0.0`, а еще год назад стояла `0.11.6`.

Еще в `4.2.2` для того чтоб дописать в конец массива `arr1`, элементы массива `arr2`, нужно было писать вот так

```javascript
var arr1 = [0, 1, 2];
var arr2 = [3, 4, 5];
// Append all items from arr2 onto arr1
Array.prototype.push.apply(arr1, arr2);
```

В `5.0.0` мы это можно сделать так

```javascript
var arr1 = [0, 1, 2];
var arr2 = [3, 4, 5];
arr1.push(...arr2);
```

Напрашивается вопрос: «Как это влияет на стабильность?». Будущее уже пришло, генераторы, классы, промисы... это то что делает код более понятным и прогнозируемым, чем понятнее код тем легче заметить в нем ненормальность и ее устранить. Использование ECMAScript 2015 (ES6) помогло нам полностью избавиться от CallBack hell-а и существенно сократить количество кода.

## 10. Тестируем (ну хоть часть кода)

Не спешите в меня бросать всем что сейчас у вас под руками. Да, я тоже не фанат тестирования на начальном этапе разработки, но ведь мы уже находимся на стадии стабилизации продукта раз читаем такие статьи, не так ли? :)

Я не тестирую все подряд. Обычно я пишу простые тесты, которые дают мне уверенность что у меня все Ok с “внешними” сервисами, и критически важными функциями самого сервиса. Например: записал, отредактировал и удалил ключ в memcached или redis, аналогично с mysql, mongodb, каким-нибудь внешним сервисом.

Часто возникает соблазн сказать: “так я ж мониторю MySQL забиксом, зачем мне проверять что-то с самого сервиса”. Не раз в моей практике была ситуация когда с сервисом вроде как все Ok, с MySQL все Ok, а вот со связью между сервисом и MySQL есть проблема, например, админ случайно добавил не совсем корректное правило в iptables, шнурок между сервером и свичем перегнули или зацепили и он плохо работает и возникают ошибки при передаче по сети, слишком «умное» ядро на базе MySQL решило что сервис производит SYN-flood и начало дропать пакеты и т.п.

Тестировать, по большому счету, можно и без фреймворка, просто вашему забиксу вы возвращаете какое-то значение (0 или 1), или какое-то число и он, при получении значения за пределами допустимого, шлет вам SMS о проблеме.

Если потребность в фреймворке есть, тогда присмотритесь к этим:

- [Jasmine](http://jasmine.github.io/2.0/introduction.html)
- [Mocha](https://mochajs.org/)

Это не все о чем хотел сказать, но уже достаточно для статьи. В следующих сериях мы рассмотрим тюнинг ядра для нужд ноды, грабли виртуализации, как правильно закэшировать nginx-ом ответы от инстансов nodejs.

Источник: Олег Черний @apelsyn https://habrahabr.ru

## Комментарии:
см. https://habrahabr.ru/post/270391/#comments-list
