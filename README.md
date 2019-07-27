# Dockerfile -- one directory up of course alongside with "data" folder which is synced.

```
# -_- mode: ruby -_-

# vi: set ft=ruby :

Vagrant.configure("2") do |config|
config.vm.box = "centos/7"

config.vm.network "forwarded_port", guest: 80, host: 8080

config.vm.network "private_network", ip: "192.168.33.88"

config.vm.synced_folder "data", "/vagrant_data"

config.vm.provision "shell", inline: <<-SHELL
sudo apt-get update
sudo apt-get install -y software-properties-common curl
sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
sudo echo 'deb https://apt.dockerproject.org/repo ubuntu-trusty main' > /etc/apt/sources.list.d/docker.list
sudo apt-get update
sudo apt-get purge lxc-docker
sudo apt-cache policy docker-engine
sudo apt-get install -y linux-image-extra-\$(uname -r)
sudo apt-get install -y docker-engine
sudo echo 'DOCKER_OPTS="-H tcp://127.0.0.1:4243 -H unix:///var/run/docker.sock"' >> /etc/default/docker
sudo service docker restart
sudo usermod -aG docker vagrant
echo 'export DOCKER_HOST=tcp://localhost:4243' >> /home/vagrant/.bashrc
curl -L https://github.com/docker/compose/releases/download/1.6.2/docker-compose-`uname -s`-`uname -m` -o docker-compose
sudo chmod +x docker-compose
sudo mv docker-compose /usr/local/bin
SHELL
end
```

## Real workflow with commands

### all code is in /Volumes/GO/VAGRANT/Centos7 -- Dockerfile and shared folder "data" in which folder "app" is where all app files are.

```
cd /Volumes/GO/VAGRANT/Centos7
vagrant up
vagrant ssh
cd /vagrant_data/app/
sudo docker pull redis:4-alpine
sudo docker image build -t web1 .
sudo docker image ls
sudo docker container ls
sudo docker container ls -a
sudo docker network ls
sudo docker network inspect bridge
sudo docker network create --driver bridge firstnetwork
sudo docker network inspect firstnetwork
sudo docker container run --rm -itd -p 6379:6379 --name redis --net firstnetwork redis:4-alpine
sudo docker container run --rm -itd -p 5000:5000 --name web1 -e FLASK_APP=app.py -e FLASK_DEBUG=1 -v $PWD:/app --net firstnetwork web1
sudo docker network inspect firstnetwork
http://192.168.33.88:5000/
```

### Above code is without data persitence for redis server so counts will be reset to 1

### Happy coding Gray Monk
