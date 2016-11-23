# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

   if Vagrant.has_plugin?("vagrant-cachier")

      # Configure cached packages to be shared between instances of the same base box.
      # More info on http://fgrehm.viewdocs.io/vagrant-cachier/usage
        config.cache.scope = :box

      # OPTIONAL: If you are using VirtualBox, you might want to use that to enable
      # NFS for shared folders. This is also very useful for vagrant-libvirt if you
      # want bi-directional sync
      config.cache.synced_folder_opts = {
        type: :rsync,
        # The nolock option can be useful for an NFSv3 client that wants to avoid the
        # NLM sideband protocol. Without this option, apt-get might hang if it tries
        # to lock files needed for /var/cache/* operations. All of this can be avoided
        # by using NFSv4 everywhere. Please note that the tcp option is not the default.
        mount_options: ['rw', 'vers=3', 'tcp', 'nolock']
      }
      # For more information please check http://docs.vagrantup.com/v2/synced-folders/basic_usage.html
    end


  config.vm.box = "ubuntu/trusty64"
  config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.network "forwarded_port", guest: 5601, host: 5601
  config.vm.network "forwarded_port", guest: 5000, host: 5000
  config.vm.network "forwarded_port", guest: 6000, host: 6000
  config.vm.network "forwarded_port", guest: 6001, host: 6001
  config.vm.network "forwarded_port", guest: 6002, host: 6002
  config.vm.network "forwarded_port", guest: 9200, host: 9200
  config.vm.network "forwarded_port", guest: 4546, host: 4546
  config.vm.network "forwarded_port", guest: 8000, host: 8000
  config.vm.network "forwarded_port", guest: 8090, host: 8090
  config.vm.network "forwarded_port", guest: 9000, host: 9000


  HOSTNAME = 'docker'
  DOMAIN   = 'elkstack.local'
  Vagrant.require_version '>= 1.7.0'
  config.ssh.insert_key = false

config.vm.host_name = HOSTNAME + '.' + DOMAIN

config.vm.provider "virtualbox" do |v|
  v.memory = 3000
  v.cpus = 2
end

config.vm.provider "vmware_fusion" do |v|
  v.vmx["memsize"] = "2024"
  v.vmx["numvcpus"] = "2"
end

$script = <<SCRIPT
sudo add-apt-repository -y ppa:webupd8team/java
sudo apt-get -y update
sudo apt-get -y dist-upgrade
sudo echo "=> Installing docker-compose ..."
sudo rmdir /etc/pki/tls/private
sudo apt-get install -y python-software-properties debconf-utils
echo "oracle-java8-installer shared/accepted-oracle-license-v1-1 select true" | sudo debconf-set-selections
sudo apt-get -y install oracle-java8-installer
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb http://packages.elastic.co/elasticsearch/2.x/debian stable main" | sudo tee /etc/apt/sources.list.d/elasticsearch-2.x.list
echo 'deb http://packages.elastic.co/logstash/2.2/debian stable main' | sudo tee /etc/apt/sources.list.d/logstash-2.2.x.list
sudo apt-get -y update
sudo apt-get -y install elasticsearch
sed -i 's/network.host:/network.host: localhost/g' /etc/elasticsearch/elasticsearch.yml
sudo service elasticsearch restart
sudo update-rc.d elasticsearch defaults 95 10
echo "=> Installing docker-engine ..."
curl -sSL https://get.docker.com/ | sh  > /dev/null 2>&1
echo "=> Configuring vagrant user ..."
sudo groupadd docker
sudo usermod -aG docker vagrant
sudo usermod -aG docker ubuntu
sudo sysctl -w vm.max_map_count=262144
sudo apt-get -y update
echo "=> Installing docker-compose ..."
sudo sh -c "curl -L https://github.com/docker/compose/releases/download/1.7.1/docker-compose-`uname -s`-`uname -m` >/usr/local/bin/docker-compose 2>/dev/null"
sudo chmod +x /usr/local/bin/docker-compose > /dev/null 2>&1
echo "=> Finished installation of Docker"
sudo apt-get -y install openssl
sudo apt-get -y install language-pack-UTF-8
echo "deb http://packages.elastic.co/kibana/4.4/debian stable main" | sudo tee -a /etc/apt/sources.list.d/kibana-4.4.x.list
sudo apt-get -y update
sudo apt-get -y install kibana
sudo update-rc.d kibana defaults
sed -i 's/server.host:/server.host: localhost/g' /opt/kibana/config/kibana.yml
sudo update-rc.d kibana defaults 96 9
sudo service kibana start
sudo update-rc.d kibana defaults
sudo apt-get install -y nginx apache2-utils
echo "kibanaadmin" |sudo htpasswd -i -c /etc/nginx/htpasswd.users kibanaadmin
sudo apt-get install -y logstash
sudo cp /vagrant/02-beats-input.conf /etc/logstash/conf.d/02-beats-input.conf
sudo cp /vagrant/10-syslog-filter.conf /etc/logstash/conf.d/10-syslog-filter.conf
sudo service logstash start
sudo update-rc.d logstash defaults
sudo mkdir -p /etc/pki/tls/certs
sudo mkdir /etc/pki/tls/private
sudo chmod 666 /etc/nginx/sites-available/*
sudo chown -R www-data:www-data /var/log/nginx
sudo cp /vagrant/certs/logstash-forwarder.crt /etc/pki/tls/certs/logstash-forwarder.crt
sudo cp /vagrant/certs/logstash-forwarder.key /etc/pki/tls/private/logstash-forwarder.key

echo "Installing kibana dashboards and beats index ...."
cd ~ ; curl -L -O https://download.elastic.co/beats/dashboards/beats-dashboards-1.1.0.zip
sudo apt-get -y install iptraf htop unzip
unzip beats-dashboards-*.zip
cd beats-dashboards-*
./load.sh

cd ~
curl -O https://gist.githubusercontent.com/thisismitch/3429023e8438cc25b86c/raw/d8c479e2a1adcea8b1fe86570e42abab0f10f364/filebeat-index-template.json


cd ~
curl -XPUT 'http://localhost:9200/_template/filebeat?pretty' -d@filebeat-index-template.json


SCRIPT

config.vm.provision "shell" , inline: "sudo sysctl -w vm.max_map_count=262144", run: "always"
config.vm.provision "shell", inline: $script
config.vm.provision "file", source: "nginx/default", destination: "/etc/nginx/sites-available/default"
config.vm.provision "shell", inline: "sudo service nginx restart"
end



