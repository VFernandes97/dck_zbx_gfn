# dck_zbx_gfn
#Repósitorio com o objetivo de subir de forma simples as aplicações zabbix e grafana em docker.


#O primeiro passo é ter o docker instalado.

#Segue abaixo lista de comandos:

#OBS: Alguns containers demoram um pouco para ficarem prontos para uso, deixarei anotado nesse documento quais.

#1 - Depois de rodar o comando aguardar cerca de 5 minutos.

sudo docker run --name mysql-server -t -e MYSQL_DATABASE="zabbix" -e MYSQL_USER="zabbix" -e MYSQL_PASSWORD="pr0dpr0d" -e MYSQL_ROOT_PASSWORD="pr0dpr0d" -d --volume mysql-server:/var/lib/mysql mysql --character-set-server=utf8 --collation-server=utf8_bin --default-authentication-plugin=mysql_native_password

#2

sudo docker run --name zabbix-java-gateway -t --restart unless-stopped -d --volume zabbix-java:/var/lib/zabbix zabbix/zabbix-java-gateway:ubuntu-latest

#3 - Depois de rodar o comando aguardar cerca de 5 minutos.

#OBS: Esse container estou iniciando algumas váriaveis para melhorar o desempenho da aplicação( é opcional ).

sudo docker run --name zabbix-server-mysql -t -e DB_SERVER_HOST="mysql-server" -e MYSQL_DATABASE="zabbix" -e MYSQL_USER="zabbix" -e MYSQL_PASSWORD="pr0dpr0d" -e MYSQL_ROOT_PASSWORD="pr0dpr0d" -e ZBX_JAVAGATEWAY="zabbix-java-gateway" -e ZBX_STARTPOLLERS=20 -e ZBX_STARTPOLLERSUNREACHABLE=10 -e ZBX_STARTPINGERS=10 -e ZBX_STARTDISCOVERERS=10 --link mysql-server:mysql --link zabbix-java-gateway:zabbix-java-gateway -p 10051:10051 --restart unless-stopped -d --volume zabbix-server:/var/lib/zabbix --volume zabbix-snmp:/var/lib/zabbix/snmptraps --volume zabbix-export:/var/lib/zabbix/export zabbix/zabbix-server-mysql:ubuntu-latest

#4

sudo docker run --name zabbix-web-nginx-mysql -t -e DB_SERVER_HOST="mysql-server" -e MYSQL_DATABASE="zabbix" -e MYSQL_USER="zabbix" -e MYSQL_PASSWORD="pr0dpr0d" -e MYSQL_ROOT_PASSWORD="pr0dpr0d" -e ZBX_SERVER_HOST="zabbix-server-mysql" --link mysql-server:mysql --link zabbix-server-mysql:zabbix-server-mysql -p 80:8080 --restart unless-stopped -d --volume zabbix-web-nginx-mysql:/usr/share/zabbix zabbix/zabbix-web-nginx-mysql:ubuntu-latest

#5

sudo docker run --name zabbix-agent --link mysql-server:mysql --link zabbix-server-mysql:zabbix-server -e ZBX_HOSTNAME="Zabbix server" -e ZBX_SERVER_HOST="zabbix-server" -d --volume zabbix-agent:/var/lib/zabbix zabbix/zabbix-agent:ubuntu-latest

#6

sudo docker volume create grafana-storage

#7

sudo docker run -d -p 3000:3000 --name=grafana -e "GF_INSTALL_PLUGINS=alexanderzobnin-zabbix-app" -e "GF_PLUGINS_ALLOW_LOADING_UNSIGNED_PLUGINS=alexanderzobnin-zabbix-datasource" --restart unless-stopped -v grafana-storage:/var/lib/grafana grafana/grafana:latest-ubuntu

#OBS: Após a aplicação Zabbix estiver no ar precisa mudar o host Zabbix Server para o container do zabbix-agent.

#Teste sua aplicação: seuip/zabbix
#EX: 192.168.1.50/zabbix
