#  Блокировка нежелательных URL с помощью iptables
Иногда может возникнуть необходимость заблокировать доступ к некоторым сайтам, и внесение их адресов или даже подсетей в фаервол не приносит желаемых результатов, так как продуманные пользователи продолжают посещать их используя веб прокси количество которых так велико что о блокировке их не может быть и речи… Есть конечно и явные минусы этого способа, но в моем случае минусы приниматься во внимание не стали. Из плюсов стоит отметить полную блокировку сайтов, они не будут работать даже через веб и обычные прокси сервера, с них не будет доставлятся почта и тд.

Далее собственно примеры блокировки с небольшим пояснением. Реализовано все на основе patch-o-matic, дополнение к iptables от Netfilter. Используем модуль string для достижения цели.

Первый пример, имеем машинку-шлюз для локальной сети, там создаем правило для блокировки например ресурса vkontakte.ru выглядеть это будет так

`iptables -A FORWARD -m string --string "vkontakte.ru" --algo kmp --to 65535 -j DROP`

Все запросы пользователей использующих данную машину как шлюз потерпят неудачу при попытке открыть сайт напрямую или через прокси, так же не будет работать ресурс и через веб прокси.

Для блокировки адресов на линукс машине немного отредактируйте правило

`iptables -A INPUT -m string --string "vkontakte.ru" --algo kmp --to 65535 -j DROP`

Вот собственно и вся настройка блокировки URL адресов с помощью iptables. Минусы данного способа в том что пользователи не смогут окрыть страницы с упоминанием заблокированных ресурсов, прочитать почту например на майл ру если в ящике лежит письмо с сайта который был заблокирован вышеописанным способом…
