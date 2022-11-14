# Jarkom-Modul-3-E04-2022
Laporan Resmi Praktikum Jarkom Modul 3 untuk kelompok E04.

# Anggota Kelompok
Anggota  | NRP
---------|-------
Muhammad Afif Dwi Ardiansyah | 5025201212
Muhammad Rafif Fadhil Naufal | 5025201273
M Labib Alfaraby | 5025201083

## Soal Jarkom-Modul-3 2022
- [Soal](https://docs.google.com/document/d/1asm7lgnTJxr17DxsE_McdUimPsRjesi6ZrHRpmXPZ4s/edit)

## Pendahuluan
Luffy yang sudah menjadi Raja Bajak Laut ingin mengembangkan daerah kekuasaannya dengan membuat peta seperti berikut: <br>
![image](https://user-images.githubusercontent.com/36225278/141254625-b7547563-2b49-43ca-8941-d6618b795894.png)

### Edit Knofigurasi Network

#### Foosha
```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 192.173.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 192.173.2.1
	netmask 255.255.255.0

auto eth3
iface eth3 inet static
	address 192.173.3.1
	netmask 255.255.255.0
```

#### EniesLobby
```
auto eth0
iface eth0 inet static
	address 192.173.2.2
	netmask 255.255.255.0
	gateway 192.173.2.1
```
#### Water7
```
auto eth0
iface eth0 inet static
	address 192.173.2.3
	netmask 255.255.255.0
	gateway 192.173.2.1
```

### Jipangu
```
auto eth0
iface eth0 inet static
	address 192.173.2.4
	netmask 255.255.255.0
	gateway 192.173.2.1
```

#### Loguetown, Alabasta, Totoland, Skypie 
```
auto eth0
iface eth0 inet dhcp
```

# --- No 1 ---

Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server

### Jawab

### Langkah Penyelesaian : 

#### Foosha

1. Menjalankan command `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.173.0.0/16` yang digunakan supaya dapat terhubung ke jaringan luar pada router `Foosha`

2. Setelah itu pada EniesLobby, Water7, Jipangu dijalankan command `echo "nameserver 192.168.122.1" > /etc/resolv.conf` untuk setting IP DNS agar dapat terhubung ke jaringan luar.

#### EniesLobby

3. pada EniesLobby jalankan command `apt-get update` dan `apt-get install bind9 -y` untuk menginstall bind9

#### Jipangu

4. Pada Jipangu jalankan command `apt-get update` dan `apt-get install isc-dhcp-server -y` untuk menginstall isc-dhcp-server

5. Kemudian setting `INTERFACES` yang digunakan oleh Jipangu pada file `/etc/default/isc-dhcp-server` dengan menambahkan `eth0`
![image](https://user-images.githubusercontent.com/36225278/141259265-4c8454e3-65eb-4d26-83b6-44f35e1237f4.png)


#### Water7

6. pada Water7 jalankan command `apt-get update` dan `apt-get install squid -y` untuk menginstall squid

# --- No 2 ---

Foosha sebagai DHCP Relay

### Langkah Penyelesaian : 

#### Foosha

1. Pada Foosha jalankan command `apt-get update` dan `apt-get install isc-dhcp-relay -y` untuk menginstall isc-dhcp-relay

2. Kemudian edit file `/etc/default/isc-dhcp-relay` dengan menambahkan `SERVER = "IP Jipangu"` dan `INTERFACES = "eth1 eth2 eth3"`
```
# What servers should the DHCP relay forward requests to?
SERVERS="192.173.2.4"

# On what interfaces should the DHCP relay (dhrelay) serve DHCP requests?
INTERFACES="eth1 eth2 eth3"
```

![image](https://user-images.githubusercontent.com/36225278/141260709-71770623-ed1f-4114-87e0-c85334fed17c.png)

3. Lalu jalankan command `service isc-dhcp-relay restart`

# --- No 3 ---

Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.20 - [prefix IP].1.99 dan [prefix IP].1.150 - [prefix IP].1.169

### Langkah Penyelesaian : 

#### Jipangu

1. Edit file `/etc/dhcp/dhcpd.conf` dengan menambahkan:

```
    subnet 192.173.1.0 netmask 255.255.255.0 {
        range 192.173.1.20 192.173.1.99;
        range 192.173.1.150 192.173.1.169;
        option routers 192.173.1.1;
        option broadcast-address 192.173.1.255;
        option domain-name-servers 192.173.2.2;
        default-lease-time 360;
        max-lease-time 7200;
    }
```
![image](https://user-images.githubusercontent.com/36225278/141260955-fbe98610-cfba-4540-8897-fbaf5f2907d0.png)

2. Lalu jalankan command `service isc-dhcp-server restart`

# --- No 4 ---

Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.10 - [prefix IP].3.30 dan [prefix IP].3.60 - [prefix IP].3.85

### Langkah Penyelesaian : 

#### Westalis

1. Edit file `/etc/dhcp/dhcpd.conf` dengan menambahkan:

```
echo ‘
subnet 192.194.3.0 netmask 255.255.255.0 {
    range 192.194.3.10 192.194.3.30;
    range 192.194.3.60 192.194.3.85;
    option routers 192.194.3.1;
    option broadcast-address 192.194.3.255;
    option domain-name-servers 192.194.2.2;
    default-lease-time 600;
    max-lease-time 6900;
} ‘ >> /etc/dhcp/dhcpd.conf
service isc-dhcp-server restart
service  isc-dhcp-server status

```
![image](https://user-images.githubusercontent.com/96496752/201639715-30e780d7-0f49-4200-8cc7-9cb6f26373b5.png)

2. Lalu jalankan command `service isc-dhcp-server restart` dan `service  isc-dhcp-server status`

Hasilnya : 

![image](https://user-images.githubusercontent.com/96496752/201639896-a83efed8-e130-4cdf-921f-6fedf77c13c7.png)

# --- No 5 ---

Client mendapatkan DNS dari WISE dan client dapat terhubung dengan internet melalui DNS tersebut.

### Langkah Penyelesaian : 

#### Wise

1. Edit file `/etc/bind/named.conf.options` dengan menambahkan

```
echo '
options {
        directory "/var/cache/bind";

        forwarders {
             192.168.122.1;
        };

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };

      allow-query{any;};
};
' > /etc/bind/named.conf.options

service bind9 restart
service bind9 restart

```
![image](https://user-images.githubusercontent.com/96496752/201640748-2b14491e-3597-4b59-9e3e-4ab2692273d7.png)


2. kemudian jalankan command `service bind9 restart`

#### Westalis

3. Edit file `/etc/dhcp/dhcpd.conf` dengan menambahkan baris `option domain-name-servers "IP Wise"` pada `subnet 192.194.1.0` dan `subnet 192.194.3.0`, karena sudah melakukannya di nomor 3 dan 4, maka tidak perlu ditambahkan kembali

#### SSS
4. Check dengan menjalankan command `cat /etc/resolv.conf`

Hasilnya : 

![image](https://user-images.githubusercontent.com/96496752/201640524-af9b5fa0-5a45-4c2d-b5fd-ab6e81c5ffab.png)

# --- No 6 ---

Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 5 menit sedangkan pada client yang melalui Switch3 selama 10 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 115 menit. 

### Langkah Penyelesaian : 

#### Westalis

Edit file `/etc/dhcp/dhcpd.conf` dengan menambahkan baris ini pada `subnet 192.194.1.0`

```
        default-lease-time 300;
        max-lease-time 6900;
```


dan menambahkan baris ini pada `subnet 192.194.3.0`

```
        default-lease-time 600;
        max-lease-time 6900;
```
sama seperti sebelumnya, karena sudah sekaligus melakukan konfigurasi pada no 3 dan 4, tidak perlu mengubahnya lagi.

# --- No 7 ---

Loid dan Franky berencana menjadikan Eden sebagai server untuk pertukaran informasi dengan alamat IP yang tetap dengan IP [prefix IP].3.13
### Langkah Penyelesaian : 

#### Westalis
1. Edit file `/etc/dhcp/dhcpd.conf` dengan menambahkan baris ini pada file nomor 7

```
echo '
host Eden {
     hardware ethernet b2:1f:f3:c4:0a:75;
     fixed-address 192.194.3.13;
}
' >> /etc/dhcp/dhcpd.conf

```

![image](https://user-images.githubusercontent.com/96496752/201641395-f97ab939-538c-47ba-86c6-13a482e33ef6.png)

2. Lalu jalankan command `service isc-dhcp-server restart`

#### Eden

3. Kemudian tambahkan `hwaddress ether "hardware address Skypie"` pada `/etc/network/interfaces` agar hwaddress tidak berubah-ubah ketika project direstart atau diexport

Berikut Hasilnya : 

![Screenshot 2022-11-14 at 17-25-46 shift-3](https://user-images.githubusercontent.com/96496752/201641593-e3f0553f-069d-407c-be33-635f14db3178.png)


# --- PROXY ---
SSS, Garden, dan Eden digunakan sebagai client Proxy agar pertukaran informasi dapat terjamin keamanannya, juga untuk mencegah kebocoran data.

Pada Proxy Server di Berlint, Loid berencana untuk mengatur bagaimana Client dapat mengakses internet. Artinya setiap client harus menggunakan Berlint sebagai HTTP & HTTPS proxy. Adapun kriteria pengaturannya adalah sebagai berikut:


# --- No 1 (Proxy) ---

Client hanya dapat mengakses internet diluar (selain) hari & jam kerja (senin-jumat 08.00 - 17.00) dan hari libur (dapat mengakses 24 jam penuh)

### Langkah Penyelesaian : 

#### Berlint

1. Isi file `/etc/squid/acl.conf` dengan

```
    acl WORKING time MTWHF 08:00-17:00
    acl WEEKEND time AS 00:00-24:00
```
2. Lalu isi file `/etc/squid/squid.conf` dengan
```
    include /etc/squid/acl.conf
    
    http_port 5000
    visible_hostname Berlint
    http_access allow WEEKEND
    http_access deny WORKING
    http_access allow all
```
4. Kemudian restart squid dengan
```
   service squid restart
```

#### Client
1. Install lynx
```
   apt-get update
   apt-get install lynx
```
3. Sambungkan Proxy
```
   export http_proxy="http://192.194.2.3:5000"
   env | grep -i proxy
```
5. Test di hari senin jam kerja:
6. Test di hari minggu:

# --- No 2 (Proxy) ---
Adapun pada hari dan jam kerja sesuai nomor (1), client hanya dapat mengakses domain loid-work.com dan franky-work.com (IP tujuan domain dibebaskan)

### Langkah Penyelesaian : 

#### Berlint

1. Isi file `/etc/squid/restrict-sites.acl` dengan
```
   loid-work.com
   franky-work.com
```

2. Isi file `/etc/squid/squid.conf` dengan
```
    include /etc/squid/acl.conf
    
    http_port 5000
    visible_hostname Berlint
    
    acl WELCOME dstdomain "/etc/squid/restrict-sites.acl"
    http_access deny WEEKEND WELCOME
    http_access allow WEEKEND
    http_access allow WORKING WELCOME
    http_access deny WORKING
    http_access deny all WELCOME
    http_access allow all
```

3. Kemudian restart squid dengan
```
service squid restart
```

#### Client
1. Jam kerja request google.com:
2. Jam kerja request loid-work.com:
3. Jam kerja request franky-work.com:
4. Weekend request loid-work.com:
---

# --- No 4 (Proxy) ---

Agar menghemat penggunaan, akses internet dibatasi dengan kecepatan maksimum 128 Kbps pada setiap host (Kbps = kilobit per second; lakukan pengecekan pada tiap host, ketika 2 host akses internet pada saat bersamaan, keduanya mendapatkan speed maksimal yaitu 128 Kbps)

### Langkah Penyelesaian : 

#### Berlint

1. Isi file `/etc/squid/acl-bandwidth.conf` dengan
```
   delay_pools 1
   delay_class 1 1
   delay_access 1 allow all
   delay_parameters 1 8000/64000
```
2. Lalu isi file `/etc/squid/squid.conf` dengan
```
   include /etc/squid/acl.conf
   include /etc/squid/acl-bandwidth.conf
   
   http_port 5000
   visible_hostname Berlint
   
   acl WELCOME dstdomain "/etc/squid/restrict-sites.acl"
   http_access deny WEEKEND WELCOME
   http_access allow WEEKEND
   http_access allow WORKING WELCOME
   http_access deny WORKING
   http_access deny all WELCOME
   http_access allow all
```
3. Kemudian restart squid dengan
```
service squid restart
```

#### Client

1. Install speedtest-cli dengan memutuskan proxy terlebih dahulu
```
unset http_proxy
apt-get update
apt install speedtest-cli
```
2. Jalankan command tersebut untuk meng-allow speetest nya
```
export PYTHONHTTPSVERIFY=0
```
3. Jalankan speedtest hasilnya berikut (tanpa tersambung proxy):

<img width="466" alt="image" src="https://user-images.githubusercontent.com/87472849/201636397-501c4d9d-3dfe-4b15-9280-d8f7fc522663.png">

4. Jalankan speedtest hasilnya berikut (tersambung proxy):

<img width="461" alt="image" src="https://user-images.githubusercontent.com/87472849/201636528-dc5c7ad7-2a2d-4d44-930f-b5d479961b8c.png">

# --- No 5 (Proxy) ---

Setelah diterapkan, ternyata peraturan nomor (4) mengganggu produktifitas saat hari kerja, dengan demikian pembatasan kecepatan hanya diberlakukan untuk pengaksesan internet pada hari libur

### Langkah Penyelesaian : 

#### Berlint
1. Ubah isi file ` /etc/squid/acl-bandwidth.conf` dengan baris berikut
```
   include /etc/squid/acl.conf
   
   delay_pools 1
   delay_class 1 1
   delay_access 1 allow WEEKEND
   delay_parameters 1 8000/64000
```
2. Kemudian restart squid dengan
```
service squid restart
```

#### Client
1. Speedtest di weekend:

<img width="466" alt="image" src="https://user-images.githubusercontent.com/87472849/201636870-74cdf822-33aa-43b7-b2c5-fbf9a1ea29b4.png">

3. Speedtest di hari kerja:

<img width="489" alt="image" src="https://user-images.githubusercontent.com/87472849/201636777-21b93eff-03c4-4e91-9f3c-273da4f6dfde.png">

## Kendala
Kendala yang dialami daripada pengerjaan modul praktikum Jarkom ini adalah :
- Belum terbiasa menggunakan proxy sehingga kagok saat mengerjakan
- Waktu yang sedikit untuk banyak soal
- Harus menggabungkan dengan materi modul sebelumnya
