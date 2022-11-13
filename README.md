# Jarkom-Modul-3-A09-2021
Laporan Resmi Praktikum Jarkom Modul 3 untuk kelompok A09.

# Anggota Kelompok
Anggota  | NRP
---------|-------
Arvel Gavrilla Raissananda | 05111940000040
Johnivan Aldo Sudiono | 05111940000051
Vincent Yonathan | 05111940000186

## Soal Jarkom-Modul-2 2021
- [Soal](https://docs.google.com/document/d/1hwuI5YpxiP-aboS7wGWPbaQeSOQl0HHVHLT3ws2BPUk/edit)

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

# --- No 8 ---

Loguetown digunakan sebagai client Proxy agar transaksi jual beli dapat terjamin keamanannya, juga untuk mencegah kebocoran data transaksi.
Pada Loguetown, proxy harus bisa diakses dengan nama jualbelikapal.yyy.com dengan port yang digunakan adalah 5000

### Langkah Penyelesaian : 

#### Water7

1. Edit file `/etc/squid/squid.conf` dengan menambahkan

```
    http_port 5000
    visible_hostname jualbelikapal.A09.com
    http_access allow all
```

2. kemudian jalankan command `service squid restart`

#### Skypie

3. Jalankan `apt-get update` dan `apt-get install lynx`

4. Aktifkan proxy dengan menjalankan command `export http_proxy="http://192.173.2.3:5000"`. Kemudian jalankan command `env | grep -i proxy` untuk mengecek apakah proxy sudah tersetting.

![image](https://user-images.githubusercontent.com/36225278/141270403-d7f48f1b-dfa6-4ff0-be01-fb468729584c.png)

# --- No 9 ---

Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy dipasang autentikasi user proxy dengan enkripsi MD5 dengan dua username, yaitu luffybelikapalyyy dengan password luffy_yyy dan zorobelikapalyyy dengan password zoro_yyy

### Langkah Penyelesaian : 

#### Water7

1. Jalankan `apt-get update` dan `apt-get install apache2-utils`

2. Kemudian jalankan command `htpasswd -cm /etc/squid/passwd luffybelikapalA09`, option `c` digunakan untuk membuat file baru, sedangkan `m` digunakan supaya enkripsinya menggunakan MD5. Setelah itu masukkan password `luffy_A09`.

3. Kemudian jalankan command `htpasswd -m /etc/squid/passwd zorobelikapalA09`. Setelah itu masukkan password `zoro_A09`.

4. Setelah itu edit file `/etc/squid/squid.conf` menjadi

```
    http_port 5000
    visible_hostname jualbelikapal.A09.com
    auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
    auth_param basic children 5
    auth_param basic realm Login
    auth_param basic credentialsttl 2 hours
    auth_param basic casesensitive on
    acl USERS proxy_auth REQUIRED
    http_access allow USERS
```

![image](https://user-images.githubusercontent.com/36225278/141270948-5d789656-d48a-40af-9eb9-94b48fdf78af.png)

5. kemudian jalankan command `service squid restart`

# --- No 10 ---

Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari Senin-Kamis pukul 07.00-11.00 dan setiap hari Selasa-Jum’at pukul 17.00-03.00 keesokan harinya (sampai Sabtu pukul 03.00)

### Langkah Penyelesaian : 

#### Water7

1. Buat file baru bernama `acl.conf` di folder squid dengan menjalankan command `nano /etc/squid/acl.conf` kemudian diisi dengan

```
acl time_done_1 time S 00:00-23:59
acl time_done_2 time MT 00:00-06:59
acl time_done_3 time M 11:01-23:59
acl time_done_4 time TWH 11:01-16:59
acl time_done_5 time WH 03:01-06:59
acl time_done_6 time F 03:01-16:59
acl time_done_7 time A 03:01-23:59
```

![image](https://user-images.githubusercontent.com/36225278/141271554-c84dcc46-20f7-4760-8b5f-f6ba95e846ee.png)

2. Setelah itu edit file `/etc/squid/squid.conf` menjadi

```
include /etc/squid/acl.conf
http_port 5000
visible_hostname jualbelikapal.A09.com

http_access deny time_done_1
http_access deny time_done_2
http_access deny time_done_3
http_access deny time_done_4
http_access deny time_done_5
http_access deny time_done_6
http_access deny time_done_7

auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Proxy
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED
```

![image](https://user-images.githubusercontent.com/36225278/141272068-60b52915-49c1-4f7d-964c-1ab15a27f5f3.png)

3. Kemudian jalankan command `service squid restart`

# --- No 11 ---

Agar transaksi bisa lebih fokus berjalan, maka dilakukan redirect website agar mudah mengingat website transaksi jual beli kapal. Setiap mengakses google.com, akan diredirect menuju super.franky.yyy.com dengan website yang sama pada soal shift modul 2. Web server super.franky.yyy.com berada pada node Skypie

### Langkah Penyelesaian

#### EniesLobby

1. Edit file `nano /etc/bind/named.conf.local` dengan mengisi

```
    zone "super.franky.A09.com" {
        type master;
        file "/etc/bind/sunnygo/super.franky.A09.com";
    };
    
    zone "3.173.192.in-addr.arpa" {
    type master;
    file "/etc/bind/kaizoku/3.173.192.in-addr.arpa";
    };

```

![image](https://user-images.githubusercontent.com/36225278/141273350-738a0118-2da3-474a-9b82-8c05cee3e114.png)

2. Kemudian buat folder sunnygo menggunakan command `mkdir /etc/bind/sunnygo`. 
3. Setelah itu jalankan command `cp /etc/bind/db.local /etc/bind/sunnygo/super.franky.A09.com`. 
4. Setelah itu edit file `nano /etc/bind/sunnygo/super.franky.A09.com` dengan mengisi

```
    $TTL    604800
    @       IN      SOA     super.franky.A09.com. root.super.franky.A09.com. (
                                2         ; Serial
                            604800         ; Refresh
                            86400         ; Retry
                            2419200         ; Expire
                            604800 )       ; Negative Cache TTL
    ;
    @       IN      NS      super.franky.A09.com.
    @       IN      A       192.173.3.69 
    www     IN      CNAME   super.franky.A09.com.
    @       IN      AAAA    ::1
```

![image](https://user-images.githubusercontent.com/36225278/141273839-03e5d10a-51d2-4d24-8d9e-cd44f0936302.png)

5. kemudian jalankan command `service bind9 restart`

#### Skypie

6. Install `apache2`, `php`, `libapache2-mod-php7.0`, `wget`, dan `unzip`.

7. Kemudian jalankan command `wget https://raw.githubusercontent.com/FeinardSlim/Praktikum-Modul-2-Jarkom/main/super.franky.zip` kemudian `unzip super.franky.zip`

8. Setelah itu, pindah ke directory `/etc/apache2/sites-available`.Kemudian copy file `000-default.conf` menjadi file `super.franky.A09.com.conf`

9. Lalu setting file `super.franky.A09.com.conf` agar memiliki line 

```
	DocumentRoot /var/www/super.franky.A09.com
      	ServerName super.franky.A09.com
        ServerAlias www.super.franky.A09.com
```

![image](https://user-images.githubusercontent.com/36225278/141274182-571c0f1f-0ebc-4d1e-ac39-83bf26954834.png)

10. Kemudian bikin directory baru dengan nama `super.franky.A09.com` pada `/var/www/` menggunakan command `mkdir /var/www/super.franky.A09.com`. lalu copy isi dari folder `super.franky` yang telah didownload ke `/var/www/super.franky.A09.com`.

11. Setelah itu jalankan command `a2ensite super.franky.A09.com` dan `service apache2 restart`

#### Water7

12. Setelah itu edit file `/etc/squid/squid.conf` menjadi

```
    include /etc/squid/acl.conf

    http_port 5000
    visible_hostname jualbelikapal.A09.com
    auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
    auth_param basic children 5
    auth_param basic realm Login
    auth_param basic credentialsttl 2 hours
    auth_param basic casesensitive on
    acl USERS proxy_auth REQUIRED
    acl google dstdomain google.com
    http_access deny google
    deny_info http://super.franky.A09.com/ google
    http_access allow USERS AVAILABLE_WORKING
    http_access deny all
```

![image](https://user-images.githubusercontent.com/36225278/141274770-ac74e7af-306a-4957-8e5b-c488bea18fa8.png)

13. Kemudian edit file `/etc/resolv.conf` menjadi

```
    nameserver 192.173.2.2
```

![image](https://user-images.githubusercontent.com/36225278/141274925-6df8a8ad-fbd4-47be-b32e-f5b187573c7f.png)

14. kemudian jalankan command `service squid restart`

# --- No 12 ---

Saatnya berlayar! Luffy dan Zoro akhirnya memutuskan untuk berlayar untuk mencari harta karun di super.franky.yyy.com. Tugas pencarian dibagi menjadi dua misi, Luffy bertugas untuk mendapatkan gambar (.png, .jpg), sedangkan Zoro mendapatkan sisanya. Karena Luffy orangnya sangat teliti untuk mencari harta karun, ketika ia berhasil mendapatkan gambar, ia mendapatkan gambar dan melihatnya dengan kecepatan 10 kbps

### Jawab

#### Water7

Jalankan command `nano /etc/squid/acl-bandwidth.conf` untuk membuat file `acl-bandwidth.conf` yang diisi dengan

```bash
   acl download url_regex -i \.png$ \.jpg$
   delay_pools 1
   delay_class 1 1
   delay_parameters 1 10000/10000
   delay_access 1 allow download
   delay_access 1 deny all
```
![image](https://user-images.githubusercontent.com/72689610/141350645-48789e6f-47c0-433e-bb8a-e15ef17aa061.png)

Kemudian tambahkan baris ini pada `/etc/squid/squid.conf`

```bash
    include /etc/squid/acl-bandwidth.conf
```

![image](https://user-images.githubusercontent.com/72689610/141350765-610a2aad-efcf-443b-8caa-596a406e57fb.png)

Setelah itu jalankan command `service squid restart`

Kemudian di `loguetown`, hal - hal yang perlu dilakukan adalah :
1. Cek apakah proxy sudah terpasang dengan menggunakan command `env | grep -i proxy`
2. Jika belum, maka proxy bisa dipasang dengan command `export http_proxy=http://192.173.2.3:5000`
3. Kemudian jalankan perintah `lynx google.com`, maka server akan diarahkan untuk menggunakan `www.super.franky.A09.com`. 
![image](https://user-images.githubusercontent.com/72689610/141358310-415ca4f2-e029-4726-a07e-717750c82211.png)

4. Isikan data username dengan `luffybelikapalA09` dan password dengan ```luffy_A09```.
![image](https://user-images.githubusercontent.com/72689610/141358392-77337808-936a-479b-a0ae-e4633da78715.png)
![image](https://user-images.githubusercontent.com/72689610/141358424-50bdeacc-990d-41df-bc9e-2eece7b74ac0.png)

5. Maka user akan diarahkan ke tampilan `www.super.franky.A09.com`
![image](https://user-images.githubusercontent.com/72689610/141358550-fc3ae59d-73a8-467e-9984-3cc67ef14e3c.png)

6. Masuk ke directory `public`, lalu pilih folder `images`. Maka akan muncul tampilan seperti gambar ke-3
![image](https://user-images.githubusercontent.com/72689610/141358691-e938d8f2-8fc7-4f8c-b0e6-3b36207d609c.png)
![image](https://user-images.githubusercontent.com/72689610/141358716-4942021b-3ce9-4eee-b71d-e079b5bf6f98.png)
![image](https://user-images.githubusercontent.com/72689610/141358737-a5203ba3-6708-4257-94dd-60a07107f189.png)

7. Untuk melakukan test kecepatan download Luffy, pilih salah satu file bertipe `jpg / png`, enter dan ketikkan `D` di keyboard untuk memulai proses download.
8. Untuk output kecepatan download Luffy, bisa dilihat sebagai berikut :
![image](https://user-images.githubusercontent.com/72689610/141359059-60726b31-e67c-48e7-917f-7ebb2933d8e5.png)

9. Jika proses download sudah selesai, akan ditampilkan output sebagai berikut :
![image](https://user-images.githubusercontent.com/72689610/141359505-8a85ea47-8b3c-41b5-a3e4-3a09a7620a36.png)

# --- No 13 ---

Sedangkan, Zoro yang sangat bersemangat untuk mencari harta karun, sehingga kecepatan kapal Zoro tidak dibatasi ketika sudah mendapatkan harta yang diinginkannya

### Langkah Penyelesaian

#### Water7

Edit file `acl-bandwidth.conf` menjadi

```
    acl download url_regex -i \.jpg$ \.png$

    auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd

    acl luffy proxy_auth luffybelikapalA09
    acl zoro proxy_auth zorobelikapalA09

    delay_pools 2
    delay_class 1 1
    delay_parameters 1 -1/-1
    delay_access 1 allow zoro
    delay_access 1 deny all
    delay_class 2 1
    delay_parameters 2 10000/10000
    delay_access 2 allow download
    delay_access 2 deny all
```

![image](https://user-images.githubusercontent.com/72689610/141359721-18d82b7d-c535-4e9e-b14b-a266457e81cb.png)

Setelah itu jalankan command `service squid restart`

Kemudian di `loguetown`, hal - hal yang perlu dilakukan adalah :
1. Cek apakah proxy sudah terpasang dengan menggunakan command `env | grep -i proxy`
2. Jika belum, maka proxy bisa dipasang dengan command `export http_proxy=http://192.173.2.3:5000`
3. Kemudian jalankan perintah `lynx google.com`, maka server akan diarahkan untuk menggunakan `www.super.franky.A09.com`. 
![image](https://user-images.githubusercontent.com/72689610/141358310-415ca4f2-e029-4726-a07e-717750c82211.png)

4. Isikan data username dengan `zorobelikapalA09` dan password dengan ```zoro_A09```.
![image](https://user-images.githubusercontent.com/72689610/141359935-c8bebc60-5f64-4b22-bbaa-f066c2a6bfe4.png)
![image](https://user-images.githubusercontent.com/72689610/141359989-5021e0ab-eb37-45b8-8590-4dfeb7523549.png)

5. Maka user akan diarahkan ke tampilan `www.super.franky.A09.com`
![image](https://user-images.githubusercontent.com/72689610/141358550-fc3ae59d-73a8-467e-9984-3cc67ef14e3c.png)

6. Masuk ke directory `public`, lalu pilih folder `images`. Maka akan muncul tampilan seperti gambar ke-3
![image](https://user-images.githubusercontent.com/72689610/141358691-e938d8f2-8fc7-4f8c-b0e6-3b36207d609c.png)
![image](https://user-images.githubusercontent.com/72689610/141358716-4942021b-3ce9-4eee-b71d-e079b5bf6f98.png)
![image](https://user-images.githubusercontent.com/72689610/141358737-a5203ba3-6708-4257-94dd-60a07107f189.png)

7. Untuk melakukan test kecepatan download Zoro, pilih salah satu file bertipe `jpg / png`, enter dan ketikkan `D` di keyboard untuk memulai proses download.
8. Untuk output kecepatan download Zoro dan hasil setelah download, bisa dilihat sebagai berikut :
![image](https://user-images.githubusercontent.com/72689610/141360151-120a8ba3-4a1b-491a-b7b4-7ab415c90397.png)

Disini kami tidak dapat melakukan screenshot kecepatan download dari Zoro, karena kecepatannya tidak diberikan limit sehingga proses download sudah selesai sebelum kami dapat melakukan screenshot.

---
## Kendala
Kendala yang dialami daripada pengerjaan modul praktikum Jarkom ini adalah :
- Belum terbiasa menggunakan proxy sehingga kagok saat mengerjakan
- Waktu yang sedikit untuk banyak soal
- Harus menggabungkan dengan materi modul sebelumnya
