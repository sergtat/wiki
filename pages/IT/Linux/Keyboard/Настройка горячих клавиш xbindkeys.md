#  Настройка горячих клавиш xbindkeys
xbindkeys это незаменимая утила, позволяющая очень гибко настраивать горячие клавиши в линух-системах.  Я приведу пример установки и настройки этой крутой утилы на примере убунты, хотя настройка ничем не отличается от настройки в других дистрах. Итак начнем!

Синтаксис конфига таков:
  #Коментарий к проге(произвольный)
  "команда_для_запуска_проги"
  Mod2+Mod4 + q
  Заключительная строка(3) и есть сама комбинация(в примере это win + q). Что бы ее узнать нужно запустить xbindkeys с ключом -mk

  xbindkeys -mk

Появится окошко и нужно просто нажать требуемую комбинацию клавишь. Таким образом заполнять конфиг. Когда конфиг заполнен нужно запустить прогу
  xbindkeys

Ну а кому влом разбираться и ковыряться в конфиге и в консоли, есть удобный гуй, который все сам сделает, нужно его просто запустить:

  xbindkeys-config