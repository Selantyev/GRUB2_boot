Vagrantfile поднимает виртуалку с 4 дисками (LVM)

***Попасть в систему без пароля несколькими способами***

Способ 1. init=/bin/sh

1. Открываем GUI VirtualBox, запускаем виртуальную машину и при выборе ядра для загрузки нажимаем клавишу E (edit). Попадаем в окно где мы можем изменить параметры загрузки
2. В конце строки с загрузочными параметрами, начинающейся с linux16 добавляем init=/bin/sh и убираем console=ttyS0,115200n8

`linux 16 /vmlinuz-3.10.0-862.2.3.el7.x86_64 root=/dev/mapper/VolGroup00-LogVol00 ro no_timer_check console=tty0 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=VolGroup00/LogVol00 rd.lvm.lv=VolGroup00/LogVol01 rhgb quite init=/bin/sh`

3. Нажимаем CTRL+X для загрузки системы
4. Попадаем в систему без пароля. Файловая система root смонтирована в режиме Read Only

`# touch /usr/local/file.txt`

`touch: cannot touch 'touch /usr/local/file.txt': Read-only file system`

`# mount`

`/dev/mapper/VolGroup00-LogVol00 on / type xfs (ro,realtime,attr2,innode64,noquota)`

5. Перемонтируем файловую систему root в режим Read-Write

`# mount -o remount,rw /`

`# mount`

`/dev/mapper/VolGroup00-LogVol00 on / type xfs (rw,realtime,attr2,innode64,noquota)`

6. Перезагружаемся

`# reboot -f`

Способ 2. rd.break

1. Открываем GUI VirtualBox, запускаем виртуальную машину и при выборе ядра для загрузки нажимаем клавишу E (edit). Попадаем в окно где мы можем изменить параметры загрузки
2. В конце строки с загрузочными параметрами, начинающейся с linux16 добавляем rd.break и убираем console=ttyS0,115200n8

`linux 16 /vmlinuz-3.10.0-862.2.3.el7.x86_64 root=/dev/mapper/VolGroup00-LogVol00 ro no_timer_check console=tty0 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=VolGroup00/LogVol00 rd.lvm.lv=VolGroup00/LogVol01 rhgb quite rd.break`

3. Нажимаем CTRL+X для загрузки системы
4. Попадаем в систему без пароля. Файловая система root смонтирована в режиме Read Only

`# touch /usr/local/file.txt`

`touch: cannot touch 'touch /usr/local/file.txt': Read-only file system`

`# mount`

`/dev/mapper/VolGroup00-LogVol00 on / type xfs (ro,realtime,attr2,innode64,noquota)`

5. Перемонтируем файловую систему root в режим Read-Write

`# mount -o remount,rw /`

`# mount`

`/dev/mapper/VolGroup00-LogVol00 on / type xfs (rw,realtime,attr2,innode64,noquota)`

6. Поменяем пароль root

`/# chroot /sysroot`

`# passwd root`

`touch /.autorelabel #если SELinux включен`

7. Перезагружаемся

`# reboot -f`

Способ 3. rw init=/sysroot/bin/sh

1. Открываем GUI VirtualBox, запускаем виртуальную машину и при выборе ядра для загрузки нажимаем клавишу E (edit). Попадаем в окно где мы можем изменить параметры загрузки
2. В конце строки с загрузочными параметрами, начинающейся с linux16 заменяем ro на rw init=/sysroot/bin/sh и убираем console=ttyS0,115200n8

`linux 16 /vmlinuz-3.10.0-862.2.3.el7.x86_64 root=/dev/mapper/VolGroup00-LogVol00 init=/sysroot/bin/sh no_timer_check console=tty0 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=VolGroup00/LogVol00 rd.lvm.lv=VolGroup00/LogVol01 rhgb quite rd.break`

3. Нажимаем CTRL+X для загрузки системы
4. Попадаем в систему без пароля. Файловая система root смонтирована в режиме Read-Write
5. Перезагружаемся

`# reboot -f`

**Установить систему с LVM, после чего переименовать VG**

1. Проверим состояние групп

`# vgs`

`VG #PV #LV #SN Attr VSize VFree`

`VolGroup00 1 2 0 wz--n- <38.97g 0`

2. Переименуем VolGroup00 в OtusRoot

`# vgrename VolGroup00 OtusRoot`

`Volume group "VolGroup00" successfully renamed to "OtusRoot"`

3. Заменяем VolGroup00 на OtusRoot в файлах - /etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg

`# sed -i 's/VolGroup00/OtusRoot/g' /etc/fstab`

`# sed -i 's/VolGroup00/OtusRoot/g' /etc/default/grub`

`# sed -i 's/VolGroup00/OtusRoot/g' /boot/grub2/grub.cfg`

4. Пересоздадим initrd image, чтобы он знал новое название Volume Group

`# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)`

5. Перезагружаемся

`# reboot`

6. Проверим состояние групп

`# vgs`

`VG #PV #LV #SN Attr VSize VFree`

`OtusRoot 1 2 0 wz--n- <38.97g 0`

Таким же образом можно переименовать LogVol00

**Добавить модуль в initrd**

Скрипты модулей хранятся в каталоге /usr/lib/dracut/modules.d/

1. Создаем каталог

`mkdir /usr/lib/dracut/modules.d/01mytestmodule`

2. Поместим в каталог 01mytestmodule скрипт module-setup.sh

`# vi /usr/lib/dracut/modules.d/01mytestmodule/module-setup.sh`

```
#!/bin/bash

check() {
    return 0
}

depends() {
    return 0
}

install() {
    inst_hook cleanup 00 "${moddir}/test.sh"
}
```

3. Поместим в каталог 01mytestmodule скрипт test.sh

`# vi /usr/lib/dracut/modules.d/01mytestmodule/test.sh`

```
#!/bin/bash

exec 0<>/dev/console 1<>/dev/console 2<>/dev/console
cat <<'msgend'
Hello! You are in dracut module!
 ___________________
< I'm dracut module >
 -------------------
   \
    \
        .--.
       |o_o |
       |:_/ |
      //   \ \
     (|     | )
    /'\_   _/`\
    \___)=(___/
msgend
sleep 10
echo " continuing...."
```

4. Пересоздадим initrd image

`mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)`

или

`dracut -f -v`

5. Проверим какие модули загружены в образ

`lsinitrd -m /boot/initramfs-$(uname -r).img | grep mytestmodule`

6. Перезагружаемся

`# reboot`

7. В конце строки с загрузочными параметрами, начинающейся с linux16 убираем rghb и quiet

`linux 16 /vmlinuz-3.10.0-862.2.3.el7.x86_64 root=/dev/mapper/VolGroup00-LogVol00 init=/sysroot/bin/sh no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=VolGroup00/LogVol00 rd.lvm.lv=VolGroup00/LogVol01`

8. Нажимаем CTRL+X для загрузки системы
    
9. Видим в выводе модуль пингвина

```
Hello! You are in dracut module!

 ___________________
< I'm dracut module >
 -------------------
   \
    \
        .--.
       |o_o |
       |:_/ |
      //   \ \
     (|     | )
    /'\_   _/`\
    \___)=(___/
```
