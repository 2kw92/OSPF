# OSPF
дз по теме статическая и динамическая маршрутизация


Для проверки дз запускаем vagrantfile,который развернет все необходимую ифраструктуру.       
Развернутся 3 роутера за которыми будут подсети 172.16.1.1/29 172.16.2.1/29 172.16.3.1/29       
которые и будут объеденены с помощью ospf.         


После того как стенд развернется проверяем маршруты:         

```
[root@r1 ~]# ip r s
default via 10.0.2.2 dev eth0 proto dhcp metric 100
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100
172.16.1.0/29 dev eth1 proto kernel scope link src 172.16.1.1 metric 101
172.16.2.0/29 via 192.168.0.2 dev eth2 proto zebra metric 20
172.16.3.0/29 via 192.168.2.2 dev eth3 proto zebra metric 20
192.168.0.0/29 dev eth2 proto kernel scope link src 192.168.0.1 metric 102
192.168.1.0/29 proto zebra metric 20
        nexthop via 192.168.0.2 dev eth2 weight 1
        nexthop via 192.168.2.2 dev eth3 weight 1
192.168.2.0/29 dev eth3 proto kernel scope link src 192.168.2.1 metric 103

[root@r2 ~]# ip r s
default via 10.0.2.2 dev eth0 proto dhcp metric 101
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 101
172.16.1.0/29 via 192.168.0.1 dev eth2 proto zebra metric 20
172.16.2.0/29 dev eth1 proto kernel scope link src 172.16.2.1 metric 100
172.16.3.0/29 via 192.168.1.1 dev eth3 proto zebra metric 20
192.168.0.0/29 dev eth2 proto kernel scope link src 192.168.0.2 metric 102
192.168.1.0/29 dev eth3 proto kernel scope link src 192.168.1.2 metric 103
192.168.2.0/29 proto zebra metric 20
        nexthop via 192.168.0.1 dev eth2 weight 1
        nexthop via 192.168.1.1 dev eth3 weight 1

[root@r3 ~]# ip r s
default via 10.0.2.2 dev eth0 proto dhcp metric 100
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100
172.16.1.0/29 via 192.168.2.1 dev eth2 proto zebra metric 20
172.16.2.0/29 via 192.168.1.2 dev eth3 proto zebra metric 20
172.16.3.0/29 dev eth1 proto kernel scope link src 172.16.3.1 metric 101
192.168.0.0/29 proto zebra metric 20
        nexthop via 192.168.2.1 dev eth2 weight 1
        nexthop via 192.168.1.2 dev eth3 weight 1
192.168.1.0/29 dev eth3 proto kernel scope link src 192.168.1.1 metric 103
192.168.2.0/29 dev eth2 proto kernel scope link src 192.168.2.2 metric 102
```         

Изначально роутинг симметричный:       
```
[root@r1 ~]# tracepath -n 172.16.2.1
 1?: [LOCALHOST]                                         pmtu 1500
 1:  172.16.2.1                                            1.266ms reached
 1:  172.16.2.1                                            0.671ms reached
     Resume: pmtu 1500 hops 1 back 1
```       

Чтобы сделать ассимтеричный роутинг изменим вес пути 192.168.0.0/29:        
```
[root@r1 ~]# vtysh

Hello, this is Quagga (version 0.99.22.4).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

r1# configure  terminal
r1(config)# interface  eth2
r1(config-if)# ip ospf cost 1000
r1(config-if)# exit
r1(config)# exit
r1# exit
[root@r1 ~]# tracepath -n 172.16.2.1
 1?: [LOCALHOST]                                         pmtu 1500
 1:  192.168.2.2                                           1.212ms
 1:  192.168.2.2                                           0.832ms
```        

Чтобы компенисровать и вернуть сииметрию добавим вес пути на eth3:        
```
[root@r1 ~]# vtysh

Hello, this is Quagga (version 0.99.22.4).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

r1# configure  terminal
r1(config)# interface  eth3
r1(config-if)# ip ospf  cost 1000
r1(config-if)# exit
r1(config)# exit
r1# exit
[root@r1 ~]# tracepath -n 172.16.2.1
 1?: [LOCALHOST]                                         pmtu 1500
 1:  172.16.2.1                                            0.682ms reached
 1:  172.16.2.1                                            0.742ms reached
     Resume: pmtu 1500 hops 1 back 1
```

На этом проверка дз заканчивается.
