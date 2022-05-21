# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.memory = "1024"
    vb.cpus = 1
  end

  config.vm.box = "ubuntu/focal64"

  (1..3).each do |i|
    config.vm.define "etcd#{i}" do |etcd|
      
      etcd.vm.hostname = "etcd#{i}"
      etcd.vm.network :private_network, ip: "10.100.0.1#{i}"
      
      etcd.vm.synced_folder "./etcd#{i}", "/vagrant_data"
      etcd.vm.provision "shell", inline: <<-SHELL
          # ETCD install and configuration
          apt-get update
          apt-get install -y etcd
          rm /etc/default/etcd
          cp /vagrant_data/etcd.txt /etc/default/etcd
          systemctl restart etcd
          #etcdctl member list
          if [ #{i} -eq 1 ]; then
            etcdctl member add etcd#{i+1} http://10.100.0.1#{i+1}:2380
          fi
          if [ #{i} -eq 2 ]; then
            sleep 30
            etcdctl member add etcd#{i+1} http://10.100.0.1#{i+1}:2380
          fi
      SHELL
    end
  end

  (1..3).each do |i|
    config.vm.define "pgsql#{i}" do |etcd|
      etcd.vm.hostname = "pgsql#{i}"
      etcd.vm.network :private_network, ip: "10.100.0.2#{i}"
      
      etcd.vm.synced_folder "./pgsql#{i}", "/vagrant_data"
      etcd.vm.provision "shell", inline: <<-SHELL
        # Percona Postgresql installation
        wget https://repo.percona.com/apt/percona-release_latest.generic_all.deb
        dpkg -i percona-release_latest.generic_all.deb
        apt-get update
        percona-release setup ppg-12
        apt-get install -y percona-postgresql-12
        systemctl stop postgresql
        systemctl disable postgresql
        rm -rf /var/lib/postgresql/12/main
        # Patroni installation
        echo "10.100.0.21 pgsql1 pgsql1" >> /etc/hosts
        echo "10.100.0.22 pgsql2 pgsql2" >> /etc/hosts
        echo "10.100.0.23 pgsql3 pgsql3" >> /etc/hosts
        mkdir /etc/patroni
        cp /vagrant_data/patroni.yml /etc/patroni/
        apt-get install -y percona-patroni
        systemctl restart patroni
      SHELL
    end
  end
end
