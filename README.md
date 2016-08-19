# How to install UniFi Controller on ClearOS 7.2

## The origin of this is here, but targeted at CentOS 7. ClearOS 7.2 is slightly different. 

https://deviantengineer.com/2014/08/unifi-controller-centos7/

## Notes:

You will do everything here as root. The UniFi Controller will run as user ubnt, which you create.
Dependencies mentioned here are avaialable under ./resources 


#1. Disable SELinux and update server:

     sed -i /etc/selinux/config -r -e 's/^SELINUX=.*/SELINUX=disabled/g'
     yum -y update
     systemctl reboot
  
It worked for me without disabling SELinux, so you can skip this step, too.

#2. Install EPEL Repo

    yum -y install epel-release
  
#3. Install Prerequisites

	useradd -r ubnt
	yum -y install mongodb-server java-1.8.0-openjdk unzip wget
	
Now, mongo will not start due to a missing dependency libtcmalloc4.so. While it's present in Centos 7, it was not in
my installation of ClearOS. Here's how to fix it:

	wget ftp://rpmfind.net/linux/opensuse/distribution/13.2/repo/oss/suse/x86_64/libtcmalloc4-2.2-3.3.1.x86_64.rpm
	rpm -Uvh libtcmalloc4-2.2-3.3.1.x86_64.rpm
	
	[TODO find proper rpm matching ClearOS]

#4. Download and Extract UniFi Controller

	cd ~ && wget http://dl.ubnt.com/unifi/5.0.7/UniFi.unix.zip
	unzip -q UniFi.unix.zip -d /opt
	chown -R ubnt:ubnt /opt/UniFi
	
#5. Create Startup Script with Systemd
	# vi /etc/systemd/system/unifi.service
	#
	#
	# Systemd unit file for UniFi Controller
	#
	
	[Unit]
	Description=UniFi AP Web Controller
	After=syslog.target network.target
	
	[Service]
	Type=simple
	User=ubnt
	WorkingDirectory=/opt/UniFi
	ExecStart=/usr/bin/java -Xmx1024M -jar /opt/UniFi/lib/ace.jar start
	ExecStop=/usr/bin/java -jar /opt/UniFi/lib/ace.jar stop
	SuccessExitStatus=143
	
	
	[Install]
	WantedBy=multi-user.target
 	
#6. Configure Firewall

In ClearOS UI, open incoming port 8443

#7. Enable UniFi on Startup

	systemctl enable unifi.service
	
#8. Cleanup
	rm -rf ~/UniFi.unix.zip
	systemctl reboot
	
#9. Accessing the UniFi Controller server
	https://<your-server-ip>:8443
	

