# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME']
ENV["LC_ALL"] = "en_US.UTF-8"

MACHINES = {
  :systemd => {
        :box_name => "centos/7",
        :box_version => "1804.02",
        :ip_addr => '192.168.11.102',
    :disks => {
    }
  },
}

Vagrant.configure("2") do |config|

    config.vm.box_version = "1804.02"
    MACHINES.each do |boxname, boxconfig|
  
        config.vm.define boxname do |box|
  
            box.vm.box = boxconfig[:box_name]
            box.vm.host_name = boxname.to_s
  
            #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset
  
            box.vm.network "private_network", ip: boxconfig[:ip_addr]
  
            box.vm.provider :virtualbox do |vb|
                    vb.customize ["modifyvm", :id, "--memory", "256"]
                    needsController = false
				boxconfig[:disks].each do |dname, dconf|
					unless File.exist?(dconf[:dfile])
					vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
									needsController =  true
					end
				end
				if needsController == true
					vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
					boxconfig[:disks].each do |dname, dconf|
						vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
					end
				end
            end
        	# hw 3.1
        box.vm.provision "shell", inline: <<-'SHELL'
          mkdir -p ~root/.ssh
          cp ~vagrant/.ssh/auth* ~root/.ssh
          # 8.1
          cat > /opt/logwatcher.sh <<-'EOF'
#!/bin/bash
WORD=$1
LOG=$2
DATE=`date`

if grep $WORD $LOG &>/dev/null
then
logger "$DATE: found $WORD in $LOG"
else
exit 0
fi
EOF
          chmod +x /opt/logwatcher.sh
          cat > /etc/systemd/system/logwatcher.service <<-'EOF'
[Unit]
Description=My logwatcher service
[Service]
Type=oneshot
EnvironmentFile=/etc/sysconfig/logwatcher
ExecStart=/opt/logwatcher.sh $WORD $LOG
EOF
          cat > /etc/systemd/system/logwatcher.timer <<-'EOF'
[Unit]
Description=Run logwatcher every 30 seconds
[Timer]
OnUnitActiveSec=30
Unit=logwatcher.service
[Install]
WantedBy=multi-user.target
EOF
          cat > /etc/sysconfig/logwatcher <<-'EOF'
# Configuration file for my logwatcher service
# Place it to /etc/sysconfig

# File and word in that file that we will be monit
WORD="ALERT"
LOG=/var/log/watchlog.log
EOF
          systemctl start logwatcher.timer
          echo -e "asdf\nadfasdf\nqerqwergafg\nqerqerqr\nqwerqwerALERT\nqerqwrea\123afa\n\n" > /var/log/watchlog.log
          #8.2
          yum install epel-release -y
          yum install -y spawn-fcgi php php-cli mod_fcgid httpd -y
          sed -E -i 's/#(SOCKET|OPTIONS)/\1/' /etc/sysconfig/spawn-fcgi
          sed -E -i 's/\s-P\s\/var\/run\/spawn-fcgi.pid//' /etc/sysconfig/spawn-fcgi
          cat > /etc/systemd/system/spawn-fcgi.service <<-'EOF'
[Unit]
Description=Spawn-fcgi startup service by Otus
After=network.target

[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/sysconfig/spawn-fcgi
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process

[Install]
WantedBy=multi-user.target
EOF
systemctl start spawn-fcgi
systemctl status spawn-fcgi
          #8.3
          cp /usr/lib/systemd/system/httpd.service /usr/lib/systemd/system/httpd@.service
          sed -i -E 's/(sysconfig\/httpd)/\1-%I/' /usr/lib/systemd/system/httpd@.service
          for i in first second; \
            do 
              cp /etc/sysconfig/httpd /etc/sysconfig/httpd-$i; \
              cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/$i.conf
              sed -i "s/#OPTIONS=/OPTIONS=-f conf\/$i.conf/" /etc/sysconfig/httpd-$i;\
          done;
          sed -i -E 's/Listen 80/Listen 8080/' /etc/httpd/conf/second.conf
          echo -e "PidFile\t/var/run/httpd-second.pid" >> /etc/httpd/conf/second.conf
          for f in first second; \
            do
              systemctl start httpd@$f;
            done;
          ss -tnulp | grep httpd
          SHELL
    	end
  	end
end