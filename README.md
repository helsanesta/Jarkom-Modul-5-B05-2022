# Jarkom-Modul-5-T7-2021      
### Laporan Resmi Pengerjaan Sesi Lab Jaringan Komputer     
        
#### Nama Anggota Kelompok :      
1. Helsa Nesta Dhaifullah (5025201005)     
2. Achmad Nashruddin Riskynanda	 (5025201021)      
3. Haniif Ahmad Jauhari (5025201224)  


## Soal Modul 5
Setelah kalian mempelajari semua modul yang telah diberikan, Loid ingin meminta bantuan untuk terakhir kalinya kepada kalian. Dan kalian dengan senang hati mau membantu Loid.
1. Tugas pertama kalian yaitu membuat topologi jaringan sesuai dengan rancangan yang diberikan Loid
2. Keterangan :	Eden adalah DNS Server
                WISE adalah DHCP Server
		Garden dan SSS adalah Web Server
		Jumlah Host pada Forger adalah 62 host
		Jumlah Host pada Desmond adalah 700 host
		Jumlah Host pada Blackbell adalah 255 host
		Jumlah Host pada Briar adalah 200 host        
3. Untuk menjaga perdamaian dunia, Loid ingin meminta kalian untuk membuat topologi tersebut menggunakan teknik CIDR atau VLSM setelah melakukan subnetting. 
4. Anya, putri pertama Loid, juga berpesan kepada anda agar melakukan Routing agar setiap perangkat pada jaringan tersebut dapat terhubung.
5. Tugas berikutnya adalah memberikan ip pada subnet Forger, Desmond, Blackbell, dan Briar secara dinamis menggunakan bantuan DHCP server. Kemudian kalian ingat bahwa kalian harus setting DHCP Relay pada router yang menghubungkannya.

## Jawaban Modul 
### Perhitungan VLSM
Berikut adalah topologi dan pembagian subnet

![image](Images/topologi_VLSM.png)

Berikut adalah Tree yang sudah kita buat

![](Images/tree_VLSM.png)

Berikut adalah Perhitungannya

![](Images/perhitungan_VLSM.png)

### Konfigurasi Network setiap node
[ Strix ]
```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 10.45.7.145
	netmask 255.255.255.252

auto eth2
iface eth2 inet static
	address 10.45.7.149
	netmask 255.255.255.252
```

[ Westalis ]
```
auto eth0
iface eth0 inet static
	address 10.45.7.146
	netmask 255.255.255.252
auto eth1
iface eth1 inet static
	address 10.45.7.129
	netmask 255.255.255.248
auto eth2
iface eth2 inet static
	address 10.45.7.1
	netmask 255.255.255.128
auto eth3
iface eth3 inet static
	address 10.45.0.1
	netmask 255.255.252.0
```

[  Ostania ]
```
auto eth0
iface eth0 inet static
	address 10.45.7.150
	netmask 255.255.255.252
auto eth1
iface eth1 inet static
	address  10.45.7.137
	netmask 255.255.255.248
auto eth2
iface eth2 inet static
	address  10.45.4.1
	netmask 255.255.254.0
auto eth3
iface eth3 inet static
	address  10.45.6.1
	netmask 255.255.255.0
```

[ Forger ], [ Desmond ], [ Blackbell ], [ Briar ]
```
auto eth0
iface eth0 inet dhcp

```
[ Eden ]
```
auto eth0
iface eth0 inet static
	address 10.45.7.130
	netmask 255.255.255.248
        gateway 10.45.7.129


```
[ Wise ]
```
auto eth0
iface eth0 inet static
	address 10.45.7.131
	netmask 255.255.255.248
        gateway 10.45.7.129
```

[ SSS ]
```
auto eth0
iface eth0 inet static
	address 10.45.7.139
	netmask 255.255.255.248
        gateway 10.45.7.137
```

[ Garden ]
```
auto eth0
iface eth0 inet static
	address 10.45.7.138
	netmask 255.255.255.248
        gateway 10.45.7.137

```

### Routing dan Konfigurasi DNS, Web server, DHCP Server, dan DHCP relay
[ Strix ]
```
route add -net 10.45.7.0 netmask 255.255.255.128 gw 10.45.7.146 #BLUENO
route add -net 10.45.0.0 netmask 255.255.252.0 gw 10.45.7.146 #CIPHER
route add -net 10.45.7.128 netmask 255.255.255.248 gw 10.45.7.146 #DORIKI & JIPANGU

route add -net 10.45.4.0 netmask 255.255.254.0 gw 10.45.7.150 #ELENA
route add -net 10.45.6.0 netmask 255.255.255.0 gw 10.45.7.150 #FUKORO
route add -net 10.45.7.136 netmask 255.255.255.248 gw 10.45.7.150 #Maingate & Jourge
```
[ Westalis ]
```
route add -net 0.0.0.0 netmask 0.0.0.0 gw 10.45.7.145
```

[ Ostania ]
```
route add -net 0.0.0.0 netmask 0.0.0.0 gw 10.45.7.149
```   
[ Eden adalah DNS Server ] </br>
Pada File  > /etc/bind/named.conf.options
```
apt update
apt install bind9 -y
echo '
options {
        directory "/var/cache/bind";
        forwarders {
                192.168.122.1;
        };
        allow-query { any; };
        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```
Kemudian lakukan Restart Dengan `service bind9 restart`     

[ Wise adalah DHCP Server ]  </br>
Pada File  > /etc/default/isc-dhcp-server     
```
apt update
apt install isc-dhcp-server -y
echo '
INTERFACES="eth0"
```
Pada File > /etc/dhcp/dhcpd.conf
```
echo '
ddns-update-style none;
option domain-name "example.org";
option domain-name-servers ns1.example.org, ns2.example.org;
default-lease-time 600;
max-lease-time 7200;
log-facility local7;
subnet 10.45.0.0 netmask 255.255.252.0 {
    range 10.45.0.2 10.45.3.254;
    option routers 10.45.0.1;
    option broadcast-address 10.45.3.255;
    option domain-name-servers 10.45.7.130;
    default-lease-time 360;
    max-lease-time 7200;
}
subnet 10.45.7.0 netmask 255.255.255.128 {
    range 10.45.7.2 10.45.7.126;
    option routers 10.45.7.1;
    option broadcast-address 10.45.7.127;
    option domain-name-servers 10.45.7.130;
    default-lease-time 720;
    max-lease-time 7200;
}
subnet 10.45.4.0 netmask 255.255.254.0 {
    range 10.45.4.2 10.45.5.254;
    option routers 10.45.4.1;
    option broadcast-address 10.45.5.255;
    option domain-name-servers 10.45.7.130;
    default-lease-time 720;
    max-lease-time 7200;
}
subnet 10.45.6.0 netmask 255.255.255.0 {
    range 10.45.6.2 10.45.6.254;
    option routers 10.45.6.1;
    option broadcast-address 10.45.6.255;
    option domain-name-servers 10.45.7.130;
    default-lease-time 720;
    max-lease-time 7200;
}
subnet 10.45.7.128 netmask 255.255.255.248 {}
subnet 10.45.7.144 netmask 255.255.255.252 {}
subnet 10.45.7.148 netmask 255.255.255.252 {}
subnet 10.45.7.136 netmask 255.255.255.248 {}
``` 
Lakukan Restart Dengan `service isc-dhcp-server restart`

[ Ostania sebagai DHCP Relay ]
```
apt update
apt install isc-dhcp-relay -y
echo '
SERVERS="10.45.7.131"
INTERFACES="eth2 eth3 eth1 eth0"
OPTIONS=""
' > /etc/default/isc-dhcp-relay
service isc-dhcp-relay restart
```

[ Westalis sebagai DHCP Relay ]
```
apt update
apt install isc-dhcp-relay -y
echo '
SERVERS="10.45.7.131"
INTERFACES="eth2 eth3 eth0 eth1"
OPTIONS=""
' > /etc/default/isc-dhcp-relay
service isc-dhcp-relay restart
```

[ Garden dan SSS adalah Web Server ]
```
apt update
apt install apache2 -y
service apache2 start
echo "$HOSTNAME" > /var/www/html/index.html
```

## Kendala
Tidak Ada
