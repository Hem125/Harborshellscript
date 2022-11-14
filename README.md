# Harborshellscript
# chmod o+x /root/
# IP Address = 192.168.65.131
# Harbor Address= harborscript.com

#!/bin/bash


# lets install docker first
echo "lets install docker "

sudo apt install docker.io

echo "docker install"

# now lets install docker-compose

echo "lets install docker compose"

sudo curl -L https://github.com/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

docker-compose --version

echo "docker-compose install"

##
echo "enter your system IPADDRESS"

read HARBOR_IP

# export HARBOR_IP=$HARBOR_IP
echo "config daemon.json file"

cd /etc/docker/

touch daemon.json

tee /etc/docker/daemon.json >/dev/null <<EOF
                                             
{
                                             
    "insecure-registries" : ["$HARBOR_IP"]
}
EOF
systemctl restart docker.service
##
# lets work on harbor part
echo "harbor part"
sudo su
cd /root
echo "change directory to root"
openssl rand -writerand .rnd
cd /home
echo "now create a folder harbor"
mkdir harbor
wget https://github.com/goharbor/harbor/releases/download/v2.0.5/harbor-online-installer-v2.0.5.tgz
tar xvf harbor-online-installer-v2.0.5.tgz
cd harbor
cp harbor.yml.tmpl harbor.yml
export HARBOR_ROOT=/home/harbor
export HARBOR_DATA_VOLUME=/root/harbor/data_volume
echo "enter HARBOR_IP"
# echo "enter your system IPADDRESS"
read HARBOR_IP
# export HARBOR_IP=$HARBOR_IP
####
echo "enter a password for harbor"
read HARBOR_ADMIN_PASSWORD
# export HARBOR_ADMIN_PASSWORD=$HARBOR_ADMIN_PASSWORD
cd $HARBOR_ROOT
mkdir cert
sed -i "s/hostname: .*/hostname: $HARBOR_IP/g" $HARBOR_ROOT/harbor.yml
sed -i "s#data_volume: .*#data_volume: $HARBOR_DATA_VOLUME#g" $HARBOR_ROOT/harbor.yml
sed -i "s/harbor_admin_password: .*/harbor_admin_password: $HARBOR_ADMIN_PASSWORD/g" $HARBOR_ROOT/harbor.yml

cd $HARBOR_ROOT
./install.sh --with-trivy
