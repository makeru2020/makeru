# NFS环境搭建

1.	运行下面的命令安装NFS服务器（需要保持网络通畅）。

	linux@ubuntu:~ $ sudo apt-get install nfs-kernel-server
2.	运行下面的命令，创建一个目录，并在该文件下创建一个文件，用于测试nfs。

	linux@ubuntu:~ $ sudo mkdir /nfs
	
	linux@ubuntu:~ $ mkdir /nfs/rootfs
	
	linux@ubuntu:~ $ echo "nfs test" > /nfs/rootfs/test.txt
3.	编辑/etc/exports配置文件。

	linux@ubuntu:~ $ sudo vim /etc/exports
	
	添加如下内容：
	
	/nfs/rootfs  *(rw,sync,no_subtree_check,no_root_squash)
	
	其中：
	
	/nfs/rootfs：共享的目录；
	
	*：不限定客户端；
	
	rw：共享目录可读可写；
	
	sync：将数据同步写入内存缓冲区与磁盘中，效率低，但可以保证数据的一致性；
	
	no_subtree_check ：即使输出目录是一个子目录，nfs服务器也不检查其父目录的权限，这样可以提高效率；
	
	no_root_squash：来访的root用户保持root帐号权限；
	
4.	Ubuntu17.10及以后版本需要增加以下配置，因为新版本Ubuntu只支持nfs 3和nfs 4，而uboot默认使用nfs 2

	linux@ubuntu:~ $sudo vi /etc/default/nfs-kernel-server
	
	RPCNFSDOPTS="--nfs-version 2,3,4 --debug --syslog"
5.	使用下面的命令，重启NFS服务。

	linux@ubuntu:~ $ sudo service nfs-kernel-server restart
6.	使用下面的命令，将共享目录挂在到/mnt目录下，并修文件。

	linux@ubuntu:~ $ sudo mount -t nfs localhost:/nfs/rootfs /mnt
	
	linux@ubuntu:~ $ vim /mnt/test.txt
7.	使用下面的命令，查看原来的文件已经被修改。

	linux@ubuntu:~ $ cat /nfs/rootfs/test.txt
8.	使用下面的命令取消挂载。

	linux@ubuntu:~ $ sudo umount /mnt



