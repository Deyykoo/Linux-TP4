# Linux-TP4

## Partie 1 : Partitionnement du serveur de stockage

```
🌞 Partitionner le disque à l'aide de LVM

*
    - [quentin@storage ~]$ sudo pvcreate /dev/sdb
    Physical volume "/dev/sdb" successfully created.
    [quentin@storage ~]$ sudo pvcreate /dev/sdc
    Physical volume "/dev/sdc" successfully created.

    [quentin@storage ~]$ sudo pvs
    PV         VG Fmt  Attr PSize  PFree
    /dev/sda2  rl lvm2 a--  <7.00g    0
    /dev/sdb      lvm2 ---   2.00g 2.00g
    /dev/sdc      lvm2 ---   2.00g 2.00g

*
    - [quentin@storage ~]$ sudo vgcreate data /dev/sdb
    Volume group "data" successfully created 

    [quentin@storage ~]$ sudo vgextend data /dev/sdc
    Volume group "data" successfully extended

    [quentin@storage ~]$ sudo vgs
    VG   #PV #LV #SN Attr   VSize  VFree
    data   2   0   0 wz--n-  3.99g 3.99g
    rl     1   2   0 wz--n- <7.00g    0

    Rename du VG data  ( j'avais pas vu )
    [quentin@storage ~]$ sudo vgrename /dev/data  
    /dev/storage
    [sudo] password for quentin:
    Volume group "data" successfully renamed to "storage"
    [quentin@storage ~]$ sudo vgs
    VG      #PV #LV #SN Attr   VSize  VFree
    rl        1   2   0 wz--n- <7.00g    0
    storage   2   0   0 wz--n-  3.99g 3.99g

*
    - [quentin@storage ~]$ sudo lvcreate -l 100%FREE storage 
    -n 
    last_data
    [sudo] password for quentin:
    Logical volume "last_data" created.
    [quentin@storage ~]$ sudo lvs
    LV        VG      Attr       LSize   Pool Origin Data%  
    Meta%  Move Log Cpy%Sync Convert
    root      rl      -wi-ao----  <6.20g
    swap      rl      -wi-ao---- 820.00m
    last_data storage -wi-a-----   3.99g
    


🌞 Formater la partition

    - [quentin@storage ~]$ sudo mkfs -t ext4 
    /dev/storage/last_data
    mke2fs 1.46.5 (30-Dec-2021)
    Creating filesystem with 1046528 4k blocks and 261632 
    inodes
    Filesystem UUID: 3670fe25-18b6-4846-b17e-f0465efb5703
    Superblock backups stored on blocks:
    32768, 98304, 163840, 229376, 294912, 819200, 884736

    Allocating group tables: done
    Writing inode tables: done
    Creating journal (16384 blocks): done
    Writing superblocks and filesystem accounting information: 
    done


🌞 Monter la partition
*
    - [quentin@storage ~]$ df -h | grep data1
    /dev/mapper/storage-last_data  3.9G   24K  3.7G   1% 
    /mnt/data1

*
    - [quentin@storage ~]$ sudo nano /dev/storage/last_data
    [sudo] password for quentin:
    [quentin@storage ~]$ sudo cat PARTITION
    CHOCOLATINE MONSIEUR

*

    - [quentin@storage ~]$ sudo cat /etc/fstab
    [sudo] password for quentin:

    #
    # /etc/fstab
    # Created by anaconda on Mon Oct 23 09:14:23 2023
    #
    # Accessible filesystems, by reference, are maintained 
    under '/dev/disk/'.
    # See man pages fstab(5), findfs(8), mount(8) and/or 
    blkid(8) for more info.
    #
    # After editing this file, run 'systemctl daemon-reload' 
    to update systemd
    # units generated from this file.
    #
    /dev/mapper/rl-root     /                       xfs     
    defaults        0 0
    UUID=56911e98-6d33-44a8-b9eb-0a1cd72c7260 /boot                   
    xfs     defaults        0 0
    /dev/mapper/rl-swap     none                    swap    
    defaults        0 0
    /dev/storage/last_data /storage  ext4 defaults 0 0
```

## Partie 2 : Serveur de partage de fichiers
``` 
🌞 Donnez les commandes réalisées sur le serveur NFS storage.tp4.linux

*
    -[quentin@web ~]$ sudo dnf install nfs-utils
     [quentin@storage ~]$ sudo mkdir /storage/site_web_1 -p
      [quentin@storage ~]$ sudo mkdir /storage/site_web_2

    - [quentin@storage ~]$ sudo chown nobody 
      /storage/site_web_1
      [quentin@storage ~]$ sudo chown nobody 
      /storage/site_web_2
    
    - [quentin@storage ~]$ sudo systemctl enable nfs-server
      Created symlink /etc/systemd/system/multi-
      user.target.wants/nfs-server.service → 
      /usr/lib/systemd/system/nfs-server.service.

    - [quentin@storage ~]$ sudo systemctl start nfs-server
      [quentin@storage ~]$ sudo systemctl status nfs-server
    ● nfs-server.service - NFS server and services
      Loaded: loaded (/usr/lib/systemd/system/nfs-
      server.service; enabled; preset: disabled)
      Drop-In: /run/systemd/generator/nfs-server.service.d
             └─order-with-mounts.conf
      Active: active (exited) since Tue 2024-02-20 09:36:28 
      CET; 1min 19s ago
      Process: 11941 ExecStartPre=/usr/sbin/exportfs -r 
      (code=exited, status=1/FAILURE)
      Process: 11942 ExecStart=/usr/sbin/rpc.nfsd 
      (code=exited, status=0/SUCCESS)
      Process: 11961 ExecStart=/bin/sh -c if systemctl -q is-
      active gssproxy; then systemctl reload gssproxy ; fi 
      (code=e>   Main PID: 11961 (code=exited, 
      status=0/SUCCESS)
      CPU: 19ms

    - [quentin@storage ~]$ sudo firewall-cmd --permanent --
      add-service=nfs
      success
      [quentin@storage ~]$ sudo firewall-cmd --permanent --
      add-service=mountd
      success
      [quentin@storage ~]$ sudo firewall-cmd --permanent --
      add-service=rpc-bind
      success
      [quentin@storage ~]$ sudo firewall-cmd --reload
      success
      [quentin@storage ~]$ sudo firewall-cmd --permanent --
      list-all | grep services
      services: cockpit dhcpv6-client mountd nfs rpc-bind ssh


🌞 Donnez les commandes réalisées sur le client NFS client.tp4.linux
*
    - [quentin@web ~]$ sudo dnf install nfs-utils*
      [quentin@web ~]$ sudo mkdir -p /var/www/site_web_1
      [quentin@web ~]$ sudo mkdir -p /var/www/site_web_2

    - [quentin@web ~]$ sudo mount 
      10.2.1.22:/storage/site_web_1 /var/www/site_web_1
      [quentin@web ~]$ sudo mount 
      10.2.1.22:/storage/site_web_2 /var/www/site_web_2

      [quentin@web ~]$ df -h
      Filesystem                     Size  Used Avail Use% 
      Mounted on
      devtmpfs                       4.0M     0  4.0M   0% 
      /dev
      tmpfs                          386M     0  386M   0% /
      dev/shm
      tmpfs                          155M  3.7M  151M   3% 
      run
      /dev/mapper/rl-root            6.2G  1.2G  5.0G  20% /
      /dev/sda1                     1014M  221M  794M  22% 
      /boot
      tmpfs                           78M     0   78M   0% 
      /run/user/1000
      10.2.1.22:/storage/site_web_1  3.9G     0  3.7G   0% 
      /var/www/site_web_1
      10.2.1.22:/storage/site_web_2  3.9G     0  3.7G   0% 
      /var/www/site_web_2

    - [quentin@web ~]$ ls -l 
      /var/www/site_web_1/CHOCOLATINEEE
      -rw-r--r--. 1 root root 0 Feb 20 17:48 
      /var/www/site_web_1/CHOCOLATINEEE

    - [quentin@web ~]$ sudo touch 
      /var/www/site_web_2/CROISSANT
      [quentin@web ~]$ ls -l /var/www/site_web_2/CROISSANT
      -rw-r--r--. 1 root root 0 Feb 20 17:51 
      /var/www/site_web_2/CROISSANT

    - [quentin@web ~]$ sudo cat /etc/fstab

      #
      # /etc/fstab
      # Created by anaconda on Mon Oct 23 09:14:23 2023
      #
      # Accessible filesystems, by reference, are maintained 
      under '/dev/disk/'.
      # See man pages fstab(5), findfs(8), mount(8) and/or 
      blkid(8) for more info.
      #
      # After editing this file, run 'systemctl daemon-reload' 
      to update systemd
      # units generated from this file.
      #
      /dev/mapper/rl-root     /                       xfs     
      defaults        0 0
      UUID=56911e98-6d33-44a8-b9eb-0a1cd72c7260 /boot                   
      xfs     defaults        0 0
      /dev/mapper/rl-swap     none                    swap    
      defaults        0 0

      10.2.1.22:/storage/site_web_1    /var/www/site_web_1   
      nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
      10.2.1.22:/storage/site_web_2    /var/www/site_web_2   
      nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
```

## Partie 3 : Serveur web
```
🌞 Installez NGINX

  - [quentin@web ~]$ sudo dnf install nginx



🌞 Analysez le service NGINX
*
  - [quentin@web ~]$ ps -ef | grep nginx
    root        1720       1  0 18:34 ?        00:00:00 nginx: 
    master process /usr/sbin/nginx
    nginx       1721    1720  0 18:34 ?        00:00:00 nginx: 
    worker process
    quentin     1731    1371  0 18:38 pts/0    00:00:00 grep -
    -color=auto nginx


  - [quentin@web ~]$ sudo ss -alnpt | grep nginx
    LISTEN 0      511          0.0.0.0:80        0.0.0.0:*    
    users:(("nginx",pid=1721,fd=6),("nginx",pid=1720,fd=6))
    LISTEN 0      511             [::]:80           [::]:*    
    users:(("nginx",pid=1721,fd=7),("nginx",pid=1720,fd=7))


  - [quentin@web nginx]$ cat nginx.conf | grep nginx
    #   * Official English Documentation: 
    http://nginx.org/en/docs/
    #   * Official Russian Documentation: 
    http://nginx.org/ru/docs/
    user nginx;
    error_log /var/log/nginx/error.log;
    pid /run/nginx.pid;
    # Load dynamic modules. See 
    /usr/share/doc/nginx/README.dynamic.
    include /usr/share/nginx/modules/*.conf;
        access_log  /var/log/nginx/access.log  main;
        include             /etc/nginx/mime.types;
        # Load modular configuration files from the 
        /etc/nginx/conf.d directory.
        # See 
        http://nginx.org/en/docs/ngx_core_module.html#include
        include /etc/nginx/conf.d/*.conf;
            root         /usr/share/nginx/html;
            include /etc/nginx/default.d/*.conf;
    #        root         /usr/share/nginx/html;
    #        ssl_certificate "/etc/pki/nginx/server.crt";
    #        ssl_certificate_ke 
            "/etc/pki/nginx/private/server.key";
    #        include /etc/nginx/default.d/*.conf;


    - [quentin@web nginx]$ ls -l nginx.conf
      -rw-r--r--. 1 root root 2334 Oct 16 20:00 nginx.conf



🌞 Configurez le firewall pour autoriser le trafic vers le service NGINX
*
    - [quentin@web nginx]$ sudo firewall-cmd --add-port=80/tcp 
    --permanent
    success

    - [quentin@web nginx]$ sudo firewall-cmd --reload
    success


🌞 Accéder au site web
*
    - [quentin@web nginx]$ sudo curl 10.2.1.21
      [sudo] password for quentin:
      <!doctype html>
      <html>
          <head>
            <meta charset='utf-8'>
            <meta name='viewport' content='width=device-width, 
            initial-scale=1'>
            <title>HTTP Server Test Page powered by: Rocky 
            Linux</title>
            <style type="text/css">
              /*<![CDATA[*/

              html {

        ( j'ai seulement mis le début du code )



🌞 Vérifier les logs d'accès
*
    - [quentin@web log]$ tail -n 3 dnf.log
      2024-02-20T18:34:13+0100 DDEBUG 
      /var/cache/dnf/appstream-
      25485261a76941d3/packages/rocky-logos-httpd-90.14-
      2.el9.noarch.rpm removed
      2024-02-20T18:34:13+0100 DDEBUG 
      /var/cache/dnf/appstream-
      25485261a76941d3/packages/nginx-filesystem-1.20.1-
      14.el9_2.1.noarch.rpm removed
      2024-02-20T18:34:13+0100 DDEBUG Plugins were unloaded.


🌞 Changer le port d'écoute
*
    - [quentin@web nginx]$ sudo cat nginx.conf | grep listen
        listen       8080;

    - [quentin@web nginx]$ sudo ss -alnpt | grep 8080
      LISTEN 0      511          0.0.0.0:8080      0.0.0.0:*    
      users:(("nginx",pid=4660,fd=6),("nginx",pid=4659,fd=6))
```