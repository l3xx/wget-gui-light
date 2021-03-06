Wget GUI Light
=========

![license MIT](http://gitshields.com/v2/text/license/MIT/green.png)&nbsp;![license MIT](http://gitshields.com/v2/text/os/linux/orange.png)&nbsp;![php 4,5](http://gitshields.com/v2/text/php/4,5/blue.png)&nbsp;![version](http://gitshields.com/v2/text/version/0.1.5/lightgrey.png)

#### Что это?

Это **web-интерфейс для [wget]** - программы загрузки файлов по сети. Построен по принципу клиент/сервер, где клиент построен с использованием [Ajax] и [CSS3], и [PHP] с серверной.

#### Какие требования к системе?

&nbsp; | Описание
---: | :-------------
unix | Написано и протестировано именно для ОС данного семейства. Для запуска на других ОС потребуются рабочие порты `wget`, `ps` и `rm`. Протестирована работа на версиях _Debian Linux 2.6.32.11_ и _FreeBSD 8.3_
bash | Используется создание [подпроцессов] и проверка условий, написано именно под эту командную оболочку. Для запуска под другой потребуется изменить все запуски системных команд. Протестирована работа на версиях _4.1.11(2)_ и _4.2.24(1)_
php  | Необходим выключенный [safe_mode], разрешение на выполнение [exec()], [posix_kill()], [unlink()] и ряда других функций. Протестирована работа на версиях _5.2.6_ и _5.3.13_

#### Особенности и настройки серверной части

Серверная часть выполняет следующие задачи:

 - Получение информации о запущенных задачах (через запуск `ps` и парсинг лог-файлов `wget`);
 - Отмена запущенных задач (через выполнение [posix_kill()]);
 - Добавление новых задач (производя запуск `wget` в подпроцессе через [exec()]);
 - Получение списка истории (парсинг файла истории);
 - Ведение логов;
 - Тестирование серверной части;
 - Возвращение результатов в формате [JSON].

Основные настройки вынесены в файл `settings.php`, и устанавливаются в формате:

> `define('параметр', 'значение');`

Описание параметров приведено в таблице ниже:

Параметр       | Описание
------------: | :--------------
`download_path` | Путь до директории, в которую будет происходить сохранение всех загружаемых файлов.<br />**Обязательный** параметр.<br />По умолчанию `BASEPATH.'/downloads'`
`tmp_path` | Путь для временных лог-файлов `wget`. Автоматически удаляются при завершении или прерывании задачи.<br />**Обязательный** параметр.<br />По умолчанию `'/tmp'`
`log_path` | Директория для лог-файлов серверной части. Возможен режим записи разных типов сообщений в разные файлы (устанавливаются в константах класса `log`). По умолчанию запись всех сообщений происходит в файл `wgetgui.log`.<br />По умолчанию `'/log'`
`history` | Вести запись истории задач. Так же отображается в GUI.<br />По умолчанию `/log/history.log`
`WgetOnetimeLimit` | Лимит на количество одновременно запущенных задач.<br />По умолчанию `10`
`wget_download_limit` | Ограничение скорости закачки `wget` в Кб/сек.<br />По умолчанию `2048`
`wget_secret_flag` | Параметр `wget`, дописываемый ко всем запускаемым задачам, для определения источника запуска. Без необходимости не надо его изменять.<br />**Обязательный** параметр.<br />По умолчанию `--max-redirect=4321`
`DebugMode` | Включается режим отладки (добавляет подробный вывод в лог-файл).<br />По умолчанию _выключен_
`wget`, `ps` и `rm` | Возможно указать свои пути до исполняемых фалов `wget`, `ps` и `rm` в случае, если в этом есть необходимость.<br />По умолчанию параметры _выключены_

Скрипт отвечает на POST и GET запросы, запросы из командной строки. Например:

> `php ./rpc.php get_list`<br />`php ./rpc.php add_task http://goo.gl/5Qi0Xs`

Поддерживается работа (автоматическое получение прямых ссылок) со следующими ресурсами:

* **YouTube**, примеры ссылок:
  - `http://www.youtube.com/watch?v=o1k8hJ1d8G4`
  - `http://youtu.be/o1k8hJ1d8G4`
  - `www.youtube.com/embed/o1k8hJ1d8G4`
* **vk.com**, примеры ссылок:
  - `<iframe src="http://vk.com/video_ext.php?oid=25654706&id=167596549&hash=22e03c697e10f723&hd=1" width="607" height="360" frameborder="0"></iframe>`
  - `http://vk.com/video_ext.php?oid=25654706&id=167596549&hash=22e03c697e10f723`


#### Особенности и настройки клиентской части

Для браузера обязательна поддержка [JavaScript] и крайне желательна свойств [CSS3]. Интерфейс выполнен в минималистичном стиле:

![screenshot](http://habrastorage.org/files/b0e/e2b/4f8/b0ee2b4f89344f7894e6e61bbd31ade9.png)

Все запросы выполняются без перезагрузки страницы. Дизайн — [адаптивный]. Изменение состояния отображается как на самой странице, так и в заголовке вкладки (окна). 

Возможно одновременное добавление нескольких задач, добавляя новые строки нажатием клавиш **Ctrl (⌘)** и **Enter** (одна строка - одна задача).

Для того, чтобы задать имя сохраняемого файла необходимо добавить к строке задачи с URL строку вида "` -> filename.ext`" (с пробелом в начале), полный вид запроса при этом будет:

> `htttp://somehost.io/oldfilename.zip -> newfilename.zip`

Если указанное указанный файл имеется - он будет _перезаписан_.

При нажатии клавиши **F5** происходит обновление списка задач, страница перезагружается только по нажатию на кнопку "Обновить страницу" в браузере.

Основной [JavaScript] код расположен в файле `js/core.js`, но его основные настройки (как и настройки `rpc.php`) располагаются в файле `settings.php`:

Параметр       | Описание
------------: | :--------------
`addTasksLimitCount` | Лимит на количество одновременно добавляемых задач.<br />**Обязательный** параметр.<br />По умолчанию `5`
`updateStatusInterval` | Интервал обновления данных в открытой вкладке или окне (будьте аккуратны с этим параметром на слабых серверах).<br />По умолчанию `5000`
`DebugMode` | Режим отладки, при активации которого выводится отладочная информация в console.log().<br />По умолчанию `false`
`CheckForUpdates` | Функция автоматической проверки обновлений. При наличии обновления выводит информационную ссылку рядом с указателем текущей версии в выдвижном меню.<br />По умолчанию `true`
`CheckExtensionInstalled` | Функция проверки установленного расширения для браузера. В случае его отсутствия или деактивированности - выводит ссылку на него в выдвижном меню.<br />По умолчанию `true`
`prc` | Путь до скрипта серверной части.<br />По умолчанию `rpc.php`

#### Установка / обновление

Подробная инструкция находится на странице "[Установка и обновление]"

#### История изменений

Подробная история изменений и нововведений находится на странице "[История изменений]"

#### Расширения для браузеров

Расширения пишутся по мере возможности, в данный момент доступны версии для:

 * [Google Chrome Extension]
 * Firefox (в планах)
 * Opera (в планах)

Если у вас есть желание и возможность - вы можете внести свой вклад разработав его самостоятельно, и прислав его исходный код. Ваше имя будет внесено в список разработчиков. Связаться с разработчиками можете воспользовавшись страницей "[Задать вопрос]"

##### Ссылки на новости о данном проекте:

 * Новость на ресурсе [Хабрахабр]
 * Запись в блоге [blog.kplus.pro]

#### Я нашел ошибку / мне необходима функция

Для обсуждения найденных недоработок, ошибок, или не хватающего функционала воспользуйтесь страницей "[Сообщить об ошибке]".

#### Использованы следующие библиотеки и решения:

 * Библиотека "[JQuery]"
 * Парсер ссылок "[url.js]"
 * JQuery-плагин уведомлений "[jquery.owl]"
 * CSS код из статьи "[CSS3 progress bar]"
 * JQuery-плагин для работы с cookies "[jquery.cookie.js]"
 * JQuery-плагин "выплывающих окон" "[bPopup]"

#### Лицензия: **MIT**

> Copyright © 2014 Samoylov Nikolay
> 
> Данная лицензия разрешает лицам, получившим копию данного программного
> обеспечения и сопутствующей документации (в дальнейшем именуемыми
> «Программное Обеспечение»), безвозмездно использовать Программное
> Обеспечение без ограничений, включая неограниченное право на
> использование, копирование, изменение, добавление, публикацию,
> распространение, сублицензирование и/или продажу копий Программного
> Обеспечения, также как и лицам, которым предоставляется данное
> Программное Обеспечение, при соблюдении следующих условий:
> 
> Указанное выше уведомление об авторском праве и данные условия должны
> быть включены во все копии или значимые части данного Программного
> Обеспечения.
> 
> ДАННОЕ ПРОГРАММНОЕ ОБЕСПЕЧЕНИЕ ПРЕДОСТАВЛЯЕТСЯ «КАК ЕСТЬ», БЕЗ
> КАКИХ-ЛИБО ГАРАНТИЙ, ЯВНО ВЫРАЖЕННЫХ ИЛИ ПОДРАЗУМЕВАЕМЫХ, ВКЛЮЧАЯ, НО
> НЕ ОГРАНИЧИВАЯСЬ ГАРАНТИЯМИ ТОВАРНОЙ ПРИГОДНОСТИ, СООТВЕТСТВИЯ ПО ЕГО
> КОНКРЕТНОМУ НАЗНАЧЕНИЮ И ОТСУТСТВИЯ НАРУШЕНИЙ ПРАВ. НИ В КАКОМ СЛУЧАЕ
> АВТОРЫ ИЛИ ПРАВООБЛАДАТЕЛИ НЕ НЕСУТ ОТВЕТСТВЕННОСТИ ПО ИСКАМ О
> ВОЗМЕЩЕНИИ УЩЕРБА, УБЫТКОВ ИЛИ ДРУГИХ ТРЕБОВАНИЙ ПО ДЕЙСТВУЮЩИМ
> КОНТРАКТАМ, ДЕЛИКТАМ ИЛИ ИНОМУ, ВОЗНИКШИМ ИЗ, ИМЕЮЩИМ ПРИЧИНОЙ ИЛИ
> СВЯЗАННЫМ С ПРОГРАММНЫМ ОБЕСПЕЧЕНИЕМ ИЛИ ИСПОЛЬЗОВАНИЕМ ПРОГРАММНОГО
> ОБЕСПЕЧЕНИЯ ИЛИ ИНЫМИ ДЕЙСТВИЯМИ С ПРОГРАММНЫМ ОБЕСПЕЧЕНИЕМ.

[wget]:http://wikipedia.org/wiki/Wget
[Ajax]:http://wikipedia.org/wiki/AJAX
[CSS3]:http://wikipedia.org/wiki/css3
[PHP]:http://wikipedia.org/wiki/php
[подпроцессов]:http://www.tldp.org/LDP/abs/html/subshells.html
[bash]:http://wikipedia.org/wiki/Bash
[safe_mode]:http://php.net/manual/en/features.safe-mode.php
[exec()]:http://php.net/manual/en/function.exec.php
[posix_kill()]:http://php.net/manual/en/function.posix-kill.php
[unlink()]:http://php.net/manual/en/function.unlink.php
[JavaScript]:http://wikipedia.org/wiki/JavaScript
[адаптивный]:http://wikipedia.org/wiki/Responsive_web_design
[JSON]:https://wikipedia.org/wiki/JSON

[Установка и обновление]:https://github.com/tarampampam/wget-gui-light/blob/master/HowUpdate.md

[История изменений]:https://github.com/tarampampam/wget-gui-light/blob/master/cahngeslog.md

[Задать вопрос]:https://github.com/tarampampam/wget-gui-light/issues/new
[Сообщить об ошибке]:https://github.com/tarampampam/wget-gui-light/issues/new

[Хабрахабр]:http://habrahabr.ru/post/234353/
[blog.kplus.pro]:http://tmblr.co/ZYW79o1PAiXW-

[Google Chrome Extension]:https://chrome.google.com/webstore/detail/wget-gui-light/dbcjcjjjijkgihaddcmppppjohbpcail

[JQuery]:http://jquery.com/
[jquery.owl]:http://codecanyon.net/item/owl-unobtrusive-css3-notifications/408575
[url.js]:https://github.com/websanova/js-url
[CSS3 progress bar]:http://css-tricks.com/css3-progress-bars/
[jquery.cookie.js]:https://github.com/carhartl/jquery-cookie
[bPopup]:http://dinbror.dk/bpopup/
