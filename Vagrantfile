# -*- mode: ruby -*-
# vi: set ft=ruby :

$common = <<SCRIPT
sudo apt-get update
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
sudo apt-get install -y docker.io
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
sudo kind create cluster --config=/vagrant/task/clusterconfig.yaml

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh

helm repo add podinfo https://stefanprodan.github.io/podinfo

sudo kubectl create namespace test

sudo helm upgrade --install --wait frontend \
--namespace test \
--set replicaCount=2 \
--set backend=http://backend-podinfo:9898/echo \
podinfo/podinfo

sudo helm test frontend --namespace test

sudo helm upgrade --install --wait backend \
--namespace test \
--set redis.enabled=true \
podinfo/podinfo

sudo kubectl apply -f /vagrant/task/svc-nodeport.yaml
sudo apt install nginx -y

curl localhost:8080
sudo echo "127.0.0.1 demo.code-intelligence.local" >> /etc/hosts
curl demo.code-intelligence.local:8080

sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt -subj "/C=BG/ST=Sofia/L=Sofia/O=Stanislav/OU=IT/CN=demo.code-intelligence.local"

sudo rm /etc/nginx/sites-enabled/default
sudo cp /vagrant/task/demo /etc/nginx/sites-enabled/demo
sudo systemctl restart nginx

curl -L --insecure http://demo.code-intelligence.local
echo "___________________________________________"
curl --insecure https://demo.code-intelligence.local
SCRIPT

Vagrant.configure(2) do |config|
    
  config.ssh.insert_key = false

  config.vm.provider "virtualbox" do |v|
    v.memory = 2048
    v.cpus = 2
  end

  config.vm.define "node1" do |node1|
    node1.vm.box = "bento/ubuntu-23.04"
    node1.vm.hostname = "node1.k8s.lab"
    node1.vm.network "private_network", ip: "192.168.99.101"
    node1.vm.synced_folder "vagrant/", "/vagrant"
    node1.vm.provision "shell", inline: $common
  end
end
