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

Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.30 - [prefix IP].3.50

### Langkah Penyelesaian : 

#### Jipangu

1. Edit file `/etc/dhcp/dhcpd.conf` dengan menambahkan:

```
    subnet 192.173.3.0 netmask 255.255.255.0 {
        range 192.173.3.30 192.173.3.50;
        option routers 192.173.3.1;
        option broadcast-address 192.173.3.255;
        option domain-name-servers 192.173.2.2;
        default-lease-time 720;
        max-lease-time 7200;
    }
```

![image](https://user-images.githubusercontent.com/36225278/141261264-52c90f00-6b27-4acc-839b-dd25e6b7096f.png)

2. Lalu jalankan command `service isc-dhcp-server restart`

# --- No 5 ---

Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut.

### Langkah Penyelesaian : 

#### EniesLobby

1. Edit file `/etc/bind/named.conf.options` dengan menambahkan

```
    forwarders {
        "IP nameserver dari Foosha";
    };

    allow-query{any;};
```

dan mengkomen bagian

```
    // dnssec-validation auto;
```

![image](https://user-images.githubusercontent.com/36225278/141263657-1796e93d-b23d-4394-ace8-a6bafe9a7943.png)


2. kemudian jalankan command `service bind9 restart`

#### Jipangu

3. Edit file `/etc/dhcp/dhcpd.conf` dengan menambahkan baris `option domain-name-servers "IP EniesLobby"` pada `subnet 192.173.1.0` dan `subnet 192.173.3.0`


# --- No 6 ---

Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit.

### Langkah Penyelesaian : 

#### Jipangu

Edit file `/etc/dhcp/dhcpd.conf` dengan menambahkan baris ini pada `subnet 192.173.1.0`

```
        default-lease-time 360;
        max-lease-time 7200;
```


dan menambahkan baris ini pada `subnet 192.173.3.0`

```
        default-lease-time 720;
        max-lease-time 7200;
```

![image](https://user-images.githubusercontent.com/36225278/141264400-aa271498-2d14-4678-b9db-3c54092fd6dc.png)

# --- No 7 ---

Luffy dan Zoro berencana menjadikan Skypie sebagai server untuk jual beli kapal yang dimilikinya dengan alamat IP yang tetap dengan IP [prefix IP].3.69

### Langkah Penyelesaian : 

#### Jipangu

1. Edit file `/etc/dhcp/dhcpd.conf` dengan menambahkan baris ini

```
    host Skypie {
        hardware ethernet "hardware address Skypie";
        fixed-address 192.173.3.69;
    }
```
![image](https://user-images.githubusercontent.com/36225278/141266005-ab6c8d05-9f23-4f5c-848f-b5c006e87e73.png)

2. Lalu jalankan command `service isc-dhcp-server restart`

#### Skypie

3. Kemudian tambahkan `hwaddress ether "hardware address Skypie"` pada `/etc/network/interfaces` agar hwaddress tidak berubah-ubah ketika project direstart atau diexport

![image](https://user-images.githubusercontent.com/36225278/141269665-a4b88db9-4d06-4b6d-96d8-c08e5ec7bdc2.png)

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

---
## Kendala
Kendala yang dialami daripada pengerjaan modul praktikum Jarkom ini adalah :
- Belum terbiasa menggunakan proxy sehingga kagok saat mengerjakan
- Waktu yang sedikit untuk banyak soal
- Harus menggabungkan dengan materi modul sebelumnya
