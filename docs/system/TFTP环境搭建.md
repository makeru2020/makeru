# TFTP环境搭建

1.	在线安装TFTP服务器和客户端（需要保证Ubuntu网络通畅）。

	linux@ubuntu:~$ sudo apt-get install tftpd-hpa tftp-hpa
2.	修改配置文件

	linux@ubuntu:~$sudo vi /etc/default/tftpd-hpa
	
	\#配置文件路径
	
	\#/etc/default/tftpd-hpa
	
	\#用户名
	
	TFTP_USERNAME="tftp"
	
	\#你的tftp服务器所在的路径
	
	TFTP_DIRECTORY="/tftpboot"
	
	\#tftp服务器的网关和端口号
	
	TFTP_ADDRESS="0.0.0.0:69"
	
	\#tftp 文件服务器的可供选择的权限， get file\ put file \ list file
	
	TFTP_OPTIONS="-l -c -s"

3.	创建tftp服务器的目录

	//和配置文件的名字和路径必须保持一致
	
	linux@ubuntu:~$sudo mkdir /tftpboot      
	
	//修改tftp服务器文件夹的权限
	
	linux@ubuntu:~$chmod a + w  tftpboot   
4.	运行下面的命令，重启TFTP服务器。

	linux@ubuntu:~$ sudo service tftpd-hpa restart
5.	运行下面的命令，新建一个文件，并将其移动到TFTP服务器的默认上传下载目录,用于测试tftp服务器是否成功。

	linux@ubuntu:~$ echo "tftp test" > test.txt
	
	linux@ubuntu:~$ sudo mv test.txt /tftpboot/
6.	运行下面的命令，从服务器上下载test.txt文件，并退出tftp程序。

	linux@ubuntu:~$ tftp localhost
	
	tftp> get test.txt
	
	tftp> q
7.	运行下面的命令，确认下载的文件内容正确。

	linux@ubuntu:~$ cat test.txt 
	
	tftp test
8.	如果TFTP的下载不成功，运行下面的命令卸载软件（连同配置信息一起），然后再重新安装，最后再重启TFTP服务器。

	$ sudo apt-get remove --purge tftpd-hpa tftp-hpa
		
	$ sudo apt-get install tftpd-hpa tftp-hpa
	
	$ sudo service tftpd-hpa restart



