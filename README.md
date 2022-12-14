### Как сменить имя хоста `hostname`
`hostname` - это имя ситемы, которое записывается в ядро __kernel__.
От этого имени пишутся логи и многео другое. В компьютерных системвъх не принято привязываться к ip адресам. имена - потсоянные логические структуры в моём домене. Поэтому все системы принято привязывать к именам и есть два типа имени:
 - __hostname__ как короткое имя
 - есть имя более длинное, включая "фамилию" __FQDN__ - __Fully Qualified Domain Name__ - включает и имя домена
Имя компьютера сохранено в фале `/etc/hostname`. Чтобы поменять имя системы, надо поменять имя в `/etc/hostname`. Это имя сохраняется при загрузке системы в ядро `/proc/sys/kernel.hostname`.
Есть еще несколько команд, которые показывают ту же самую информацию 
- `uname -n` (`uname -all` или `uname -a` - оказывает общую инфорацию о системе)
- `hostname -f` `hostname --fqdn` - длинное __fqdn__ имя

Очень важно чтобы у кажого сервера было свое собственое имя отличное от других
Если длинного имени нет, то система может не знать в каком домене нахолится текущая машина. 

Домейн - это группа или зона имён, как бы "фамилия" к которой может относиться имя *hostname*. __Domainname__ - часть от системы ресолвинга. Домейн не сидит в кернеле. Ресолв происходит в файле `/etc/hosts`:
```
127.0.0.1 ServerName-1.domain.com ServerName-1 www1
192.168.0.1 <полное имя> <алиасы или короткое имя> <alias #2> 
```

теперь можно обращатсья к серверу `ServerName-1.domain.com` через его алиасы: `ServerName-1` и `www1`. Нaпример пингануть командой 
```bash
ping www1
```
В этот моментсистема оращается к фсистеме ресолвинга, в частности ке файлу `/etc/hosts/` и превращает короткое имя *alias* в полное.

Полное имя __fqdn__ важно для сертификатов, важно когда мы работаем в __domain__ с группой компьютеров. Многие системы работают по имени домейна (по группе имён).

Очень важно делать правильнубю систему имён, а не называтьв сех одним именем. например это могут быть:
- `dmz.firma.com`
- `active-directory.firma.com`
- `remote.firma.com` - для внешних клиентов

При обращении к алиасу `www3` мы хотим чтобы система добавляла "хвост" `firma.com`- __dns suffix__. Эти "хвосты" прописаны в `/etc/resolv.conf`. Например чтобы набрав команду `ping www3` система обращалась к `ping www3.firma.com`.
В файле `/etc/resolv.conf`:
```
nameserver 8.8.8.8  # dns сервер
search firma.com    # dns-suffix, который будет добавлятсья, если имя не было заресолвено в dns-серверах
```
*В системе Windows это также называется dns-suffix и находится в свойствах сетевой карточки*

`/etc/hosts` - превращает короткое имя в полное имя и задаёт статические ip адреса.
Но работает это только для данного компьютера. А для всей сети есть dns server'а.
`/etc/resolv.conf` - содержит список dns серверов и *search*, который дописывает *dns-suffix* неизвестным именам.

Проблема в том, что посе рестарта __Network manager__(CentOS, __systemd-resolve__ в ubuntu) сбросит все настройки.
Чтобы __dhcp-client__ не смог затереть файл ресолва `/etc/resolv.conf` надо добавить ему аттрибут *read-only*:
```bash
chattr +i /etc/resolv.conf  
```
*chattr* - добавка к стандартной security системе линукса (сегодня это тоже стандарт)

### [14. Информация о процессах](https://basis.gnulinux.pro/ru/latest/basis/14/14._%D0%9F%D1%80%D0%BE%D1%86%D0%B5%D1%81%D1%81%D1%8B_%E2%84%961%3A_%D0%98%D0%BD%D1%84%D0%BE%D1%80%D0%BC%D0%B0%D1%86%D0%B8%D1%8F_%D0%BE_%D0%BF%D1%80%D0%BE%D1%86%D0%B5%D1%81%D1%81%D0%B0%D1%85_%E2%84%961.html)

При зауске программы она загружается в оперативную память. Как правило, большие программы загружаются в оперативную память не полностью, а по мере необходимости. При этом для каждой программы создается иллюзия, что она единственная в оперативной памяти. Т.е. для каждой программы создается так называемая виртуальная память. Также в память щагружается не только программа, но и файлы, с которыми она работает (библиотеки, файлы настроек, текущие рабочие файлы, переменные окружение и т.п.). 

Савокупность вычислений, выполняемых программой, виртуальной памяти называется __процессом__. Процессы выполняюится от имени поьзователей. От этого зависят права процесса.

Иногда одной программе бываетнужно выполнить несколько операция параллельно. Для этого один процесс может разделяться на __потоки__. Все потоки используют общую виртуальную память. У каждого процесса есть, как минимум, один поток.

Выполняемая программа - это процесс. Администратору важно видеть список процессов. 
Для этого есть несколько способов.

Утилита __ps__  
```bash
ps                  # список процессов, запушенных в этом терминале
```
```bash
ps -ef              # вывод всех процессов (-e) в виде полного списка (-f)
```
```bash
ps -ef | less -S    # вывод через less без переноса строк
```
- __UID__ - user id пользваотея, что запустил процесс
- __PID__ - process id - уникальный идентификатор процесса, совпадает для потоков одного процесса

- __PPID__ - Parent PID - идентификатор родительского процесса. Почти все прцоессы в сисетме были запущено каким-то процессом. 
- __С__ - % использование процессора данным процессом
- __STIME__ - время запуска процесса
- __TTY__ - телетайп (pts - pseudo terminal)
- __TIME__ - процессороное время потраченное на работу с данным процессом
- __CMD__  - команда, которая запустила процесс (квадратные скобки, еси ps не смог найти аргументы)

Всё есть файл. В unix-подобных системах ипроцессы тоже являются файлами. Ядро создаёт виртуальную файловую систему, которая существует только в оперативной памяти. Виртуальный файловых систем несколько и используются они для разных задач.

Виртуальнаяфайловая система __procfs__ примонтирована в `/proc/`. Директории в `/proc/` - это ид процессов. В `/'proc/` омимо процессов есть и другие файлы, например `/proc/version` - версия ядра. `/proc/uptime` - сколько секунд включена система.

___
### [15. Процессы. Информация о процессах.](https://basis.gnulinux.pro/ru/latest/basis/15/15._%D0%9F%D1%80%D0%BE%D1%86%D0%B5%D1%81%D1%81%D1%8B_%E2%84%962%3A_%D0%98%D0%BD%D1%84%D0%BE%D1%80%D0%BC%D0%B0%D1%86%D0%B8%D1%8F_%D0%BE_%D0%BF%D1%80%D0%BE%D1%86%D0%B5%D1%81%D1%81%D0%B0%D1%85_%E2%84%962.html)
#### Утилита top
__Первая строка__ из вывода __top__ - вывод утилиты __uptime__:
- текущее время
- время рабты системы
- количество пользвоательских сессий (`w` - информация о залогиненых в систему пользователях)
- [средняя загрузка](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbGVFSnp6OU1LNFAwM0lTMGNobzhqLUJqMjJ3d3xBQ3Jtc0tsdktjWFUzaHJoNFdmVUIyUWFGWXlHMnFIZV9CdzloRkQzV0RKUHp4TTB6RURqOVBCMXc3b0hzSm02d0Y0c0JScEtRTHMwOHZOUXlQQV9ReVZCVEFDUzM1WkFwRmFCQlBGR24yX3FpTjE3aHNaZkZMSQ&q=https%3A%2F%2Fru.wikipedia.org%2Fwiki%2FLoad_Average&v=fkC3ZD7qIf8) системы за 1 мин, 5 мин и 15 минут

__Вторая строка__ - информация о запущенных процессах:
- __total__ - общее коичество запущенных процессов
- __running__ - выполняется в данныймомент
- __sleeping__ - сколько процессов спят в ожидании каких-то данных 
- __stopped__ - количество остановленных процессов. Процессы можно сотанавливать. Остановленные процессы остатс в оперативной памяти, но их код перестает выполняться на ЦП.
- __zombie__ - зомби процессы. Дочерние от родительских процессы при выполнении "умирают держа в руках табличку" со статусом завершения процесса. Такой процесс называется зомби. При этом родительский процесс дожен прочитать эту табличку, тем самым отпустив зомби-процесс с покоем. Зомби процессы не используют никаких ресурсов, и пока родительский процесс не прочтёт статус завершения они числятся в таблице процессов. 
Если родительсккая программа не читает статусы заршения, то число процессов будет увеличиваться. Для 32-битных систем максимальное количество потоков - 32768, а для 64-битных более 4 миллионов. Но это количество можно и уменьшить. Увидеть текущий максимум потоков можно в файле `/proc/sys/kernel/pid_max`. 
__ulimit__
Обычно для каждого пользователя стоит ограничеие на количество процессов. Максимальное количество процессов пользователя можно увидеть с помощью команды `ulimit -u`.
Чтобы у пользователя ограничить число процессов надо выполнить команду `ulimit -u 1024`, например записав её в `.bashrc`. Но лучше задавать лимит в `/etc/security/limits.conf`, т.к. у пользователей нет туда доступа на запись. Либо в `/etc/security/limits.d/` создается файл, оканчивающийся на `*.conf` по аналогии с `/etc/limits.conf` (параметр `nproc`).

__Третья строчка.__ Немного информации про использование процессора. Отображается процент вермени, который процессор потратил на те или иные типы задач в промежуток времени обновления информациии:
- __us__ - user space - впемя потрченное процессором на процессы пользовательского пространства. речь идет  всех программах, кроме ядра (для яда есть _kernel space_)
- __sy__ -system time - время потраченное на _kernel space_.
- __ni__ - nice cpu time - время потраченное на процессы с низким приоритетом
Для выставления приоритетов используется утилита __nice__ и число в диапазоне 0..19. Чем больше число, тем вежливее программа. Программа с вежливостью 19 будет уступать процессорное время другим процессам.
  - -20 - наивысший приоритет
  - 19 - самый низкий приоритет
Приоритет по-умолчанию 0 (это можно увидеть выполнив команду `nice`). При этом более высокий приоритет -1..-20 обычный поьзовател задать не может.
```bash
nice -n firefox     # запуск firefox с приоритетом 5
```
```bash
renice -n 10 <PID>   # установить вежливость процесса с PID
```
```bash
ps -el      # просмотр текущего приоритета процессов всех пользователей
```
        PRI - приоритет
        NI - вежливость
Если программа уж запущена, то можно заменить приоритет командой `renice -n 10 firefox`. Причем уменьшить приоритет можно всегда, а дя увеличения приоритета нужны права суперпользоватея __root__.
- __id__ - idle - время, проведенное процессом в ожидании программ.
- __wa__ - input/output wait - время процессора в ожидании операций ввода/вывода (чтения/записи н диск, например)
- __hi__ - hardware interrupts - _аппаратное прерывание_ - сигнал от оборудования процессору о каком-либо соьытии со стороны оборужования. Этовремя процессора, потраценное процессором на аппаратные прерывания.
- __si__ - software interrupts - _програмные прерыания_
- __st__ - steal time - параматр относящийся к виртуальным машинам. говорит о том, какое время реальный процессор бы недоступен виртуальной машине.

__4 строка__ - информация про оперативную память.
`MiB` - mebibites. В 1 КилБайте - 1000 Байт, а в 1 Мегабайте 1000 Килобайт. А вот 1024 - это про Кибибайты и Мебибиайты. 1 KiB = 1024 Bytes, 1 MiB = 1024 KiB
 - _total_ - всего доступно памяти
 - _free_ - свободно памяти
 - _used_ - количество используемой оперативной памяти [linuxatemyram.com](https://linuxatemyram.com)
 - _buff/cache_ - память исползуемая под кэш
Можно ограничить колчиество оеративной памяти, выделяемую каждому процессу. 
Можно прописать параметр в файл ```/etc/security.limits.conf``` с параметром `as`
Для этого нужно выполнить команду ```ulimit -v```, например
```bash
ulimit -v 1024    # ограничть выделение памяти процессу 1024 KB
```
 - _swarp_ - "филиал" оперативки на жёстком диске.
