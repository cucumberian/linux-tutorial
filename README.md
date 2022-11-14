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
