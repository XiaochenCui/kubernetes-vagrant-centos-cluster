# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box_check_update = false
  config.vm.provider 'virtualbox' do |vb|
   vb.customize [ "guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold", 1000 ]
  config.vm.synced_folder ".", "/vagrant", type: "nfs", nfs_udp: false
  end

  $num_instances = 3
  # curl https://discovery.etcd.io/new?size=3
  $etcd_cluster = "node1=http://172.17.8.101:2380"
  (1..$num_instances).each do |i|
    config.vm.define "node#{i}" do |node|
      node.vm.box = "bento/centos-7.5"
      node.vm.hostname = "node#{i}"
      ip = "172.17.8.#{i+100}"
      node.vm.network "private_network", ip: ip
      node.vm.provider "virtualbox" do |vb|
        vb.memory = "3072"
        vb.cpus = 1
        vb.name = "node#{i}"
      end
      node.vm.provision "shell", path: "install.sh", args: [i, ip, $etcd_cluster]
    end
  end

  config.ssh.username = "root"
  config.ssh.password = "vagrant"
  config.ssh.insert_key = "true"

  if Vagrant.has_plugin?("vagrant-proxyconf")
    config.proxy.http     = "http://10.0.2.2:1087/"
    config.proxy.https    = "http://10.0.2.2:1087/"
    config.proxy.no_proxy = "localhost,127.0.0.1,.example.com"
  end

  config.vm.provision "bootstrap", type: "shell",
    inline: $bootstrap

  config.vm.provision "toolkit", type: "shell",
    inline: $toolkit

end

$bootstrap = <<SCRIPT

yum update
yum install -y epel-release
yum upgrade -y
yum update -y
yum install -y --nogpgcheck git zsh hstr

SCRIPT

$dns = <<SCRIPT

echo "nameserver 1.1.1.1" > /etc/resolv.conf

SCRIPT

$toolkit = <<SCRIPT

cd ~ && rm -rf xiaochen-toolkit ; git clone --depth 1 https://github.com/XiaochenCui/xiaochen-toolkit.git && cd xiaochen-toolkit && ./setup/minimal.sh

SCRIPT

$go = <<SCRIPT

wget -q https://dl.google.com/go/go1.12.2.linux-amd64.tar.gz
tar -xzf go1.12.2.linux-amd64.tar.gz
rm -r /usr/local/go
mv go /usr/local

echo "" > ~/.localrc
echo "\nexport GOROOT=/usr/local/go" >> ~/.localrc
echo "export PATH=$GOROOT/bin:$PATH" >> ~/.localrc

SCRIPT
