# Homework3

*Лог файл homework3.log

## 1.Уменьшить том под / до 8G
 
 #### 1. Подготавливаем том
  
        pvcreate /dev/sdb
        vgcreate vg_root /dev/sdb
        lvcreate -n lv_root -l +100%FREE /dev/vg_root
      
#### 2. Создаем файловую систму и монтируем
 
       mkfs.xfs /dev/vg_root/lv_root
       mount /dev/vg_root/lv_root /mnt
  
#### 3. Копируем все данные с / раздела в /mnt
 
       xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /mnt
 
#### 4. Конфигурируем grub
 
       for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
       chroot /mnt/
       grub2-mkconfig -o /boot/grub2/grub.cfg

#### 5. Обновляем initrd
 
      cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done
      
#### 6. В файле /boot/grub2/grub.cfg меняем d.lvm.lv=VolGroup00/LogVol00 на rd.lvm.lv=vg_root/lv_root

#### 7. Перезагружаемся и меняем размер тома

    lvremove /dev/VolGroup00/LogVol00
    lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00

#### 8. Создаем файловую систему копируем данные

    mkfs.xfs /dev/VolGroup00/LogVol00
    mount /dev/VolGroup00/LogVol00 /mnt
    xfsdump -J - /dev/vg_root/lv_root | xfsrestore -J - /mnt
    
#### 9.
