<div align="center">

# ArchLinux-Bootstrap-Guide
Это личная памятка по bootstrapping'у с командами, созданная на основе гайда по установке на [Arch wiki](https://wiki.archlinux.org/title/Installation_guide)

![arch_users](Source/arch_users.PNG)
</div>

# 1. Перед установкой

## Установка раскладки клавиатуры и шрифта
Установка раскладки клавиатуры:
```shell
loadkeys ru
```
Установка шрифта:
```shell
setfont ter-c32b
```

## Соединение с интернетом
Запуск в интерактивном режиме [iwd]#: 
```shell
iwctl
```

Запросить список всех Wi-Fi устройств:
```shell
device list
```
> [!IMPORTANT]
> Если устройство или соответствующий адаптер выключен, включите его:
> ```shell
> device устройство set-property Powered on
> ```
> ```shell
> adapter адаптер set-property Powered on
> ```

Запустить сканирование сети (команда ничего не выведет):
```shell
station устройство scan
```

Вывести список обнаруженных сетей:
```shell
station устройство get-networks
```

Подключится к сети:
```shell
station устройство connect SSID
```

Выйти из [iwd]: 
```shell
exit
```

Проверка подключения (вместо 8.8.8.8 можно использовать любой ресурс):
```shell
ping 8.8.8.8
```


## Разметка дисков
Чтобы посмотреть список накопителей, используйте:
```shell
fdisk -l
```
или
```shell
lsblk
```

Для изменения таблицы разделов:
```shell
fdisk /dev/диск_для_разметки
```

> [!TIP]  
> Таблица-пример разметки:
> | Точка монтирования | Раздел                    | Тип раздела        | Рекомендуемый размер     |
> |:-------------------|:--------------------------|:-------------------|:-------------------------|
> |/boot               |/dev/*системный_раздел_efi*|Системный раздел EFI|1 ГиБ                     |
> |[SWAP]              |/dev/*раздел_подкачки*     |Linux swap          |Не менее 4 ГиБ(x2RAM)     |
> |/                   |/dev/*корневой_раздел*     |Linux x86-64 root   |Остаток, минимум 23–32 ГиБ|
> |/home               |/dev/*домашний_раздел*     |Linux lifesystem    |Остаток                   |

## Форматирование разделов
/boot:
```shell
mkfs.fat -F 32 /dev/системный_раздел_efi
```
[SWAP]:
```shell
mkswap /dev/раздел_подкачки_Linux_swap
```
/:
```shell
mkfs.ext4 /dev/корневой_раздел
```
/home:
```shell
mkfs.ext4 /dev/домашний_католог_home
```

## Монтирование разделов
/boot:
```shell
mount --mkdir /dev/системный_раздел_efi /mnt/boot
```
[SWAP]:
```shell
swapon /dev/раздел_подкачки
```
/:
```shell
mount /dev/корневой_раздел /mnt
```
/home:
```shell
mount --mkdir /dev/домашний_раздел /mnt/home
```
Для проверки верности монтирования:
```shell
lsblk
```

# 2. Установка
Установка основных пакетов:
```shell
pacstrap -K /mnt base linux linux-firmware
```


# 3. Настройка
## Fstab
Сгенерировать файл fstab:
```shell
genfstab -U /mnt >> /mnt/etc/fstab
```

***

## Chroot
Перейти к корневому каталогу новой системы:
```shell
arch-chroot /mnt
```
***

## Время
Задать часовой пояс:
```shell
ln -sf /usr/share/zoneinfo/Регион/Город /etc/localtime
```
Сгенерировать ```/etc/adjtime```:
```shell
hwclock --systohc
```

***

## Локализация
Установить ```nano```:
```shell
pacman -Sy nano
```

Отредактировать файл ```/etc/locale.gen```:

```shell
nano /etc/locale.gen
```
> Раскомментировать:
> ```shell
> #en_US.UTF-8 UTF-8
> ```
> и другие необходимые UTF-8 локали, например:
> ```shell
> #ru_RU.UTF-8 UTF-8
> ```

Сгенерировать локали:
```shell
locale-gen
```

Создать файл ```locale.conf```:
```shell
nano /etc/locale.conf
```
> и задать переменной ```LANG``` необходимое значение:
> ```shell
> LANG=ru_RU.UTF-8
> ```

Сделайте изменения раскладки и шрифта постоянными:
```shell
nano /etc/vconsole.conf
```
> прописав их в файле ```vconsole.conf```:
> ```shell
> KEYMAP=ru
> FONT=cyr-sun16
> ```

***

## Настройка сети
Создать файл ```hostname```:
```shell
nano /etc/hostname
```
> и написать туда:  
> *имявашегохоста*

Установить ```NetworkManager```:
```shell
pacman -S networkmanager
```
и включаем его:
```shell
systemctl enable NetworkManager
```

***

## Cуперпользователь
Установить пароль суперпользователя:
```shell
passwd
```

***

## Новый пользователь
Создать нового пользователя:
```shell
useradd -m -G wheel -s /bin/bash имяпользователя
```
Задать для нового пользователя пароль:
```shell
passwd имяпользователя
```
Скачать ```sudo```:
```shell
pacman -S sudo
```
Открыть файл конфигурации:
```shell
EDITOR=nano visudo
```
> и раскомментировать строку:
> ```shell
> # %wheel ALL=(ALL:ALL) ALL
> ```

***

## Загрузчик
Скачать загрузчик ```grub```:
```shell
pacman -S grub efibootmgr
```
Установить его:
```shell
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```
Сгенерировать конфиг ```GRUB```:
```shell
grub-mkconfig -o /boot/grub/grub.cfg
```

***

# 4. Перезагрузка
Выйти из ```chroot```:
```shell
exit
```
Размонтировать все разделы:
```shell
umount -R /mnt
```
Выключить систему:
```shell
poweroff
```

> [!IMPORTANT]  
> Перед повторным включением вытащите установочную флешку

***

# 5. После установки
## Подключение к wifi
Список доступных подключений:
```shell
nmcli connection show
```
Список активных устройств:
```shell
nmcli device status
```
Сканируйте доступные сети:
```shell
nmcli device wifi list
```
Подключится к выбранной сети:
```shell
nmcli device wifi connect "имя_сети" password "пароль"
```