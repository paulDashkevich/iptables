# -*- mode: ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
:inetRouter => {
        :box_name => "centos/7",
        #:public => {:ip => '10.10.10.1', :adapter => 1},
        :net => [
                   {ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "router-net"},
                ]
  },

:inetRouter2 => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.255.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "router-net"},
	           {ip: '192.168.0.1',   adapter: 3, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"}, 
               ]
  },
  
:centralServer => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                ]
  },
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|
        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s
        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ipconf
        end
        if boxconfig.key?(:public)
          box.vm.network "public_network", boxconfig[:public]
        end
        box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh
          cp ~vagrant/.ssh/auth* ~root/.ssh
        SHELL
        
        case boxname.to_s
        when "inetRouter"
        box.vm.network 'forwarded_port', guest: 8080, host: 8080, host_ip: '127.0.0.1'  
	box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf
	    echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
	    sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
	    echo "192.168.0.0/16 via 192.168.255.2 dev eth1" > /etc/sysconfig/network-scripts/route-eth1
            sudo yum install -y iptables-services; sudo systemctl enable iptables && sudo systemctl start iptables
            sudo iptables -F; 
	    sudo iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
	    sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
            sudo iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT
	    sudo iptables-restore < /vagrant/iptables.rules
	    sudo service iptables save
	    sudo reboot
	    SHELL
        when "inetRouter2"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf
            echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
	    echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
	    echo "192.168.0.0/16 via 192.168.255.3 dev eth1" > /etc/sysconfig/network-scripts/route-eth1
            sudo yum install -y iptables-services; sudo systemctl enable iptables && sudo systemctl start iptables
            sudo iptables -F; 
	    sudo iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
	    sudo iptables -t nat -A PREROUTING -d 192.168.255.2 -p tcp -m tcp --dport 8080 -j DNAT --to-destination 192.168.0.2:80
	    sudo iptables -t nat -A POSTROUTING -d 192.168.0.2 -p tcp -m tcp --dport 80 -j SNAT --to-source 192.168.255.2
	    sudo iptables -t nat -A OUTPUT -d 192.168.255.2 -p tcp -m tcp --dport 80 -j DNAT --to-destination 192.168.0.2
	    sudo iptables -I FORWARD 1 -i eth1 -o eth2 -d 192.168.0.2 -p tcp -m tcp --dport 80 -j ACCEPT
	    sudo service iptables save
            sudo reboot
            SHELL
        when "centralServer"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
	    sudo sleep 40
	    sudo yum install epel-release -y
	    sudo yum install nginx -y
	    sudo yum install nmap -y
	    sudo systemctl start nginx
	    sudo systemctl enable nginx
	    sudo touch /home/vagrant/knock.sh
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            sudo reboot
            SHELL
        end
      end
  end
end
