#!/bin/bash

set -x

#Variable Hosts
XCC=172.16.132.248
VOPROS1=172.16.72.248
VOPROS2=172.16.72.251
RMS_ID=172.16.132.80
RDGW=172.16.132.207
Files_TP=172.16.132.251
Veeam_Cloud=172.16.165.249
ORIONVC=172.16.132.228

#Users
IP_GEORGE_EXT=213.141.132.209
IP_ARTUR_EXT=91.236.141.211
IP_SERGEY_EXT=78.90.235.11
IP_ANDREY_EXT="89.17.55.0/24"

#Branchs
SKTB16_1=5.101.200.82
SKTB16_2=217.77.110.22
BITBUCKED_PIPELINES="34.199.54.113/32,34.232.25.90/32,34.232.119.183/32,34.236.25.177/32,35.171.175.212/32,52.54.90.98/32,52.202.195.162/32,52.203.14.55/32,52.204.96.37/32,34.218.156.209/32,34.218.168.212/32,52.41.219.63/32,35.155.178.254/32,35.160.177.10/32,34.216.18.129/32,95.29.44.136/32,62.217.190.243/32,13.52.5.96/28,13.236.8.224/28,18.136.214.96/28,18.184.99.224/28,18.234.32.224/28,18.246.31.224/28,52.215.192.224/28,104.192.137.240/28,104.192.138.240/28,104.192.140.240/28,104.192.142.240/28,104.192.143.240/28,185.166.143.240/28,185.166.142.240/28,217.118.92.54/32"

IF_EXT1=eth3
IP_EXT1=$(ip addr show ${IF_EXT1} | grep -m1 "inet\s" | cut -d" " -f6 | cut -d"/" -f1)
IF_INT=eth0
IP_ADMIN="172.16.169.140,172.16.169.33,172.16.82.20,172.16.128.0/21"


IPT=iptables

$IPT -F
$IPT -X
$IPT -t nat -F
$IPT -t nat -X
#$IPT -t nat -N ipcustom


$IPT -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
$IPT -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
$IPT -A INPUT -i lo -j ACCEPT
$IPT -A INPUT -i ${IF_INT} -s ${IP_ADMIN} -j ACCEPT
$IPT -A INPUT -i ${IF_INT} -p udp --dport 123 -j ACCEPT
$IPT -A INPUT -p ICMP --icmp-type 3 -j ACCEPT
$IPT -A INPUT -p ICMP --icmp-type 4 -j ACCEPT
$IPT -A INPUT -p ICMP --icmp-type 8 -j ACCEPT
$IPT -A INPUT -p ICMP --icmp-type 12 -j ACCEPT

$IPT -A INPUT -s ${IP_ANDREY_EXT} -j ACCEPT


#$IPT -A FORWARD -f -m limit --limit 3/minute --limit-burst 3 -j LOG --log-level debug --log-prefix "IPT FRAGMENTED: "
$IPT -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
$IPT -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
$IPT -A FORWARD -i ${IF_INT} -o ${IF_EXT1} -j ACCEPT


#Prerouting
$IPT -t nat -A PREROUTING -p tcp -i ${IF_EXT1} -d ${IP_EXT1} --dport 1950 -j DNAT --to-destination ${XCC}
$IPT -t nat -A PREROUTING -p tcp -i ${IF_EXT1} -d ${IP_EXT1} --dport 5306 -j DNAT --to-destination ${VOPROS1}:3306
$IPT -t nat -A PREROUTING -p tcp -i ${IF_EXT1} -d ${IP_EXT1} --dport 443 -j DNAT --to-destination ${VOPROS1}
$IPT -t nat -A PREROUTING -p tcp -i ${IF_EXT1} -d ${IP_EXT1} --dport 32 -j DNAT --to-destination ${VOPROS1}:22
$IPT -t nat -A PREROUTING -p tcp -i ${IF_EXT1} -d ${IP_EXT1} --dport 42 -j DNAT --to-destination ${VOPROS2}:22
$IPT -t nat -A PREROUTING -p tcp -i ${IF_EXT1} -d ${IP_EXT1} --dport 7533 -j DNAT --to-destination ${RMS_ID}:7533
$IPT -t nat -A PREROUTING -s ${IP_GEORGE_EXT},${IP_ARTUR_EXT},${IP_SERGEY_EXT},${SKTB16_1},${SKTB16_2} -p tcp -i ${IF_EXT1} -d ${IP_EXT1} --dport 7543 -j DNAT --to-destination ${RDGW}
$IPT -t nat -A PREROUTING -p tcp -i ${IF_EXT1} -d ${IP_EXT1} --dport 8443 -j DNAT --to-destination ${Files_TP}
$IPT -t nat -A PREROUTING -s ${IP_GEORGE_EXT},77.223.127.162,95.29.44.52,80.76.225.210 -p tcp -i ${IF_EXT1} -d ${IP_EXT1} --dport 9180 -j DNAT --to-destination ${Veeam_Cloud}
$IPT -t nat -A PREROUTING -s ${BITBUCKED_PIPELINES} -p tcp -i ${IF_EXT1} -d ${IP_EXT1} --dport 7722  -j DNAT --to-destination 172.16.39.103:22
$IPT -t nat -A PREROUTING -s ${BITBUCKED_PIPELINES} -p tcp -i ${IF_EXT1} -d ${IP_EXT1} --dport 7780  -j DNAT --to-destination 172.16.39.103:80
$IPT -t nat -A PREROUTING -s ${BITBUCKED_PIPELINES} -p tcp -i ${IF_EXT1} -d ${IP_EXT1} --dport 7723  -j DNAT --to-destination 172.16.39.8:22
$IPT -t nat -A PREROUTING -s ${BITBUCKED_PIPELINES} -p tcp -i ${IF_EXT1} -d ${IP_EXT1} --dport 7724  -j DNAT --to-destination 172.16.39.8:443
$IPT -t nat -A PREROUTING -s ${BITBUCKED_PIPELINES} -p tcp -i ${IF_EXT1} -d ${IP_EXT1} --dport 7743  -j DNAT --to-destination 172.16.39.114:22
$IPT -t nat -A PREROUTING -s ${BITBUCKED_PIPELINES} -p tcp -i ${IF_EXT1} -d ${IP_EXT1} --dport 7781  -j DNAT --to-destination 172.16.39.114:80






#Forwards
$IPT -A FORWARD -p tcp -d ${XCC} --dport 1950 -j ACCEPT
$IPT -A FORWARD -p tcp -d ${VOPROS1} --dport 3306 -j ACCEPT
$IPT -A FORWARD -p tcp -d ${RMS_ID} --dport 7533 -j ACCEPT
$IPT -A FORWARD -p tcp -d ${RDGW} --dport 7543 -j ACCEPT
$IPT -A FORWARD -p tcp -d ${Files_TP} --dport 8443 -j ACCEPT
$IPT -A FORWARD -p tcp -d ${Veeam_Cloud} --dport 9180 -j ACCEPT
$IPT -A FORWARD -p tcp -d 172.16.39.114 --dport 22 -j ACCEPT
$IPT -A FORWARD -p tcp -d 172.16.39.114 --dport 80 -j ACCEPT
$IPT -A FORWARD -p tcp -d 172.16.39.103 --dport 22 -j ACCEPT
$IPT -A FORWARD -p tcp -d 172.16.39.103 --dport 80 -j ACCEPT
$IPT -A FORWARD -p tcp -d 172.16.39.8 --dport 443 -j ACCEPT
$IPT -A FORWARD -p tcp -d 172.16.39.8 --dport 22 -j ACCEPT
#Krasnov rdrs - for move
$IPT -A FORWARD -p tcp -d ${VOPROS1} --dport 443 -j ACCEPT
$IPT -A FORWARD -p tcp -d ${VOPROS1} -s 90.156.153.164 --dport 3306 -j ACCEPT
$IPT -A FORWARD -p tcp -d ${VOPROS1} --dport 22 -j ACCEPT
$IPT -A FORWARD -p tcp -d ${VOPROS2} --dport 22 -j ACCEPT
#####
$IPT -A FORWARD -d ${ORIONVC} -p tcp --dport 80 -j ACCEPT
#$IPT -A FORWARD -m limit --limit 3/minute --limit-burst 3 -j LOG --log-level debug --log-prefix "IPT FORWARD packet died: "
$IPT -A FORWARD -j DROP




#ntp fix. if not allowed incoming traffic UDP:123. snat from udp:119
#$IPT -t nat -A POSTROUTING -o ${IF_EXT1} -p udp --sport 123 --dport 123 -j SNAT --to-source ${IP_EXT1}:119

$IPT -I INPUT -i eth3 -j ACCEPT    

#POSTROUTING
#$IPT -t nat -A POSTROUTING -o ${IF_INT}  -p tcp --dport 1950 -d ${XCC} -j SNAT --to-source ${IP_EXT1}
#$IPT -t nat -A POSTROUTING -o ${IF_INT}  -p tcp --dport 7533 -d ${RMS_ID} -j SNAT --to-source ${IP_EXT1}
#$IPT -t nat -A POSTROUTING -o ${IF_INT}  -p tcp --dport 7543 -d ${RDGWD} -j SNAT --to-source ${IP_EXT1}
#$IPT -t nat -A POSTROUTING -o ${IF_INT}  -p tcp --dport 8443 -d ${Files_TP} -j SNAT --to-source ${IP_EXT1}
#$IPT -t nat -A POSTROUTING -o ${IF_INT}  -p tcp --dport 9180 -d ${Veeam_Cloud} -j SNAT --to-source ${IP_EXT1}
#$IPT -t nat -A POSTROUTING -o ${IF_INT}  -p tcp --dport 22 -d 172.16.39.8 -j SNAT --to-source ${IP_EXT1}
#$IPT -t nat -A POSTROUTING -o ${IF_INT}  -p tcp --dport 443 -d 172.16.39.8 -j SNAT --to-source ${IP_EXT1}
#$IPT -t nat -A POSTROUTING -o ${IF_INT}  -p tcp --dport 22 -d 172.16.39.103 -j SNAT --to-source ${IP_EXT1}
#$IPT -t nat -A POSTROUTING -o ${IF_INT}  -p tcp --dport 22 -d 172.16.39.114 -j SNAT --to-source ${IP_EXT1}
#Krasnov rdrs - for move
#$IPT -t nat -A POSTROUTING -o ${IF_INT}  -p tcp --dport 443 -d ${VOPROS1} -j SNAT --to-source ${IP_EXT1}
#$IPT -t nat -A POSTROUTING -o ${IF_INT}  -p tcp --dport 3306 -d ${VOPROS1} -j SNAT --to-source ${IP_EXT1}
#$IPT -t nat -A POSTROUTING -o ${IF_INT}  -p tcp --dport 22 -d ${VOPROS1} -j SNAT --to-source ${IP_EXT1}
#$IPT -t nat -A POSTROUTING -o ${IF_INT}  -p tcp --dport 22 -d ${VOPROS2} -j SNAT --to-source ${IP_EXT1}
####
#$IPT -t nat -A POSTROUTING -o ${IF_INT} -p tcp -d ${ORIONVC} --dport 80 -j SNAT --to-source  ${IP_EXT1}
#$IPT -t nat -A POSTROUTING -o ${IF_EXT1} -j ipcustom
$IPT -t nat -A POSTROUTING -o ${IF_EXT1} -j SNAT --to-source ${IP_EXT1}





$IPT -t raw -F
#$IPT -t raw -A PREROUTING -i ${IF_INT} -d 10.0.0.0/8 -j CT --notrack
#$IPT -t raw -A PREROUTING -i ${IF_INT} -d 172.16.0.0/12 -j CT --notrack
#$IPT -t raw -A PREROUTING -i ${IF_INT} -d 192.168.0.0/16 -j CT --notrack
$IPT -t raw -A PREROUTING -p tcp --dport 21 -j CT --helper ftp
$IPT -t raw -A PREROUTING -p udp --dport 5060 -j CT --helper sip
$IPT -t raw -A PREROUTING -p tcp --dport 5060 -j CT --helper sip
