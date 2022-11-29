# Домашнее задание к занятию "10.4 Резервное копирование" - Кувайсков Дмитрий


### Инструкция по выполнению домашнего задания

   1. Сделайте `fork` данного репозитория к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/git-hw или  https://github.com/имя-вашего-репозитория/7-1-ansible-hw).
   2. Выполните клонирование данного репозитория к себе на ПК с помощью команды `git clone`.
   3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
      - впишите вверху название занятия и вашу фамилию и имя
      - в каждом задании добавьте решение в требуемом виде (текст/код/скриншоты/ссылка)
      - для корректного добавления скриншотов воспользуйтесь [инструкцией "Как вставить скриншот в шаблон с решением](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md)
      - при оформлении используйте возможности языка разметки md (коротко об этом можно посмотреть в [инструкции  по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md))
   4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`);
   5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
   6. Любые вопросы по выполнению заданий спрашивайте в чате учебной группы и/или в разделе “Вопросы по заданию” в личном кабинете.
   
Желаем успехов в выполнении домашнего задания!
   
### Дополнительные материалы, которые могут быть полезны для выполнения задания

1. [Руководство по оформлению Markdown файлов](https://gist.github.com/Jekins/2bf2d0638163f1294637#Code)

---

### Задание 1

В чем отличие между:

	полное резервное копирование,
	дифференциальное резервное копирование,
	инкрементное резервное копирование.
	
`Приведите ответ в свободной форме........`

1. `В полном резервном копировании делаеться полное копирование всего`
2. `В дифференциальном резервном копировании делается только копирование измененных файлов с момента полного копирования`
3. `Инкрементное резервное копирование работает так же как и дифференциальное только точка отсчета это бекап n-1`


---

### Задание 2
	
Установите программное обеспечении Bacula, настройте bacula-dir, bacula-sd, bacula-fd. Протестируйте работу сервисов (трех сервисов).

`Пришлите скриншот рабочей конфигурации`


`При необходимости прикрепитe сюда скриншоты`
![Bacula](https://github.com/Fameq/10.4-hw/blob/main/img/image1.png)


---

### Задание 3

Установите программное обеспечении rsync. Настройте синхронизацию на двух нодах. Протестируйте работу сервиса

`Пришлите скриншот рабочей конфигурации.`


`При необходимости прикрепитe сюда скриншоты
![Название скриншота](ссылка на скриншот)`

```
bacula-dir.conf
Director {
  Name = master-node-dir 
  DIRport = 9101 
  QueryFile = "/etc/bacula/scripts/query.sql" 
  WorkingDirectory = "/var/lib/bacula"
  PidDirectory = "/run/bacula"
  Maximum Concurrent Jobs = 20 
  Password = "12345" 
  Messages = Standard
  DirAddress = 127.0.0.1
}

JobDefs {
  Name = "DefaultJob" 
  Type = Backup 
  Level = Incremental  
  Client = master-node-fd - 
  FileSet = "Local-configuration"  
  Schedule = "LocalDaily" 
  Storage = master-node-sd 
  Messages = Standard
  Pool = LocalPool 
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/var/lib/bacula/%c.bsr"
}

Catalog {
  Name = MyCatalog
  dbname = "bacula"; DB Address = "localhost"; dbuser = "bacula"; dbpassword = "12345"
}

Messages {
  Name = Standard

  mailcommand = "/usr/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula: %t %e of %c %l\" %r"
  operatorcommand = "/usr/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula: Intervention needed for %j\" %r"
  mail = root = all, !skipped
  operator = root = mount
  console = all, !skipped, !saved
  append = "/var/log/bacula/bacula.log" = all, !skipped
  catalog = all
}

Console {
  Name = master-node-mon
  Password = "12345"
  CommandACL = status, .status
}

Pool {
  Name = LocalPool 
  Pool Type = Backup 
  Recycle = yes 
  AutoPrune = yes 
  Volume Retention = 365 days 
  Maximum Volume Bytes = 50G 
  Maximum Volumes = 100 
  Label Format = "Local-"
}

Client {
  Name = master-node-fd 
  Address = localhost 
  FDPort = 9102 
  Catalog = MyCatalog 
  Password = "12345" 
  File Retention = 60 days 
  Job Retention = 6 months 
  AutoPrune = yes 
}

FileSet {
  Name = "Local-configuration"
  Include { 
    Options {
      signature = MD5
    }
    File = /etc
  }
}

Schedule {
  Name = "LocalDaily"
  Run = Level=Full daily at 06:00
}

Job {
  Name = "LocalBackup" 
  JobDefs = "DefaultJob" 
  Enabled = yes 
  Level = Full 
  FileSet = "Local-configuration" 
  Schedule = "LocalDaily" 
  Storage = master-node-sd 
  Pool = "LocalPool"
}

Autochanger {
  Name = master-node-sd
# Do not use "localhost" here
  Address = localhost               
  SDPort = 9101
  Password = "12345"
  Device = Local-Device
  Media Type = File
  Maximum Concurrent Jobs = 10        
  Autochanger = master-node-sd               
}

```

```
bacula-sd.conf

Storage {
  Name = master-node-sd 
  SDPort = 9103 
  WorkingDirectory = "/var/lib/bacula"
  Pid Directory = "/run/bacula"
  Plugin Directory = "/usr/lib/bacula"
  Maximum Concurrent Jobs = 20
  SDAddress = 127.0.0.1 
}

Director {
  Name = master-node-dir 
  Password = "12345"
}

Device {
  Name = Local-Device 
  Media Type = File 
  Archive Device = /backup 
  LabelMedia = yes;
  Random Access = Yes; 
  AutomaticMount = yes;
  RemovableMedia = no; 
  AlwaysOpen = no; 
  Maximum Concurrent Jobs = 5 
}

Messages {
  Name = Standard
  director = master-node-dir = all
}

```
### Задание 4

`Приведите ответ в свободной форме........`

1. `Заполните здесь этапы выполнения, если требуется ....`
2. `Заполните здесь этапы выполнения, если требуется ....`
3. `Заполните здесь этапы выполнения, если требуется ....`
4. `Заполните здесь этапы выполнения, если требуется ....`
5. `Заполните здесь этапы выполнения, если требуется ....`
6. 

```
Поле для вставки кода...
....
....
....
....
```

`При необходимости прикрепитe сюда скриншоты
![Название скриншота](ссылка на скриншот)`

---
## Дополнительные задания (со звездочкой*)

Эти задания дополнительные (не обязательные к выполнению) и никак не повлияют на получение вами зачета по этому домашнему заданию. Вы можете их выполнить, если хотите глубже и/или шире разобраться в материале.

### Задание 5

`Приведите ответ в свободной форме........`

1. `Заполните здесь этапы выполнения, если требуется ....`
2. `Заполните здесь этапы выполнения, если требуется ....`
3. `Заполните здесь этапы выполнения, если требуется ....`
4. `Заполните здесь этапы выполнения, если требуется ....`
5. `Заполните здесь этапы выполнения, если требуется ....`
6. 

`При необходимости прикрепитe сюда скриншоты
![Название скриншота](ссылка на скриншот)`
