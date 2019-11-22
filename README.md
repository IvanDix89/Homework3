# Homework3
===========

*Лог файл homework3.log

 ## 1.Уменьшить том под / до 8G
 
   ### 1.1 Подготавливаем топ
  
      pvcreate /dev/sdb
      vgcreate vg_root /dev/sdb
      lvcreate -n lv_root -l +100%FREE /dev/vg_root
