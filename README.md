# Jarkom_Modul3_Lapres_B06

## Jawaban 

### 1. Membuat Topologi Baru
- Membuat topologi berdasarkan soal

cara :
+ rm kota sebelumnya yang ada di praktikum
  ```
  rm SURABAYA MALANG MOJOKERTO SIDOARJO GRESIK BANYUWANGI
  ```
+ menambahkan kota malang sebagai DNS server
+ kota Tuban sebagai DHCP server
+ kota Mojokerto sebagai Proxy server
+ menambahkan kota dalam klien, yaitu Sidoarjo, Gresik, Banyuwangi, Madiun
+ memory maisng-masing ktoa diset sesuai ketentuan soal
   ```
   Switch
  uml_switch -unix switch1 > /dev/null < /dev/null &
  uml_switch -unix switch2 > /dev/null < /dev/null &
  uml_switch -unix switch3 > /dev/null < /dev/null &

  Router
  xterm -T SURABAYA -e linux ubd0=SURABAYA,jarkom umid=SURABAYA eth0=tuntap,,,10.151.74.30 eth1=daemon,,,switch1 eth2=daemon,,,switch3 eth3=daemon,,,switch2 mem=256M &

  Server
  xterm -T MALANG -e linux ubd0=MALANG,jarkom umid=MALANG eth0=daemon,,,switch2 mem=160M &
  xterm -T MOJOKERTO -e linux ubd0=MOJOKERTO,jarkom umid=MOJOKERTO eth0=daemon,,,switch2 mem=128M &
  xterm -T TUBAN -e linux ubd0=TUBAN,jarkom umid=TUBAN eth0=daemon,,,switch2 mem=128M &

  Client-1
  xterm -T SIDOARJO -e linux ubd0=SIDOARJO,jarkom umid=SIDOARJO eth0=daemon,,,switch1 mem=64M &
  xterm -T GRESIK -e linux ubd0=GRESIK,jarkom umid=GRESIK eth0=daemon,,,switch1 mem=64M &

  Client-2
  xterm -T BANYUWANGI -e linux ubd0=BANYUWANGI,jarkom umid=BANYUWANGI eth0=daemon,,,switch3 mem=64M &
  xterm -T MADIUN -e linux ubd0=MADIUN,jarkom umid=MADIUN eth0=daemon,,,switch3 mem=64M &
   ```
   
   
### 2. Surabaya menjadi perantara ( DHCP Relay ) antara DHCP Server dan client.

cara :
+ menginstall isc-dhcp relay pada uml surabaya
  ```
  apt-get install isc-dhcp-relay
  ```
+ menambahkan command pada file default dhcp relay
  ```
  nano etc/default/isc-dhcp-relay
  ```
+ menambahkan `SERVERS="10.151.83.60"` dalam DHCP relay forward (Tuban)
+ menambahkan `INTERFACES="eth1 eth2 eth3"` dalam interface DHCP relay request


### 3. Client pada subnet 1 mendapatkan range IP dari 192.168.0.10 sampai 192.168.0.100 dan 192.168.0.110 sampai 192.168.0.200.

 cara:
+ pada UML Tuban, ketik seperti berikut
  ```
  nano etc/default/isc-dhcp-server
  ```
+ menambahkan `INTERFACEv4="eth0"`
+ kemudian, pada UML Tuban lagi, ketikan seperti berikut
  ```
  nano etc/dhcp/dhcp.conf
  ```
+ tambahkan command subnet
  ```
  subnet 192.168.0.0 netmask 255.255.255.0 {
  range 192.168.0.10 192.168.0.100;
  range 192.168.0.110 192.168.0.200;
  option routers 192.168.0.1;
  option broadcast-address 192.168.0.255;
  option domain-name-servers 10.151.77.66, 202.46.129.2;
  default-lease-time 300;
  max-lease-time 300;
  }
  ```
+ restart isc-dhcp
  ```
  service isc-dhcp-server restart
  ```
+ melakukan pengecekan IP pada UML Gresik & Sidoarjo dengan mengetikan pada masing-masing UML sebagai berikut 
  ```
  ifconfig
  ```


### 4. Client pada subnet 3 mendapatkan range IP dari 192.168.1.50 sampai 192.168.1.70.

cara :
+ ketik command berikut pada UML Tuban
  ```
  nano etc/dhcp/dhcp.conf
  ```
+ mengubah range IP dengan mengetikkan
  ```
  subnet 192.168.0.0 netmask 255.255.255.0 {
  range 192.168.1.50 192.168.1.70;
  option routers 192.168.1.1;
  option broadcast-address 192.168.1.255;
  option domain-name-servers 10.151.77.66, 202.46.129.2;
  default-lease-time 600;
  max-lease-time 600;
  }
  ```
+ melakukan konfigurasi pada banyuwangi dan madiun dengan menulis
  ```
  ifconfig
  ```

### 5. Client mendapatkan DNS Malang dan DNS 202.46.129.2 dari DHCP

cara :
+ ketik command berikut pada UML Tuban
  ```
  nano etc/dhcp/dhcp.conf
  ```
+ mengubah domain-name-sercers (IP Malang)
  ```
  10.151.83.58, 202.46.129.2
  ```
 
  
### 6.	Client di subnet 1 mendapatkan peminjaman alamat IP selama 5 menit, sedangkan client pada subnet 3 mendapatkan peminjaman IP selama 10 menit.

cara :
+ ketik command berikut pada UML Tuban
  ```
  nano etc/dhcp/dhcp.conf
  ```
+ mengubah `default-lease-time` dan `max-lease-time` pada subnet 1 dan 3
+ pada subnet 1 ditulis `default-lease-time 300` (5menit)
+ pada subnet 3 ditulis `default-lease-time 600` (10menit)


### 7. User autentikasi milik Anri

cara :
+ install squid pada uml Mojokerto (proxy server)
  ```
  apt-get install squid
  ```
+ mengecek apakah squid sudah running
  ```
  service squid status
  ```
+ kemdudian install apache-2
  ```
  apt-get install apache2-utils
  ```
+ buat user dan passsword
  ```
  htpasswd -c /etc/squid/passwd userta_b06
  inipassw0rdta_b06
  ```
+ ketik command
  ```
  nano etc/squid/squid.conf
  ```
+ mengecek apakah sudah diganti `nano etc/squid/passwd
+ melakukan konfigurasi squid
  ```
  http_port 8080
  visible_hostname mojokerto
  auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
  auth_param basic children 5
  auth_param basic realm Proxy
  auth_param basic credentialsttl 2 hours
  auth_param basic casesensitive on
  acl USERS proxy_auth REQUIRED
  http_access deny !USERS
  ```
+ `restart squid`
+ ubah proxy browser menjadi ip Mojokerto
+ coba buka website yang diinginkan


### 8. Menjadwal pengerjaan TA setiap hari Selasa-Rabu pukul 13.00-18.00

cara :
+ membuat file baru bernama acl.conf dalam folder squid
  ```
  nano etc/squid/acl.conf
  ```
+ menambahkan isi acl.conf dengan
  ```
  acl WORK1 time TW 13:00-18:00
  ```
+ simpan
+ buka file squid.conf
+ menambahkan konfigurasi
  ```
  http_access allow WORK1
  ```


### 9.	Menjadwal bimbingan setiap hari Selasa-Kamis pukul 21.00 - 09.00 keesokan harinya (sampai Jumat jam 09.00)

cara :
+ buka file acl.conf dalam folder squid
  ```
  nano etc/squid/acl.conf
  ```
+ menambahkan isi acl.conf dengan
  ```
  acl DEADLINE1 time TWH 21:00-24:00
  acl DEADLINE2 time TWH 00:00-09:00
  ```
+ simpan
+ buka file squid.conf
+ menambahkan konfigurasi
  ```
  http_access allow DEADLINE1
  http_access allow DEADLINE2
  ```


### 10. Saat mengakses google.com, maka akan di redirect menuju monta.if.its.ac.id

+ buka file acl.conf, tambahkan
  ```
  acl lan src all
  acl google dstdomain .google.com
  ```
+ simpan
+ buka file squid.conf, tambahkan
  ```
  deny_info http://monta.if.its.ac.id lan
  http_reply_access deny google lan
  http_access allow all
  ```

### 11.	error page default squid

cara :
+ download file yang diinginkan
  ```
  wget 10.151.36.202/error403.tar.gz
  ```
+ melakukan pengekstrakkan dengan command `tar -xvf error403.tar.gz`
+ ketikkan
  ```
  mv /usr/share/squid/errors/English/ERR_ACCESS_DENIED usr/share/squid/errors/English/ERR_ACCESS_DENIDE
  cp -r ERR_ACCESS_DENIED /usr/share/squid/errors/English/ERR_ACCESS_DENIED
  ```


### 12. Ketika memakai proxy cukup dengan mengetikkan domain janganlupa-ta.yyy.pw dan memasukkan port 8080

cara :
+ menuliskan dalam kolom HTTP Proxy "janganlupa-ta.c07.pw" dengan port "8080"
+ masuk kedalam folder named.conf.local dalam malang, karena sebagai DNS Server
  ```
  nano etc/bind/named.conf.local
  ```
+ menambahkan
  ```
  zone "janganlupa-ta.b06.pw" {
  type master;
  file "/etc/bind/jarkom/janganlupa-ta.b06.pw";
  }
  ```
+ buka file janganlupa-ta.b06.pw
  ```
  nano etc/bind/jarkom/janganlupa-ta.b06.pw
  ```
+ menambahkan IP malang (10.151.83.58)
