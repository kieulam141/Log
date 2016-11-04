#/bin/bash
#############################################################
# Graylog V2.1
# Ubuntu 16.04.1
# Created: 1/11/2016
# Author: Lamkt
#############################################################
# Run with root account  
# wget https://raw.githubusercontent.com/hocchudong/log-script/master/graylog/graylog-server.sh
# bash graylog-server.sh
# Wait ....join

###############################

echo -e "\033[33m  ##### Script install Graylog V2.1 (Script cai dat Graylog V2.1) ###### \033[0m"
sleep 3

###############################
echo -e "\033[33m ##### install additional packages:
sleep 3
apt-get install apt-transport-https openjdk-8-jre-headless uuid-runtime pwgen
echo -e "\033[33m ##### Update repos packages Elasticsearch, MongoDB, Graylog \033[0m"
sleep 3
# Install Elasticsearch
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://packages.elastic.co/elasticsearch/2.x/debian stable main" | sudo tee -a /etc/apt/sources.list.d/elasticsearch-2.x.list
sudo apt-get update && sudo apt-get install elasticsearch
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
sudo /bin/systemctl restart elasticsearch.service

# Install MongoDB
apt-get install mongodb-server

# Install Graylog
wget https://packages.graylog2.org/repo/packages/graylog-2.1-repository_latest.deb
sudo dpkg -i graylog-2.1-repository_latest.deb
sudo apt-get update && sudo apt-get install graylog-server

# Update
apt-get update
