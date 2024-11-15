---
title: 'How to Secure Webserver and Databases Instance using Security Groups'
date: 2024-11-13
permalink: /posts/2024/11/secure-instance-with-security-group/
tags:
  - Openstack
  - Documentation
  - Tutorial
---

![Secure-instance](/images/secure-instance.png)

---
Hallo Semuanya! Blog ini merupakan dokumentasi sekaligus panduan  mengenai cara menjaga Instance Webserver dan Instance Databases mengggunakan Security Group. Security group disini bertindak sebagai firewall yang menjaga dan membatasi port yang terbuka, sehingga keamanan dari kedua Instance meningkat.

---
## Latar Belakang
Dalam era digital yang semakin maju, layanan cloud computing telah menjadi solusi utama bagi perusahaan untuk mengelola dan menyimpan data serta aplikasi secara efisien. Namun, dengan meningkatnya penggunaan cloud, ancaman terhadap keamanan data dan infrastruktur juga ikut berkembang. Setiap instance atau server virtual yang berjalan di cloud berisiko menjadi target serangan siber seperti akses tidak sah, eksploitasi kerentanan, dan serangan Distributed Denial of Service (DDoS). Oleh karena itu, menjaga keamanan instance menjadi prioritas penting bagi perusahaan yang menggunakan infrastruktur cloud.

Salah satu cara efektif untuk melindungi instance dari ancaman keamanan adalah dengan menggunakan Security Group. Security Group berfungsi sebagai firewall virtual yang mengatur lalu lintas masuk dan keluar berdasarkan aturan yang telah ditentukan. Dengan Security Group, perusahaan dapat secara selektif membuka dan menutup akses ke port dan protokol tertentu, yang membantu mengurangi risiko serangan dan meningkatkan keamanan sistem secara keseluruhan. Penggunaan Security Group memungkinkan tim operasional untuk mengizinkan akses hanya dari alamat IP yang terpercaya dan memastikan hanya layanan yang diperlukan saja yang dapat diakses oleh pengguna eksternal.

---

Sebelum masuk, sebenernya security group itu apa sih?
---
## Security Group
Security Group adalah fitur dalam Openstack yang berfungsi sebagai firewall virtual untuk mengontrol lalu lintas masuk dan keluar dari instance atau resource dalam jaringan cloud. Security Group memungkinkan kita menentukan aturan akses yang spesifik berdasarkan alamat IP, port, dan protokol, sehingga dapat meningkatkan keamanan dari setiap instance yang digunakan.

## Cara Kerja Security Group
Security Group bekerja dengan menerapkan aturan yang ditentukan untuk mengontrol akses ke resource. Aturan ini menetapkan izin untuk lalu lintas yang diizinkan masuk atau keluar. Setiap aturan umumnya terdiri dari:

* Protocol: Menentukan protokol jaringan (misalnya TCP, UDP, ICMP).
* Port Range: Rentang port yang diperbolehkan untuk lalu lintas masuk atau keluar.
* Source/Destination IP: Alamat IP atau subnet yang diizinkan untuk mengakses port yang ditentukan.
* Direction (Ingress/Egress): Menentukan apakah aturan tersebut berlaku untuk lalu lintas masuk (ingress) atau keluar (Egress).

Setiap instance atau resource yang diproteksi dapat dihubungkan dengan satu atau beberapa Security Group, dan setiap Security Group bisa memiliki beberapa aturan. Semua aturan bersifat whitelist (izin akses), artinya hanya lalu lintas yang sesuai dengan aturan yang ditentukan yang diizinkan.

---
# Langkah Pengerjaan | Implementasi
Pada kesempatan kali ini, saya akan Launching Instance webserver yang akan dipasangkan nginx dan Instance Databases yang akan dipasangkan MySQL. Pertama-tama, pastikan sudah melakukan Instalasi atau deployment Openstack terlebih dahulu, jika belum bisa ikuti panduan ini [How To Deploy Openstack With Kolla-Ansible](https://gantengjanuar.github.io//posts/2024/11/deploy-openstack/)

1.Aktifkan admin kredensial dan aktifkan virtual env.
```
$ source /etc/kolla/admin-openrc.sh
$ source ~/kolla-venv/bin/activate
```

2.Buat image Ubuntu 20.04 LTS lalu verifikasi
```
$ wget https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img

$ openstack image create --disk-format qcow2 \
  --container-format bare --public \
  --file ./focal-server-cloudimg-amd64.img ubuntu-image

$ openstack image list
```

3.Buat external network serta subnetnya dan verifikasi.
```
$ openstack network create --share --external \
  --provider-physical-network physnet1 \
  --provider-network-type flat ganteng-external-net

$ openstack network list
$ openstack subnet create --network ganteng-external-net \
  --gateway 20.13.13.1 --no-dhcp \
  --subnet-range 20.13.13.0/24 ganteng-external-subnet

$ openstack subnet list
```

4.Buat Internal network serta subnetnya dan verifikasi.
```
$ openstack network create ganteng-internal-net

$ openstack subnet create --network ganteng-internal-net \
  --allocation-pool start=192.168.13.10,end=192.168.13.254 \
  --dns-nameserver 8.8.8.8 --gateway 192.168.13.1 \
  --subnet-range 192.168.13.0/24 ganteng-internal-subnet

$ openstack network list
$ openstack subnet list
```
5.Buat router dan verifikasi.
```
$ openstack router create ganteng-router
$ openstack router set --external-gateway ganteng-external-net ganteng-router
$ openstack router add subnet ganteng-router ganteng-internal-subnet

$ openstack router list
```

6.Buat security group yang mengamankan port yang terbuka
```
$ openstack security group create Database security
$ openstack security group create Webserver security
```

7.Atur rules untuk kedua security group tersebut
**Web Server Security Group (Nginx)**

| Nama Rule     | Protokol | Port      | Source            |
|---------------|----------|-----------|-------------------|
| HTTP (80)     | TCP      | 80        | 0.0.0.0/0 (Publik) |
| HTTPS (443)   | TCP      | 443       | 0.0.0.0/0 (Publik) |
| MySQL (3306)  | TCP      | 3306      | 192.168.13.40/24  |
| SSH (22)      | TCP      | 22        | 10.13.13.10/24    |
| ICMP (ping)   | ICMP     | All ICMP  | 192.168.13.40/24  |

**Database Security Group (MySQL)**

| Nama Rule     | Protokol | Port      | Source            |
|---------------|----------|-----------|-------------------|
| MySQL (3306)  | TCP      | 3306      | 192.168.13.47/24  |
| SSH (22)      | TCP      | 22        | 10.13.13.10/24    |
| ICMP (ping)   | ICMP     | All ICMP  | 192.168.13.47/24  |

> **Note:** Jika anda menggunakan database lain, sesuaikan port yang dibuka seperti contoh jika menggunakan (PostgreSQL) buka port **5432** . Untuk ip Source nya bisa disesuaikan tergantung IP anda sendiri.

8.Buat keypair dan verifikasi
```
$ openstack keypair create --public-key ~/.ssh/id_rsa.pub controller-key-gan

$ openstack keypair list
```

9.Buat flavor dan verifikasi.
```
$ openstack flavor create --ram 2048 --disk 8 --vcpus 1 --public mid-spec

$ openstack flavor list
```

10.Launching instance Web-instance
```
$ openstack server create --flavor mid-spec \
--image ubuntu-image \
--key-name controller-key-gan \
--security-group Web security \
--network ganteng-internal-net \
Web-instance
```

11.Pasang floating ip ke instance Web-security dan verifikasi.
```
$ openstack floating ip create --floating-ip-address 20.13.13.11 ganteng-external-net
$ openstack server add floating ip Web-instance 20.13.13.11

$ openstack server list
```

12.Coba akses instance Web-security
```
$ ssh -o 'PubkeyAcceptedKeyTypes +ssh-rsa' ubuntu@20.13.13.11
```

13.Launching instance DB-instance
```
$ openstack server create --flavor mid-spec \
--image ubuntu-image \
--key-name controller-key-gan \
--security-group Database security \
--network ganteng-internal-net \
DB-instance
```

14.Pasang floating ip ke instance DB dan verifikasi.
```
$ openstack floating ip create --floating-ip-address 20.13.13.200 ganteng-external-net
$ openstack server add floating ip DB-instance 20.13.13.200

$ openstack server list
```

15.Coba akses instance node2-gan.
```
$ ssh -o 'PubkeyAcceptedKeyTypes +ssh-rsa' ubuntu@20.13.13.200
```
---
Sejauh ini, saya sudah berhasil membuat Instance Databases dan Instance Websever yang dimana keduanya dipasangkan security group untuk meningkatkan keamanannya. Saya mengatur security group seperti itu supaya:
* Webserver bisa diakses Publik via HTTP dan HTTPS
* Kedua Instance bisa diakses melalui SSH lewat IP spesifik Controller, yang lain tidak akan bisa SSH
* Menghubungkan Webserver dan Database via port 3306 yang terfilter sehingga instance sudah siap digunakan.

## Langkah Pengujian
Untuk pengujian, saya akan memasang webserver Nginx dan Databases MySQL sederhana di kedua Instance lalu menghubungkannya, setelah itu menguji koneksi keduanya apakah sudah terhubung.

### Instalasi dan Konfigurasi Instance Web Server (Nginx + PHP-FPM)
1.Akses melalui SSH Instance Webserver
```
$ ssh -o 'PubkeyAcceptedKeyTypes +ssh-rsa' ubuntu@20.13.13.11
```

2.Instalasi Nginx dan PHP-FPM, dan mysqli
```
$ sudo apt update
$ sudo apt install -y nginx php-fpm && sudo apt install php7.4-mysqli
```
 
3.Edit file konfigurasi Nginx
```
$ sudo nano /etc/nginx/sites-available/default
```
```
server {
    listen 80;
    server_name _;

    root /var/www/html;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php-fpm.sock;
    }
}
```

4.Buat file PHP untuk pengujian koneksi ke MySQL, Buat file db_test.php di /var/www/html.
```
$ echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/index.php
```

5.Restart Nginx
```
$ sudo systemctl restart nginx
```

---
### Instalasi dan Konfigurasi MySQL di Instance Database

1.Akses Instance DB
```
$ ssh -o 'PubkeyAcceptedKeyTypes +ssh-rsa' ubuntu@20.13.13.200
```

2.Update dan instal MySQL
```
$ sudo apt update
$sudo apt install -y mysql-server
```

3.Konfigurasi MySQL untuk menerima koneksi dari Web Server. Edit file konfigurasi MySQL
```
$ sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

bind-address = 0.0.0.0
```

4.Restart MySQL
```
$ sudo systemctl restart mysql
```

5.Buat Database dan User untuk Aplikasi, masuk ke MySQL
```
$ sudo mysql
```
```
CREATE DATABASE db_test_gan;
CREATE USER 'ganteng'@'%' IDENTIFIED BY 'gan123';
GRANT ALL PRIVILEGES ON db_gan.* TO 'ganteng'@'%';
FLUSH PRIVILEGES;
```
> **Note:** nama DB, user, dan passwordnya bisa disesuaikan, ini hanya pengujian.

### Uji Koneksi dari Web Server ke Database

1.Edit file PHP di instance **Webserver** untuk koneksi ke MySQL, buat file baru db_test_gan.php di /var/www/html:
```
$ sudo nano /var/www/html/db_test_gan.php
```

2.Masukkan kode PHP untuk menghubungkan ke database
```
<?php
$conn = new mysqli("192.168.13.40", "ganteng", "gan123", "db_test_gan");

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "Connected successfully";
?>
```

3.Akses file PHP di browser
Akses http://20.13.13.11/db_test_gan.php di browser. Jika koneksi berhasil, akan ada pesan "Connected successfully".

---
## Langkah Verifikasi

- Pastikan Web Server dapat terhubung ke Database:
![Verif](/images/verif-koneksi.png)

- Pastikan bahwa hanya port yang diperlukan (HTTP, HTTPS, MySQL, SSH) yang terbuka di security group dan firewall, serta memastikan bahwa ping (ICMP) dapat dilakukan antar instance jika dibutuhkan untuk pengujian koneksi.

  ![DB-sec](/images/DB-sec.png)
  
  ![WEB-sec](/images/WEB-sec.png)

Sudah dipastikan, kedua instance aman karna port yang terbuka itu terbatas dan kedua instance tersebut hanya bisa diakses oleh IP Controller (admin).

---
## Referensi
* [Cara Menginstal nginx-mysql php di ubuntu 20.04](https://www.nusa.id/knowledge-base/cara-menginstal-linux-nginx-mysql-php-di-ubuntu-20-04/)

* [Cara koneksi Nginx-PHP dengan mySQL](https://www.youtube.com/watch?v=kALMxgVMZF4&t=1249s)




