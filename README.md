# Jarkom-Modul-5-B04-2021

Naufal Fajar Imani             05111940000007

Jundullah H. R.                05111940000144

Yeremia D Limantara            05111940000232

---

A. Tugas pertama kalian yaitu membuat topologi jaringan sesuai dengan rancangan yang diberikan Luffy dibawah ini:
![image](https://user-images.githubusercontent.com/40772378/145680897-ff9e544d-c519-4485-8201-6ef1dbf2fbfe.png)
Keterangan :   Doriki adalah DNS Server
               Jipangu adalah DHCP Server
               Maingate dan Jorge adalah Web Server
               Jumlah Host pada Blueno adalah 100 host
               Jumlah Host pada Cipher adalah 700 host
               Jumlah Host pada Elena adalah 300 host
               Jumlah Host pada Fukurou adalah 200 host


B. Karena kalian telah belajar subnetting dan routing, Luffy ingin meminta kalian untuk membuat topologi tersebut menggunakan teknik CIDR atau VLSM. setelah melakukan subnetting, 

C. Kalian juga diharuskan melakukan Routing agar setiap perangkat pada jaringan tersebut dapat terhubung.

D. Tugas berikutnya adalah memberikan ip pada subnet Blueno, Cipher, Fukurou, dan Elena secara dinamis menggunakan bantuan DHCP server. Kemudian kalian ingat bahwa kalian harus setting DHCP Relay pada router yang menghubungkannya.


1. Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi Foosha menggunakan iptables, tetapi Luffy tidak ingin menggunakan MASQUERADE.
2. Kalian diminta untuk mendrop semua akses HTTP dari luar Topologi kalian pada server yang merupakan DHCP Server dan DNS Server demi menjaga keamanan.
3. Karena kelompok kalian maksimal terdiri dari 3 orang. Luffy meminta kalian untuk membatasi DHCP dan DNS Server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan menggunakan iptables, selebihnya didrop.
4. Kemudian kalian diminta untuk membatasi akses ke Doriki yang berasal dari subnet Blueno, Cipher, Elena dan Fukuro dengan beraturan sebagai berikut
5. Akses dari subnet Blueno dan Cipher hanya diperbolehkan pada pukul 07.00 - 15.00 pada hari Senin sampai Kamis.Selain itu di reject
6. Akses dari subnet Elena dan Fukuro hanya diperbolehkan pada pukul 15.01 hingga pukul 06.59 setiap harinya. Selain itu di reject
7. Karena kita memiliki 2 Web Server, Luffy ingin Guanhao disetting sehingga setiap request dari client yang mengakses DNS Server akan didistribusikan secara bergantian pada Jorge dan Maingate

##Water7
`apt-get update
DEBIAN_FRONTEND=noninteractive apt-get install -y isc-dhcp-relay && cp /root/isc-dhcp-relay /etc/default && dhcrelay 10.9.7.131 # IP Jipangu
DEBIAN_FRONTEND=noninteractive apt-get install -y --reinstall rsyslog && service rsyslog restart

route add -net 0.0.0.0 netmask 0.0.0.0 gw 10.9.7.145

# Fix bug habis restart iptables error
iptables -F

# Revisi default policy FORWARD jadi DROP
iptables -P FORWARD DROP

# Semua revisi dijadikan -A FORWARD

# Revisi allow forward dari DNS dan DHCP
iptables -A FORWARD -s 10.9.7.130 -j ACCEPT
iptables -A FORWARD -s 10.9.7.131 -j ACCEPT

iptables -A FORWARD -p icmp -d 10.9.7.130 -m connlimit --connlimit-above 3 -j REJECT
iptables -A FORWARD -p icmp -d 10.9.7.131 -m connlimit --connlimit-above 3 -j REJECT

# Revisi allow forward ke DNS dan DHCP selain ICMP
iptables -A FORWARD ! -p icmp -d 10.9.7.130 -j ACCEPT
iptables -A FORWARD ! -p icmp -d 10.9.7.131 -j ACCEPT

# block blueno cipher
# Revisi ditambah -d IP DORIKI
# Hapus -m state .... -> tidak tahu manfaatnya
iptables -A FORWARD -s 10.9.7.0/25 -d 10.9.7.130 -m time --timestart 07:00 --timestop 15:00 --weekdays Mon,Tue,Wed,Thu -j ACCEPT
iptables -A FORWARD -s 10.9.0.0/22 -d 10.9.7.130 -m time --timestart 07:00 --timestop 15:00 --weekdays Mon,Tue,Wed,Thu -j ACCEPT

# Revisi selalu allow paket ke/dari cipher atau blueno selain yang di atas
iptables -A FORWARD -s 10.9.7.0/25 ! -d 10.9.7.130 -j ACCEPT
iptables -A FORWARD -s 10.9.0.0/25 ! -d 10.9.7.130 -j ACCEPT
iptables -A FORWARD -d 10.9.7.0/22 -j ACCEPT
iptables -A FORWARD -d 10.9.0.0/22 -j ACCEPT`

##Foosha
`# Fix bug habis restart iptables error 	 
iptables -F

# https://gist.github.com/tomasinouk/eec152019311b09905cd
# assuming eth0 is STA interface
ip=$(ip -o addr show up primary scope global eth0 |
  	while read -r num dev fam addr rest; do echo ${addr%/*}; done)
echo $ip

# all packets leaving wlan1 will change source IP to STA interface IP
iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to $ip

# block request to dns and dhcp
# Revisi iptables -A OUTPUT -s 10.9.7.128/29 -j DROP
iptables -A FORWARD -i eth0 -d 10.9.7.128/29 -p tcp --dport 80 -j DROP

apt-get update
DEBIAN_FRONTEND=noninteractive apt-get install -y isc-dhcp-relay && cp /root/isc-dhcp-relay /etc/default && dhcrelay 10.9.7.131 # IP Jipangu
DEBIAN_FRONTEND=noninteractive apt-get install -y --reinstall rsyslog && service rsyslog restart

route add -net 10.9.7.128 netmask 255.255.255.248 gw 10.9.7.146
route add -net 10.9.7.0 netmask 255.255.255.128 gw 10.9.7.146
route add -net 10.9.0.0 netmask 255.255.252.0 gw 10.9.7.146
route add -net 10.9.4.0 netmask 255.255.254.0 gw 10.9.7.150
route add -net 10.9.6.0 netmask 255.255.255.0 gw 10.9.7.150
route add -net 10.9.7.136 netmask 255.255.255.248 gw 10.9.7.150`

##Guanhao
`apt-get update
DEBIAN_FRONTEND=noninteractive apt-get install -y isc-dhcp-relay && cp /root/isc-dhcp-relay /etc/default && dhcrelay 10.9.7.131 # IP Jipangu
DEBIAN_FRONTEND=noninteractive apt-get install -y --reinstall rsyslog && service rsyslog restart

route add -net 0.0.0.0 netmask 0.0.0.0 gw 10.9.27.149

# Fix bug habis restart iptables error 	 
iptables -F

# load balance jorge maingate
# Revisi jadinya per IP webservernya
#iptables -A PREROUTING -t nat -p tcp -d 10.9.7.137 --dport 80 \
#     	-m statistic --mode nth --every 2 --packet 0          	\
#     	-j DNAT --to-destination 10.9.7.138:80

#iptables -A PREROUTING -t nat -p tcp -d 10.9.7.137 --dport 80 \
#     	-j DNAT --to-destination 10.9.7.139:80

iptables -A PREROUTING -t nat -p tcp -d 10.9.7.138 --dport 80 \
     	-m statistic --mode nth --every 2 --packet 0          	\
     	-j DNAT --to-destination 10.9.7.138:80

iptables -A PREROUTING -t nat -p tcp -d 10.9.7.138 --dport 80 \
     	-j DNAT --to-destination 10.9.7.139:80

iptables -A PREROUTING -t nat -p tcp -d 10.9.7.139 --dport 80 \
     	-m statistic --mode nth --every 2 --packet 0          	\
     	-j DNAT --to-destination 10.9.7.139:80

iptables -A PREROUTING -t nat -p tcp -d 10.9.7.139 --dport 80 \
     	-j DNAT --to-destination 10.9.7.138:80

iptables -t filter -A FORWARD -d 10.9.7.138 -j ACCEPT
iptables -t filter -A FORWARD -d 10.9.7.139 -j ACCEPT
iptables -t filter -A FORWARD -s 10.9.7.138 -j ACCEPT
iptables -t filter -A FORWARD -s 10.9.7.139 -j ACCEPT

# Semua revisi dijadikan -A FORWARD

# block elena fukurou
# Revisi hapus --state karena tidak tahu manfaatnya
# Revisi negasi aturan waktu
iptables -A FORWARD -s 10.9.4.0/23 -m time --timestart 07:00 --timestop 15:00 -j DROP
iptables -A FORWARD -s 10.9.6.0/24 -m time --timestart 07:00 --timestop 15:00 -j DROP
# iptables -A FORWARD -s 10.9.4.0/23 -m time --timestart 00:00 --timestop 06:59 -j ACCEPT
# iptables -A FORWARD -s 10.9.6.0/24 -m time --timestart 00:00 --timestop 06:59 -j ACCEPT`

##Doriki
`apt-get update
apt-get install bind9 -y`

##Jipangu
`apt-get update
apt-get install -y isc-dhcp-server
cp /root/dhcpd/dhcpd.conf /etc/dhcp/dhcpd.conf
service isc-dhcp-server restart

>> File dhcpd.conf Jipangu
subnet 10.9.7.0 netmask 255.255.255.128 {
	range 10.9.7.2 10.9.7.126;
	option routers 10.9.7.1;
	option domain-name-servers 192.168.122.1;
	default-lease-time 360; #6m
	max-lease-time 720; #12m
}

subnet 10.9.0.0 netmask 255.255.252.0 {
	range 10.9.0.2 10.9.3.254;
	option routers 10.9.0.1;
	option domain-name-servers 192.168.122.1;
	default-lease-time 360; #6m
	max-lease-time 720; #12m
}

subnet 10.9.4.0 netmask 255.255.254.0 {
	range 10.9.4.2 10.9.5.254;
	option routers 10.9.4.1;
	option domain-name-servers 192.168.122.1;
	default-lease-time 360; #6m
	max-lease-time 720; #12m
}

subnet 10.9.6.0 netmask 255.255.255.0 {
	range 10.9.6.2 10.9.6.254;
	option routers 10.9.6.1;
	option domain-name-servers 192.168.122.1;
	default-lease-time 360; #6m
	max-lease-time 720; #12m
}


# https://unix.stackexchange.com/a/294947 Thanks for the answer!
subnet 10.9.7.128 netmask 255.255.255.248 {
	option routers 10.9.7.129;
}`

##Jorge
`apt-get update
apt-get install -y apache2 && service apache2 restart`

#Maingate
`apt-get update
apt-get install -y apache2 && service apache2 restart`

