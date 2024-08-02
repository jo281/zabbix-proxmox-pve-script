# zabbix-proxmox-pve-script
Когда мониторим Proxmox PVE, то нужно сконфигурировать сам проксмокс. По мотивам статей ниже

https://itproblog.ru/%D0%BC%D0%BE%D0%BD%D0%B8%D1%82%D0%BE%D1%80%D0%B8%D0%BD%D0%B3-proxmox-%D1%87%D0%B5%D1%80%D0%B5%D0%B7-zabbix/

https://megapuper.ru/index.php?title=PROXMOX._Zabbix-%D0%BC%D0%BE%D0%BD%D0%B8%D1%82%D0%BE%D1%80%D0%B8%D0%BD%D0%B3

[zabbix-proxmox-pve-script config from pveum](https://www.zabbix.com/ru/integrations/proxmox)

Выполняем на узле проксмокс
```
#Создаём роль
pveum role add Zabbix --privs "Sys.Audit VM.Audit Datastore.Audit"
#Добавляем группу
pveum group add zabbix
#Выдаём права группе
pveum acl modify / -group zabbix -role Zabbix
#Создаём пользователя
pveum user add zabbix@pam
#Добавляем пользователя в группу
pveum user modify zabbix@pam -group zabbix
#Получаем api-токен
pveum user token add zabbix@pam monitoring -privsep 0
```
Сразу, чтобы не забыть, скрипт установки заббикс агента на дебиан 12
```
apt install wget -y
wget https://repo.zabbix.com/zabbix/7.0/debian/pool/main/z/zabbix-release/zabbix-release_7.0-2+debian12_all.deb
dpkg -i zabbix-release_7.0-2+debian12_all.deb
apt update -y
#Install Zabbix agent
apt install zabbix-agent -y
#Start Zabbix agent process
#Start Zabbix agent process and make it start at system boot.
mkdir -p /etc/zabbix/zabbix_agentd.d
sed -i 's/^Server=127.0.0.1/Server=zabbix.local/g' /etc/zabbix/zabbix_agentd.conf
systemctl restart zabbix-agent
systemctl enable zabbix-agent
systemctl status zabbix-agent
zabbix_agentd -V|grep daemon

grep ^Server /etc/zabbix/zabbix_agentd.conf

iptables -A INPUT -p tcp -m tcp --dport 10050 -j ACCEPT
iptables-save
```
CentOS 7
```
sed -i s/mirror.centos.org/vault.centos.org/g /etc/yum.repos.d/*.repo
sed -i s/^#.*baseurl=http/baseurl=http/g /etc/yum.repos.d/*.repo
sed -i s/^mirrorlist=http/#mirrorlist=http/g /etc/yum.repos.d/*.repo


rpm -Uvh https://repo.zabbix.com/zabbix/6.4/rhel/7/x86_64/zabbix-release-6.4-1.el7.noarch.rpm
yum clean all 
yum install zabbix-agent -y
sed -i 's/Server=127.0.0.1/Server=zabbix.angstrem.yar/g' /etc/zabbix/zabbix_agentd.conf
systemctl restart zabbix-agent
systemctl enable zabbix-agent

firewall-cmd --permanent --new-service=zabbix
firewall-cmd --permanent --service=zabbix --add-port=10050/tcp
firewall-cmd --permanent --service=zabbix --set-short="Zabbix Agent"

# dnf -y zabbix-selinux-policy
# semodule -l | grep zabbix
zabbix
zabbix_policy

setsebool -P zabbix_can_network on
sesearch -b zabbix_can_network -A

```
