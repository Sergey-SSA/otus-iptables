# otus-iptables

Домашнее задание
Сценарии iptables
1) реализовать knocking port
- centralRouter может попасть на ssh inetrRouter через knock скрипт
пример в материалах
2) добавить inetRouter2, который виден(маршрутизируется) с хоста
3) запустить nginx на centralServer
4) пробросить 80й порт на inetRouter2 8080
5) дефолт в инет оставить через inetRouter

В файле `iptables` лежат правила:

```
*filter
:INPUT DROP [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:TRAFFIC - [0:0]
:SSH-INPUT - [0:0]
:SSH-INPUTTWO - [0:0]
# TRAFFIC chain for Port Knocking. The correct port sequence in this example is  8881 -> 7777 -> 9991; any other sequence will drop the traffic 
-A INPUT -j TRAFFIC
-A TRAFFIC -p icmp --icmp-type any -j ACCEPT
-A TRAFFIC -m state --state ESTABLISHED,RELATED -j ACCEPT
-A TRAFFIC -m state --state NEW -m tcp -p tcp --dport 22 -m recent --rcheck --seconds 30 --name SSH2 -j ACCEPT
-A TRAFFIC -m state --state NEW -m tcp -p tcp -m recent --name SSH2 --remove -j DROP
-A TRAFFIC -m state --state NEW -m tcp -p tcp --dport 9991 -m recent --rcheck --name SSH1 -j SSH-INPUTTWO
-A TRAFFIC -m state --state NEW -m tcp -p tcp -m recent --name SSH1 --remove -j DROP
-A TRAFFIC -m state --state NEW -m tcp -p tcp --dport 7777 -m recent --rcheck --name SSH0 -j SSH-INPUT
-A TRAFFIC -m state --state NEW -m tcp -p tcp -m recent --name SSH0 --remove -j DROP
-A TRAFFIC -m state --state NEW -m tcp -p tcp --dport 8881 -m recent --name SSH0 --set -j DROP
-A SSH-INPUT -m recent --name SSH1 --set -j DROP
-A SSH-INPUTTWO -m recent --name SSH2 --set -j DROP 
-A TRAFFIC -j DROP
COMMIT
```

# Работоспособность

Запускаем стенд `vagrant up`

Поключаемся под рутом к `centralRouter`

Коннект к `192.168.225.1` неудачный
```
[root@centralRouter vagrant]# ssh vagrant@192.168.255.1
^C
```

Чекаем порты 
```
[root@centralRouter vagrant]# hping3 -S 192.168.255.1 -p 8881 -c 1; hping3 -S 192.168.255.1 -p 7777 -c 1; hping3 -S 192.168.255.1 -p 9991 -c 1
HPING 192.168.255.1 (eth1 192.168.255.1): S set, 40 headers + 0 data bytes

--- 192.168.255.1 hping statistic ---
1 packets transmitted, 0 packets received, 100% packet loss
round-trip min/avg/max = 0.0/0.0/0.0 ms
HPING 192.168.255.1 (eth1 192.168.255.1): S set, 40 headers + 0 data bytes

--- 192.168.255.1 hping statistic ---
1 packets transmitted, 0 packets received, 100% packet loss
round-trip min/avg/max = 0.0/0.0/0.0 ms
HPING 192.168.255.1 (eth1 192.168.255.1): S set, 40 headers + 0 data bytes

--- 192.168.255.1 hping statistic ---
1 packets transmitted, 0 packets received, 100% packet loss
round-trip min/avg/max = 0.0/0.0/0.0 ms
```
И коннектимся еще раз

```
[root@centralRouter vagrant]# ssh vagrant@192.168.255.1
The authenticity of host '192.168.255.1 (192.168.255.1)' can't be established.
RSA key fingerprint is SHA256:ZcwraORtJEEK32AcXFsghO5kelZjsBrg/K/WF79dL44.
RSA key fingerprint is MD5:80:fd:95:a2:0a:68:35:8e:eb:76:37:24:aa:a6:d8:17.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.255.1' (RSA) to the list of known hosts.
vagrant@192.168.255.1's password:
```

