#  Печать на CUPS-принтер из-под Windows
Научить Windows печатать на принтер, раздаваемый CUPS, просто! Адрес сетевого принтера совпадает с URL настройки этого принтера в CUPS, а драйвер надо указать «Generic» → «MS Publisher Imagesetter»

Настраиваем принтер Cups-PDF на машине grail (имя домена опускаю).

- Заходим на машине с принтером на http:_localhost:631/admin (или удалённо на http:_grail:631/admin, если разрешено), включаем галочку «Разрешить совместный доступ к принтерам, подключенным к этой системе». Сохраняем.
- Заходим в раздел «Принтеры», тыкаем в нужный принтер. Получаем примерно такой URL: http://grail:631/printers/Cups-PDF . Запоминаем его
- В Windows вызвать «Мастер установки принтеров» («Принтеры и факсы» → «Установка принтера»). «Далее» → «Сетевой принтер…» → «подключиться к принтеру в Интернете…». Сюда вбиваем URL (в примере http://grail:631/printers/Cups-PDF); стоит обратить внимание на что, что адрес принтера — не localhost :). «Далее» → изготовитель «Generic» → принтер «MS Publisher Imagesetter» → «Ok» → «Готово».

Проверить можно, ткнув правой кнопкой в образовавшийся принтер → «Свойства» → «Пробная печать». Кстати, в нашем случае результат (файл .pdf) попадает довольно глубоко: в каталог /var/spool/cups-pdf/ANONYMOUS.

На всякий случай проверить ещё 631 порт в брандмауэре на обеих машинах и настройки прокси Windows.