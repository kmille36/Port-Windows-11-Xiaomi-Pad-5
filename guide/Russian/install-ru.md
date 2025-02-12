﻿﻿<img align="right" src="https://raw.githubusercontent.com/erdilS/Port-Windows-11-Xiaomi-Pad-5/main/nabu.png" width="425" alt="Windows 11 Running On A Xiaomi Pad 5">


# Windows на Xiaomi Pad 5

## Установка

### Требования

- [ARM образ Windows](https://uupdump.net/)
- [Образ UEFI](/images/)
- [Скрипт монтирования разделов](../../../../releases/tag/1.0)
- [DriverUpdater](https://github.com/WOA-Project/DriverUpdater/releases/latest)
- [Драйверы](https://github.com/map220v/MiPad5-drivers)

### Перезапустите рекавери чтобы начать установку Windows

```cmd
fastboot boot <recovery.img>
```

#### Скопируйте скрипт msc в /sbin

```cmd
adb push msc.sh /sbin/
```

#### Выполните скрипт msc

```cmd
adb shell sh /sbin/msc.sh
```

### Привязка букв к разделам
  

#### Запустите Менеджер дисков Windows

> Как только планшет определился как диск

```cmd
diskpart
```


#### Привязка буквы  `X` к разделу Windows

#### Выберите Windows раздел планшета
> Используйте команду `list volume` чтобы найти разделы "WINNABU" и "ESPNABU"

```diskpart
select volume <number>
```

#### Привяжите букву X
```diskpart
assign letter=x
```

### Привязка буквы  `Y`  к разделу ESP

#### Выберите ESP раздел планшета
> Используйте команду `list volume` чтобы найти его, обычно это последний раздел

```diskpart
select volume <number>
```

#### Привяжите букву Y

```diskpart
assign letter=y
```

#### Закройте diskpart
```diskpart
exit
```

  
  

### Установка Windows

> Замените `<path/to/install.wim>` действительным путём к файлу `install.wim`, который расположен в папке `sources` внутри вашего ISO. Вы можете получить его, смонтировав образ или разархивировав его

```cmd
dism /apply-image /ImageFile:<path/to/install.wim> /index:1 /ApplyDir:X:\
```

### Установка драйверов

> Замените `<nabudriversfolder>` расположением папки с драйверами

```cmd
driverupdater.exe -d <nabudriversfolder>\definitions\Desktop\ARM64\Internal\nabu.txt -r <nabudriversfolder> -p X:
```

  

### Создайте файлы загрузчика Windows для EFI

```cmd
bcdboot X:\Windows /s Y: /f UEFI
```

  
  

### Разрешите использование неподписанных драйверов

> Если вы не сделаете этого, то получите BSOD

```cmd
bcdedit /store Y:\EFI\Microsoft\BOOT\BCD /set {default} testsigning on
```


## Запуск Windows

### Создайте резервную копию текущего ядра Android

```cmd
adb shell "dd if=/dev/block/bootdevice/by-name/boot$(getprop ro.boot.slot_suffix) of=/tmp/boot.img"
```

### Скопируйте РК на компьютер

```cmd
adb pull /tmp/boot.img
```
### Выясните какой у вас дисплей

```cmd
adb shell "dmesg | grep dsi_display_bind"
```

- Если у вас дисплей от Huaxing вывод команды будет таким ```dsi_k82_42_02_0a_dual_cphy_video```
- Если у вас дисплей от Tianma вывод команды будет таким ```dsi_k82_36_02_0b_dual_cphy_video```

### Перезапустите планшет в загрузчик 

```cmd
adb reboot bootloader
```

### Скачайте и прошейте образ UEFI для вашего дисплея
> Образ для дисплея [Huaxing](/../../raw/main/images/xiaomi-nabu_huaxing.img) и [Tianma](/../../raw/main/images/xiaomi-nabu_tianma.img)
```cmd
fastboot flash boot <путь к образу UEFI>
```

### Загрузка в Android
> Прошейте скопированное ранее ядро в режиме загрузки

```cmd
fastboot flash boot boot.img
```

## Готово!
