# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  # centos6 (32-bit) client
  config.vm.define "centos6" do |centos6|
    centos6.vm.box = "puppetlabs/centos-6.6-32-nocm"
    centos6.vm.box_url = 'puppetlabs/centos-6.6-32-nocm'
    centos6.vm.network "private_network", ip: "192.168.0.10"
    centos6.vm.provider "virtualbox" do |v|
      v.memory = 128
      v.cpus = 1
    end
  end
  # centos7 client
  config.vm.define "centos7" do |centos7|
    centos7.vm.box = "puppetlabs/centos-7.0-64-nocm"
    centos7.vm.box_url = 'puppetlabs/centos-7.0-64-nocm'
    centos7.vm.network "private_network", ip: "192.168.0.20"
    centos7.vm.provider "virtualbox" do |v|
      v.memory = 128
      v.cpus = 1
    end
  end
  # ubuntu client
  config.vm.define "ubuntu14" do |ubuntu14|
    ubuntu14.vm.box = "puppetlabs/ubuntu-14.04-64-nocm"
    ubuntu14.vm.box_url = 'puppetlabs/ubuntu-14.04-64-nocm'
    ubuntu14.vm.network "private_network", ip: "192.168.0.30"
    ubuntu14.vm.synced_folder ".", "/vagrant", disabled: true
    ubuntu14.vm.provider "virtualbox" do |v|
      v.memory = 128
      v.cpus = 1
    end
  end
  # muninserver
  config.vm.define "muninserver" do |muninserver|
    muninserver.vm.box = "puppetlabs/centos-6.6-64-nocm"
    muninserver.vm.box_url = 'puppetlabs/centos-6.6-64-nocm'
    muninserver.vm.network "private_network", ip: "192.168.0.5"
    muninserver.vm.network "forwarded_port", guest: 80, host: 8080
    muninserver.vm.provider "virtualbox" do |v|
      v.memory = 128
      v.cpus = 1
    end
  end
  # stop iptables
  $service_iptables_stop = <<SCRIPT
service iptables stop
SCRIPT
  # stop firewalld
  $systemctl_stop_firewalld = <<SCRIPT
systemctl stop firewalld.service
SCRIPT
  # common settings on all machines
  $etc_hosts = <<SCRIPT
cat <<END >> /etc/hosts
192.168.0.5 muninserver
192.168.0.10 centos6
192.168.0.20 centos7
192.168.0.30 ubuntu14
END
SCRIPT
  $epel6 = <<SCRIPT
yum -y install http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
SCRIPT
  $epel7 = <<SCRIPT
yum -y install http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-9.noarch.rpm
SCRIPT
  $apache_status_conf = <<SCRIPT
cat <<'END' > /etc/httpd/conf.d/status.conf
 <Location /server-status>
    SetHandler server-status
    Order deny,allow
    Deny from all
    Allow from 127.0.0.1
 </Location>
END
SCRIPT
  # the actual provisions of machines
  config.vm.define "centos6" do |centos6|
    centos6.vm.provision :shell, :inline => "hostname centos6", run: "always"
    centos6.vm.provision :shell, :inline => $etc_hosts
    centos6.vm.provision :shell, :inline => $epel6
    centos6.vm.provision :shell, :inline => $service_iptables_stop, run: "always"
    centos6.vm.provision :shell, :inline => "yum install -y munin-node"
    centos6.vm.provision :shell, :inline => "sed -i 's/host_name .*/host_name centos6/' /etc/munin/munin-node.conf"
    centos6.vm.provision :shell, :inline => "sed -i 's/allow ^127.*/allow ^192\\.168\\.0\\.5$/' /etc/munin/munin-node.conf"
    centos6.vm.provision :shell, :inline => "yum install -y httpd"
    centos6.vm.provision :shell, :inline => $apache_status_conf
    centos6.vm.provision :shell, :inline => "service munin-node start"
    centos6.vm.provision :shell, :inline => "chkconfig munin-node on"
  end
  config.vm.define "centos7" do |centos7|
    centos7.vm.provision :shell, :inline => "hostname centos7", run: "always"
    centos7.vm.provision :shell, :inline => $etc_hosts
    centos7.vm.provision :shell, :inline => $epel7
    centos7.vm.provision :shell, :inline => $systemctl_stop_firewalld, run: "always"
    centos7.vm.provision :shell, :inline => "yum install -y munin-node"
    centos7.vm.provision :shell, :inline => "sed -i 's/host_name .*/host_name centos7/' /etc/munin/munin-node.conf"
    centos7.vm.provision :shell, :inline => "sed -i 's/allow ^127.*/allow ^192\\.168\\.0\\.5$/' /etc/munin/munin-node.conf"
    centos7.vm.provision :shell, :inline => "service munin-node start"
    centos7.vm.provision :shell, :inline => "chkconfig munin-node on"
  end
  config.vm.define "ubuntu14" do |ubuntu14|
    ubuntu14.vm.provision :shell, :inline => "hostname ubuntu14", run: "always"
    ubuntu14.vm.provision :shell, :inline => $etc_hosts
    ubuntu14.vm.provision :shell, :inline => "DEBIAN_FRONTEND=noninteractive apt-get install -y munin-node"
    ubuntu14.vm.provision :shell, :inline => "sed -i 's/.*host_name .*/host_name ubuntu14/' /etc/munin/munin-node.conf"
    ubuntu14.vm.provision :shell, :inline => "sed -i 's/allow ^127.*/allow ^192\\.168\\.0\\.5$/' /etc/munin/munin-node.conf"
    ubuntu14.vm.provision :shell, :inline => "update-rc.d munin-node defaults"
  end
  # last provision muninserver
  config.vm.define "muninserver" do |muninserver|
    muninserver.vm.provision :shell, :inline => "hostname muninserver", run: "always"
    muninserver.vm.provision :shell, :inline => $etc_hosts
    muninserver.vm.provision :shell, :inline => $epel6
    muninserver.vm.provision :shell, :inline => $service_iptables_stop, run: "always"
    muninserver.vm.provision :shell, :inline => "yum install -y httpd"
    muninserver.vm.provision :shell, :inline => "yum install -y munin munin-node"
    muninserver.vm.provision :shell, :inline => "htpasswd -b -c /etc/munin/munin-htpasswd munin munin"
    muninserver.vm.provision :shell, :inline => "chown apache.apache /etc/munin/munin-htpasswd"
    muninserver.vm.provision :shell, :inline => "chmod 700 /etc/munin/munin-htpasswd"
    muninserver.vm.provision :shell, :inline => "echo -e '[centos6]\naddress 192.168.0.10\nuse_node_name yes\n' >> /etc/munin/munin.conf"
    muninserver.vm.provision :shell, :inline => "echo -e '[centos7]\naddress 192.168.0.20\nuse_node_name yes\n' >> /etc/munin/munin.conf"
    muninserver.vm.provision :shell, :inline => "echo -e '[ubuntu14]\naddress 192.168.0.30\nuse_node_name yes\n' >> /etc/munin/munin.conf"
    muninserver.vm.provision :shell, :inline => "service munin-node start"
    muninserver.vm.provision :shell, :inline => "chkconfig munin-node on"
    muninserver.vm.provision :shell, :inline => "service httpd start"
    muninserver.vm.provision :shell, :inline => "chkconfig httpd on"
  end
end
