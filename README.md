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

* Doriki
```
apt-get update
apt-get install bind9 -y
```
* Jipangu
```apt-get update
apt-get install -y isc-dhcp-server
cp /root/dhcpd/dhcpd.conf /etc/dhcp/dhcpd.conf
service isc-dhcp-server restart
```

* File dhcpd.conf Jipangu
```
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
}
```
* Jorge
```
apt-get update
apt-get install -y apache2 && service apache2 restart
```

* Maingate
```
apt-get update
apt-get install -y apache2 && service apache2 restart
```

