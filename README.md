# Homework3

*Log файл учебного стенда homework3.log*

## 1.Уменьшить том под / до 8G
 
 #### 1. Подготавливаем том
  
        pvcreate /dev/sdb
        vgcreate vg_root /dev/sdb
        lvcreate -n lv_root -l +100%FREE /dev/vg_root
      
##### 2. Создаем файловую систму и монтируем
 
       mkfs.xfs /dev/vg_root/lv_root
       mount /dev/vg_root/lv_root /mnt
  
##### 3. Копируем все данные с / раздела в /mnt
 
       xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /mnt
 
##### 4. Конфигурируем grub
 
       for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
       chroot /mnt/
       grub2-mkconfig -o /boot/grub2/grub.cfg

##### 5. Обновляем initrd
 
      cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done
      
##### 6. В файле /boot/grub2/grub.cfg меняем d.lvm.lv=VolGroup00/LogVol00 на rd.lvm.lv=vg_root/lv_root

##### 7. Перезагружаемся и меняем размер тома

    lvremove /dev/VolGroup00/LogVol00
    lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00

##### 8. Создаем файловую систему копируем данные

    mkfs.xfs /dev/VolGroup00/LogVol00
    mount /dev/VolGroup00/LogVol00 /mnt
    xfsdump -J - /dev/vg_root/lv_root | xfsrestore -J - /mnt
    
##### 9. Конфигурируем grub

    for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
    chroot /mnt/
    grub2-mkconfig -o /boot/grub2/grub.cfg
    cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done
    
## 2. Выделить том под /var в зеркало

##### 1. Создаем зеркало

    pvcreate /dev/sdc /dev/sdd
    vgcreate vg_var /dev/sdc /dev/sdd
    lvcreate -L 950M -m1 -n lv_var vg_var
    
##### 2. Форматируем и монтируем

    mkfs.ext4 /dev/vg_var/lv_var
    mount /dev/vg_var/lv_var /mnt
    cp -aR /var/* /mnt/      # rsync -avHPSAX /var/ /mnt/
    mkdir /tmp/oldvar && mv /var/* /tmp/oldvar
    umount /mnt
    mount /dev/vg_var/lv_var /var

##### 3. Правим fstab

    echo "`blkid | grep var: | awk '{print $2}'` /var ext4 defaults 0 0" >> /etc/fstab
    
##### 4. Перезагружаемся и удаляем временный root

    lvremove /dev/vg_root/lv_root
    vgremove /dev/vg_root
    pvremove /dev/sdb
    
## 3. Выделить том под /home

##### 1. Выделяем том под /home

    lvcreate -n LogVol_Home -L 2G /dev/VolGroup00

##### 2. Создаем ФС

    mkfs.xfs /dev/VolGroup00/LogVol_Home

##### 3. Монтируем

     mount /dev/VolGroup00/LogVol_Home /mnt/
     cp -aR /home/* /mnt/  
     rm -rf /home/*
     umount /mnt
     mount /dev/VolGroup00/LogVol_Home /home/

##### 4. Редактируем fstab

    echo "`blkid | grep Home | awk '{print $2}'` /home xfs defaults 0 0" >> /etc/fstab
    
## 4. /home - сделать том для снапшотов

##### 1. Генерируем файлы

    touch /home/file{1..20}
    
##### 2. Делаем снапшот

    lvcreate -L 100MB -s -n home_snap /dev/VolGroup00/LogVol_Home
    
##### 3. Удаляем файлы

    rm -f /home/file{11..20}
    
##### 4. Восстанавливаем файлы

    umount /home
    lvconvert --merge /dev/VolGroup00/home_snap
    mount /home


