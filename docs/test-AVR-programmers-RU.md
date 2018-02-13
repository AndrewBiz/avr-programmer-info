# Прошивка МК через порт UART (самопрограммирование)

Прошивка чипа посредством программатора, зашитого в сам этот чип (секция загрузчика - Boot Loader Section). В руководствах по чипам AVR данный процесс называется "самопрограммирование" микроконтроллера.


**Плюсы:**

 - Быстрый программатор
 - Все платы ARDUINO с USB разъемом готовы к работе прямо из коробки (классно для новичков)
 - дешевые USB-to-Serial (или USB-to-TTL) переходники (FTDI, CH430, CP2102, PL2303), позволяющие быстро подключить и прошить ARDUINO pro mini (те, что без USB порта)

**Минусы**

 - UART занят. Если изделию для работы нужен этот порт, то нужно извращаться с переключателями (отрубать порт RX от внешнего устройства на время прошивки)
 - внутренний программатор отъедает память flash - почти килобайт становится недоступным
 - новый чип требует предварительной прошивки программатора (например через SPI)
 - нет возможности тонкой настройки чипа - прошивки фьюзов


**Общий принцип прошивки методом самопрограммирования**:
Программа под целевой (target) микроконтроллер (МК) пишется, собирается на хост машине (Windows, Linux, MacOS). Затем готовый hex файл с помощью программы avrdude, запускаемой на хост машине, через COM порт (как правило, виртуальный COM порт поверх реального USB соединения) передается на UART вход МК, где "подхватывается" программатором (программка в бутлоадере МК) и уже программатор записывает (прошивает) получаемые от хоста данные во FLASH память МК.

Для реализации описанной схемы нужно устройство - USB-TTL (или USB-Serial) переходник, одним концом включаемый в USB порт хоста, другим - к UART порту целевого МК.

### Обзор USB-Serial переходников
Прошивка целевого МК (ATMEGA168) производилась командой с использованием самопрограмматора arduino:

```shell
avrdude -v -p atmega168 -c arduino -b 19200 -P COM3 -D -U flash:w:firmware.hex:i
```
*Примечание 1* во всех случаях использовались последние драйвера с сайта производителя чипа переходника

*Примечание 2* тесты проводились с последней доступной версией arvdude (v6.3)

| Устройство | Изображение | Комментарий  |
|------------|------|--------------|
|[FT232RL-USB-Serial][FT232RL-USB-Serial-link]| ![FT232RL-picture][FT232RL-USB-Serial]|**Ok** <br> Замечательно прошивает, не требует никаких особых телодвиждений, работает в режиме моста UART-Host|
|[FT232RL-red-USB-Serial][FT232RL-red-USB-Serial-link]| ![FT232RL-picture][FT232RL-red-USB-Serial]|**Nok - прошивка НЕ РАБОТАЕТ** (sync error). <br> **Ок** - работа в режиме моста UART-Host (взаимодействие с МК через программу-терминал на хосте)|
|[CH340G-USB-TTL][CH340G-CP2102-PL2303-USB-TTL-link] | ![CH340G-picture][CH340G-USB-TTL]|**Ok** Работает и прошивка и мост UART-Host<br> Требуется жать и отпускать RESET на целевом чипе, чтобы прошивка стартовала|
|[CP2102-USB-TTL][CH340G-CP2102-PL2303-USB-TTL-link]| ![CP2102-picture][CP2102-USB-TTL]|**Ok** Работает и прошивка и мост UART-Host<br> Требуется жать и отпускать RESET на целевом чипе, чтобы прошивка стартовала|
|[PL2303XA-USB-TTL][CH340G-CP2102-PL2303-USB-TTL-link]|![PL2303XA-picture][PL2303XA-USB-TTL]|**Nok - прошивка НЕ РАБОТАЕТ**. Процесс прошивки повисает на 87%.<br> Требуется не забывать нажимать RESET на целевом МК в момент запуска прошивки <br>  **Ок** - работа в режиме моста UART-Host (взаимодействие с МК через программу-терминал на хосте) |
|[Arduino Original (m16u2)][Arduino-orig-link]|![Arduino official][Arduino-orig]|**Ok** Замечательно прошивает, не требует никаких особых телодвиждений, работает в режиме моста UART-Host. <br> *В качестве USB-UART переходника используется полноценный микроконтроллер ATMEGA16u2. Т.е. на родной плате Arduino Uno имеем целых два МК от Атмел*|
|[Arduino China (CH340)][Arduino-china-link]  |![Arduino China][Arduino-china]|**Ok** Замечательно прошивает, не требует никаких особых телодвиждений, работает в режиме моста UART-Host. <br> *В качестве USB-UART переходника используется китайский чип CH340G. Работает вполне надежно, при этом на порядок дешевле варианта m16u2*|


[FT232RL-USB-Serial-link]: https://ru.aliexpress.com/item/FT232RL-USB-To-Serial-Adapter-Module-Pro-Mini-Atmega168-5V-16M-for-Arduino/1960622904.html?spm=a2g0v.10010108.1000016/B.1.66f71406HJ5koX&isOrigTitle=true
[FT232RL-red-USB-Serial-link]: https://ru.aliexpress.com/item/20pcs-lot-FT232RL-FT232-USB-TO-TTL-5V-3-3V-Download-Cable-To-Serial-Adapter-Module/1913268497.html?spm=a2g0s.9042311.0.0.eQKB6C
[CH340G-CP2102-PL2303-USB-TTL-link]: https://ru.aliexpress.com/item/Free-shipping-3pcs-lot-1PCS-PL2303-1PCS-CP2102-1PCS-CH340-USB-TO-TTL/32649042986.html?spm=a2g0s.9042311.0.0.eQKB6C
[Arduino-orig-link]: https://store.arduino.cc/usa/arduino-uno-rev3
[Arduino-china-link]: https://www.aliexpress.com/item/Free-shipping-UNO-R3-MEGA328P-for-Arduino-Compatible-with-USB-cable-and-9V-battery-clip-snap/1960629582.html?spm=2114.12010108.1000013.3.1c744149ip0JhR&traffic_analysisId=recommend_2088_2_-1_iswistore&scm=1007.13339.90158.0&pvid=a426d722-30f2-45f7-a15e-995602897287&tpp=1

[FT232RL-USB-Serial]: /docs/pics/FT232RL-USB-Serial-adapter.jpg
[FT232RL-red-USB-Serial]: /docs/pics/FT232RL-red-USB-Serial-adapter.jpg
[CH340G-USB-TTL]: /docs/pics/CH340G-USB-TTL-adapter.jpg
[CP2102-USB-TTL]: /docs/pics/CP2102-USB-TTL-adapter.jpg
[PL2303XA-USB-TTL]: /docs/pics/PL2303-USB-TTL-adapter.jpg
[Arduino-orig]: /docs/pics/Arduino-Uno-official.jpg
[Arduino-china]: /docs/pics/Arduino-Uno-CH340G.jpg


-----------


# Прошивка МК через порт SPI
Прошивка специальным внешним (или гибридным) программатором, подключаясь к пинам SPI порта целевого чипа.

**Плюсы**

 - Можно прошивать прямо в устройстве (In-System-Programming)
 - Можно прочитать\изменить фьюзы
 - Можно прошить "нулевый" (только с завода) чип
 - Можно восстановить "убитый" ARDUINO чип (если программатор внутри слетел)
 - для программирования чипа доступна вся его память (не тратим байтики на программатор внутри чипа)

**Минусы**

 - Нужно делать на плате устройства 6 пиновый разъем
 - нужен отдельный (внешний) программатор

----------------------------------------------------
### (SPI) Программатор USBasp

#### Принцип действия
Программа под целевой (target) микроконтроллер (МК) пишется, собирается на хост машине (Windows, Linux, MacOS). Затем готовый hex файл с помощью программы avrdude, запускаемой на хост машине, по интерфейсу USB передается на устройство USBasp, где обрабатывается микроконтроллером устройства (программа - программатор, прошитая в МК) и по интерфейсу SPI прошивается на целевой МК.

#### Изображение

[usbasp-face]: /docs/pics/USBASP-AVR-programmer.jpg "USBasp front-side"
![USBasp front-side][usbasp-face]

#### Характеристики

|Характерисика | Описание     |
|--------------:|:--------------|
| Мозги устройства    | МК ATMEGA8 или подобный
| Вход (порт на комп) | USB порт
| Windows driver| USB для Win7: libusb-win32 v1.2.6
| Выход (порт на целевой чип) | Стандарт ICSP 10 пин
| Производитель | http://www.fischl.de/usbasp/
| Версия прошивки | самая свежая на момент тестирования: usbasp.2011-05-28 <br> *Требуется прошивка МК, т.к. китайты поставляют старую прошивку, которая может выдавать ошибки\предупреждения!*<br>*Рекомендуется запрограммировать (прошить 0) FUSE-бит CKOPT - это увеличивает максимальную частоту МК до 16 МГц*
| Где купить | [пример на ALIEXPRESS за 1.5 USD]( https://ru.aliexpress.com/item/1pcs-New-USBASP-USBISP-AVR-Programmer-USB-ISP-USB-ASP-ATMEGA8-ATMEGA128-Support-Win7-64K/32653187143.html?spm=a2g0s.9042311.0.0.NNvfPE)
| Обзоры| [Неплохой обзор на Микроконтроллер.ру](http://microkontroller.ru/programmirovanie-mikrokontrollerov-avr/usbasp-usb-avr-programmator/)

#### (USBasp) Тесты AVRDUDE (командная строка)
Программа входит в стандартный AVR toolchain. Сайт разработчика: http://savannah.nongnu.org/projects/avrdude/

**Важно** не нужно использовать параметр `-D` (отмена очистки памяти перед прошивкой). Т.е. FLASH память должна обнуляться перед прошивкой. В противном случае часто вылезают ошибки верификации и прошитая программа не работает.

Ниже представлены результаты тестов путем прошивки контроллера ATMEGA328P (плата ARDUINO) командой:

```shell
avrdude -v -p atmega328p -c usbasp  -U flash:w:firmware.hex:i
```

| Test case | Результат | Комментарий  |
|-----------|:------:|--------------|
| avrdude 6.3 - [стандартная](http://savannah.nongnu.org/forum/forum.php?forum_id=8461)| **Ok** | Прошивка чипа происходит без ошибок с правильным USB драйвером|
| avrdude 6.0.1 (найдена [на форуме easyelectronics](http://forum.easyelectronics.ru/))|Nok| error: programm enable: target doesn't answer |
| avrdude 5.10 [DI HALT edition](http://easyelectronics.ru/files/soft/avrdude.zip) |Nok| error: no usb support. please compile again with libusb installed|
| avrdude 5.11patch (в составе SinaProg 2.1)| **Ok**| Прошивка чипа происходит без ошибок с правильным USB драйвером|
| avrdude 6.1svn (в составе AVRDUDE_PROG 3.3) |**Ok** | Прошивка чипа происходит без ошибок с правильным USB драйвером|

#### (USBasp) Тесты Windows GUI программ-прошивальщиков

| GUI программа | Тип | Сайт | Комментарий  |
|---------------|-----|------|--------------|
| platformIO core v3.5.2a | Оболочка над avrdude, <br> IDE на базе Atom или VSCode  | http://platformio.org/ | Ок. Лучшая среда разработки и прошивки МК (михо) <br> **Важно!!** Для прошивки целевого чипа нужно: <br> 1) В параметрах **platformio.ini** указать программатор: <br>`upload_protocol = usbasp` <br> `upload_flags = -Pusb` <br> 2) Для прошивки чипа использовать команду `program` (ctrl-shift-alt-p), а не `upload` (ctrl-alt-u)
|ADS v 11.11.2011 | Оболочка над avrdude | [скачать](http://easyelectronics.ru/files/soft/ADS.ZIP) | Ок. Гибкая программа, все параметры вызова avrdude настраиваются, информация о фьюзах подтягивается онлайн. <br> Можно генерировать Cmd файлы для последующего автономного использования. <br> Консультант - сам [DI HALT](http://easyelectronics.ru/author/di-halt)
|SinaProg 2.1| Оболочка над avrdude | официальный сайт недоступен | Nok. Слабовата на фоне других программ. Бывают непонятные ошибки запуска avrdude |
| Arduino IDE 1.8.5| IDE и оболочка над avrdude| https://www.arduino.cc/ | Nok. Хреновая IDE, глючная оболочка. Тестируемый программатор не заработал ни для обычной прошивки чипа, ни для прошивки бутлоадера|
|AVRDUDE_PROG 3.3|Оболочка над avrdude|[Сайт разработчика](http://www.yourdevice.net/proekty/avrdude-prog)|Ок. Гибкая, вполне удобная|

--------------------------------------------------------------------
### (SPI) Программатор на базе модуля FT2232C (DI HALT)

#### Принцип действия
Программа под целевой (target) МК пишется, собирается на хост машине (Windows, Linux, MacOS). Затем готовый hex файл с помощью программы avrdude, запускаемой на хост машине, по интерфейсу USB прошивается в целевой МК, подключенный к модулю по SPI. Используется специальный режим микросхемы FTDI "bitbang" (8 разрядный порт чипа FT2232 + специально заточенный драйвер FTDI позволяют программно управлять периферией с хоста, в частности, писать команды и данные в МК через SPI). Роль программатора (т.е. алгоритма, который берет данные из hex файла и пишет их во flash память целевого МК) здесь выполняет сама avrdude.

#### Изображение

[ft2232-face]: /docs/pics/FT2232C_DIHALT_programmer.jpg "USBasp front-side"
![FT2232 front-side][ft2232-face]

#### Характеристики

|Характерисика | Описание     |
|--------------:|:--------------|
| Мозги устройства    | Микросхема FTDI FT2232C
| Вход (порт на комп) | USB порт (в устройствах Windows виден как COM порт)
| Windows driver| FTDI для Win7: v2.12.28
| Выход (порт на целевой чип) | Стандарт ICSP 10 пин (при использовании [модуля расширения](http://shop.easyelectronics.ru/index.php?productID=165))
| Производитель | http://shop.easyelectronics.ru/index.php?productID=163
| Версия прошивки | прошивка микросхеме FTDI не нужна

#### (FT2232C) Тесты AVRDUDE (командная строка)
Программа входит в стандартный AVR toolchain. Сайт разработчика: http://savannah.nongnu.org/projects/avrdude/

**Важно** не нужно использовать параметр `-D` (отмена очистки памяти перед прошивкой). Т.е. FLASH память должна обнуляться перед прошивкой.

Ниже представлены результаты тестов путем прошивки контроллера ATMEGA328P (плата ARDUINO) командой:

```shell
avrdude -v -p atmega328p -c 2ftbb -P ft0 -U flash:w:firmware.hex:i
```
*Примечание*: необходимо в файл avrdude.conf вручную добавить конфигурацию программатора ибо он нестандартен:
 - для avrdude версии < 6 параметры выглядят так:
```ini
#FTDI_Bitbang
programmer
  id    = "2ftbb";
  desc  = "FT232R Synchronous BitBang DI-HALT";
  type  = ft245r;
  miso  = 5;  # DCD
  sck   = 6;  # DSR
  mosi  = 4;  # CTS
  reset = 7;  # RI
;  
```
 - для avrdude версии > 6 параметры выглядят так:
```ini
programmer
  id    = "2ftbb";
  desc  = "FTDI DI-HALT with ICSP adapter";
  type  = "ftdi_syncbb";
  connection_type = usb;
  miso  = 5;
  sck   = 6;
  mosi  = 4;
  reset = 7;
;
```

| Test case | Результат | Комментарий  |
|-----------|:------:|--------------|
| avrdude 6.3 - [стандартная](http://savannah.nongnu.org/forum/forum.php?forum_id=8461)| Nok | error: no pthread support. Please compile again with pthread installed|
| avrdude 6.0.1 (найдена [на форуме easyelectronics](http://forum.easyelectronics.ru/))|Nok| error: no ftdi support. Please compile again with libftdi installed|
| avrdude 5.10 [DI HALT edition](http://easyelectronics.ru/files/soft/avrdude.zip) |**Ok**| Прошивка чипа происходит без ошибок |
| avrdude 5.11patch (в составе SinaProg 2.1)| Nok | Данная сборка avrdude не знает про ftdi bitbang режим - невозможно настроить avrdude.conf |
| avrdude 6.1svn (в составе AVRDUDE_PROG 3.3) |Nok|error: no pthread support. Please compile again with pthread installed|

#### (FT2232C) Тесты Windows GUI программ-прошивальщиков
Описание GUI прошивальщиков приведено [ранее](#usbasp-Тесты-windows-gui-программ-прошивальщиков). Если есть желание использовать определенную GUI оболочку - не забудьте связать ее с рабочей версией avrdude.
