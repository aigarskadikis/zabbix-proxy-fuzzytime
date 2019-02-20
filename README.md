# Auto fuzzytime trigger for Zabbix Proxy

This template generates a fuzzytime trigger for all proxies in your instance. The template must be assigned to host "Zabbix server".

Solution is based on Zabbix API. In order to work with API you must setup credentials at first.

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
# check what is a default location home location for user 'zabbix'
grep zabbix /etc/passwd

# try to navigate to the dir. it probably do not exist
cd /var/lib/zabbix

# if directory do not exist then create it
mkdir -p /var/lib/zabbix

# owner for the dir is user 'zabbix'
chown -R zabbix. /var/lib/zabbix

# enter bash environment under 'zabbix' environment 
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

Attach the template to "Zabbix server". The discoery by executes in 1h interval.