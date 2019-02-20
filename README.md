# Auto fuzzytime trigger for Zabbix Proxy

![com-property-url](https://raw.githubusercontent.com/catonrug/zabbix-proxy-fuzzytime/master/png/fuzzytime-trigger-prototype-zabbix-proxy.png)

This template generates a fuzzytime trigger for all proxies in your instance. The template must be assigned to host "Zabbix server".

Solution is based on Zabbix API and checking on Zabbix internal items. In order to work with API you must install Python PyZabbix module, setup credentials, and create an external script.

## Prepare PyZabbix module
```
# on CentOS/RHEL family
yum -y install epel-release
yum -y install python-pip

# on Debian/Ubuntu family
apt -y install python-pip

# install module
pip -U pyzabbix
```

## Setup credentials for Zabbix Python API
```
# check what is a default 'home' location for user 'zabbix'
grep zabbix /etc/passwd

# try to navigate to the dir. it probably do not exist
cd /var/lib/zabbix

# if directory do not exist then create it
mkdir -p /var/lib/zabbix

# owner for the dir must be user 'zabbix'
chown -R zabbix. /var/lib/zabbix

# enter bash environment under user 'zabbix' 
su - zabbix -s /bin/bash

# make sure you are at home dir
cd && pwd
# this should report '/var/lib/zabbix'

# if your server is located on http://127.0.0.1/zabbix/ (default) use
cat <<'EOF'> config.py
url = "http://127.0.0.1/zabbix"
username = "Admin"
password = "zabbix"
EOF

# if your server is located on http://127.0.0.1/ (web root) use
cat <<'EOF'> config.py
url = "http://127.0.0.1/"
username = "Admin"
password = "zabbix"
EOF

# check again the config one last time
cat config.py

# quit zabbix user
exit
```

## Install externalscript

The following script will use Zabbix API to list all Proxy servers in your instance and convert the output to JSON format which is suitable for low-level discovery.

```
# move to externalscripts direcotry
cd /usr/lib/zabbix/externalscripts/

# install script
curl https://raw.githubusercontent.com/catonrug/externalscripts/master/list-all-proxies-json.py > list-all-proxies-json.py
chown zabbix. list-all-proxies-json.py
chmod +x list-all-proxies-json.py

# test script
./list-all-proxies-json.py
# it must insatantly show the list of all proxies
```

## Install template
Upload [auto-fuzzytime-trigger-zabbix-proxy.xml](https://raw.githubusercontent.com/catonrug/zabbix-proxy-fuzzytime/master/auto-fuzzytime-trigger-zabbix-proxy.xml) to you Zabbix instance

Link the template to "Zabbix server". The discovery by default executes every 1h.