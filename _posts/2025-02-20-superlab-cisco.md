---
title: 'Cisco: Superlab '
date: 2025-02-20
permalink: /posts/2025/02/cisco-superlab-1/
tags:
  - Cisco Packet Tracer
  - Networking
  - Superlab
---

![thumbnail](/images/thumbnail.png)

---

# CISCO CCNA  SUPERLAB - 1


## Logical Topology
![topologi](/images/l-topologi.png)

---

## Physical Topology
![topologi](/images/p-topologi.png)

---

## Ketentuan IP Address


### Router

| Interface | Core-RTR1       | Core-RTR2       | Core-RTR3       |
|-----------|----------------|----------------|----------------|
| Fa0/0     | 10.10.10.1/30  | 10.10.10.13/30 | 10.10.10.6/30  |
| Fa0/1     | 10.10.10.5/30  | 10.10.10.9/30  |                |
| Fa1/0     | 10.10.10.2/30  |                |                |

### Core-Switch

| Interface        | Core-Switch1      | Core-Switch2    |
|-----------       |----------------   |----------------|
| Fa0/1            | 10.10.10.14/30    | 10.10.10.10/30 |


### Server
| Server   | IP Address     |
|----------|--------------|
| Server1  | 192.168.90.2/24 |
| Server2  | 192.168.90.3/24 |

### VLAN
| VLAN   | IP Address       |
|--------|-----------------|
| VLAN10 | 192.168.10.0/24 |
| VLAN20 | 192.168.20.0/24 |
| VLAN30 | 192.168.30.0/24 |
| VLAN40 | 192.168.40.0/24 |
| VLAN50 | 192.168.50.0/24 |
| VLAN60 | 192.168.60.0/24 |
| VLAN70 | 192.168.70.0/24 |
| VLAN80 | 192.168.80.0/24 |
| VLAN90 | 192.168.90.0/24 |

### DNS Server
| Service    | IP Address     |
|------------|---------------|
| DNS Server | 192.168.90.2  |


Superlab ini dirancang untuk menguji pemahaman serta keterampilan dalam mengonfigurasi jaringan berbasis Cisco, dengan mencakup berbagai teknologi inti yang umum digunakan dalam lingkungan enterprise. Dalam lab ini, kita akan mengimplementasikan VLAN, trunking, routing OSPF, akses kontrol (ACL), serta konfigurasi server seperti DNS dan Web Server.

Topologi yang digunakan terdiri dari beberapa switch dan router yang dikonfigurasi untuk mendukung komunikasi antar VLAN serta akses ke berbagai layanan jaringan.

### Objective:

1.  Mengonfigurasi hostname dan enable secret pada semua router & switch.

2.  Membangun VLAN & Trunking di berbagai switch dengan SVI dan DHCP Server.

3.  Mengimplementasikan Router on Stick pada router.

4.  Mengonfigurasi OSPF agar seluruh router dapat berkomunikasi.

5.  Mengimplementasikan layanan DNS dan Web Server.

6.  Menggunakan Access List untuk membatasi akses ke server dari VLAN tertentu.

Pada dokumentasi ini, setiap bagian akan dijelaskan secara rinci, termasuk langkah-langkah konfigurasi dan verifikasi hasil untuk memastikan pengerjaan sudah sesuai.

>**Note**: Tiap step dibawah yang memerlukan untuk menjalankan **command**, itu semua harus dilakukan dalam **CLI**, Misal pada part 1 mengkonfigurasi hostname switch, buka switch lalu masuk ke bagian **CLI** baru jalankan Command nya 

Part 1 - Basic Configuration
============================

1\. Konfigurasi Hostname
------------------------

### Switch 1 sampai 4

Lakukan pada semua switch 1 sampai 4 dan beri nama sesuai angka:

```
enable
configure terminal
hostname SW1
```
> **Note** : Lakukan pada semua switch, dengan contoh switch satu namanya SW1, switch 2 SW2, dan seterusnya

### Core Switch 1 dan 2

Lakukan pada semua core switch 1 sampai 2 dan beri nama sesuai angka:

```
enable
configure terminal
hostname CSW1
```
> **Note** : Lakukan pada semua Core switch, dengan contoh core switch 1 namanya CSW1, core switch 2 CSW2, dan seterusnya

### Router 1 sampai 3

Lakukan pada semua router 1 sampai 3 dan beri nama sesuai angka:

```
enable
configure terminal
hostname RTR1
```

> **Note** : Lakukan pada semua Router, dengan contoh Router 1 namanya RTR1, Router 2 RTR2, dan seterusnya

2\. Konfigurasi Enable Secret
-----------------------------

Lakukan pada semua Switch, Router, dan Core Switch:

```
enable
configure terminal
enable secret cisco
exit
write memory
```
## Verifikasi Hostname Part-1

Jika sudah, nanti semua hostname akan berubah sesuai yang kita ganti, contoh menjadi `SW1` `SW2` dan seterusnya.

![v](/images/v-p1.png)

---

Part 2 - Configure VLAN & Trunking in Core-SW2 & SW4
====================================================

1\. Create VLAN & Set VLAN Name
-------------------------------

Lakukan di Core-Switch2:

```
enable
configure terminal
ip routing
```

Lakukan di Core-Switch2 dan Switch4:

```
vlan 10
name VLAN10
exit

vlan 20
name VLAN20
exit

vlan 30
name VLAN30
exit
```

2\. Konfigurasi Trunk di Interface yang Menghubungkan Core-SW2 dengan Switch4
-----------------------------------------------------------------------------

```
interface fastEthernet 0/6
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30
exit
```

3\. Konfigurasi Portfast di Semua Interface yang Terhubung ke PC
----------------------------------------------------------------

Lakukan di Core-Switch2 dan Switch4:

```
enable
configure terminal

interface fastEthernet 0/2
switchport mode access
switchport access vlan 10 # Tentukan VLAN sesuai kebutuhan
spanning-tree portfast
exit
```

4\. Konfigurasi SVI & DHCP Server di Core-SW2
---------------------------------------------

Lakukan di Core-Switch2:

### Konfigurasi SVI (IP Address untuk Tiap VLAN)

```
interface vlan 10
ip address 192.168.10.1 255.255.255.0
no shutdown

interface vlan 20
ip address 192.168.20.1 255.255.255.0
no shutdown

interface vlan 30
ip address 192.168.30.1 255.255.255.0
no shutdown
```

### Konfigurasi DHCP Server untuk Masing-Masing VLAN

```
ip dhcp excluded-address 192.168.10.1
ip dhcp excluded-address 192.168.20.1
ip dhcp excluded-address 192.168.30.1

ip dhcp pool VLAN10
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
dns-server 192.168.90.2

ip dhcp pool VLAN20
network 192.168.20.0 255.255.255.0
default-router 192.168.20.1
dns-server 192.168.90.2

ip dhcp pool VLAN30
network 192.168.30.0 255.255.255.0
default-router 192.168.30.1
dns-server 192.168.90.2
```

5\. Verifikasi dengan Ping Tiap PC - Part 2
----------------------------------
Hasil ping pc ke pc pada vlan10. vlan20. vlan30
![v](/images/v-p2.png)

> Note: Jika tidak bisa melakukan ping, gunakan command berikut untuk troubleshooting:

```
show vlan brief # Cek VLAN
show interfaces trunk # Cek Trunk ke Core-SW2
```


Part 3 - Configure VLAN & Trunking in Core-SW1, SW1, & SW2
----------------------------------------------------------

### Create VLAN & set the VLAN name as display

**Lakukan di Core-Switch1, Switch1, & Switch2:**

```
enable
configure terminal
ip routing #khusus core switch
vlan 40
name VLAN40
exit

vlan 50
name VLAN50
exit

vlan 60
name VLAN60
exit

vlan 70
name VLAN70
exit
```

**Konfigurasi Trunking di Core-SW1, SW1, dan SW2:**

```
interface fastEthernet 0/1
switchport mode trunk
switchport trunk allowed vlan 40,50,60,70
exit
```

### Configure portfast in all interfaces connected to PCs

**Lakukan di Switch1 & Switch2:**

```
interface fastEthernet 0/2
switchport mode access
switchport access vlan 40
spanning-tree portfast
exit
```

### Configure SVI & DHCP Server in Core-SW1 for all VLANs

**Lakukan di Core-Switch1:**

```
enable
configure terminal
ip routing

interface vlan 40
ip address 192.168.40.1 255.255.255.0
no shutdown
exit

interface vlan 50
ip address 192.168.50.1 255.255.255.0
no shutdown
exit

interface vlan 60
ip address 192.168.60.1 255.255.255.0
no shutdown
exit

interface vlan 70
ip address 192.168.70.1 255.255.255.0
no shutdown
exit

ip dhcp excluded-address 192.168.40.1 192.168.40.10
ip dhcp excluded-address 192.168.50.1 192.168.50.10
ip dhcp excluded-address 192.168.60.1 192.168.60.10
ip dhcp excluded-address 192.168.70.1 192.168.70.10

ip dhcp pool VLAN40
 network 192.168.40.0 255.255.255.0
 default-router 192.168.40.1
 dns-server 192.168.90.2
exit

ip dhcp pool VLAN50
 network 192.168.50.0 255.255.255.0
 default-router 192.168.50.1
 dns-server 192.168.90.2
exit

ip dhcp pool VLAN60
 network 192.168.60.0 255.255.255.0
 default-router 192.168.60.1
 dns-server 192.168.90.2
exit

ip dhcp pool VLAN70
 network 192.168.70.0 255.255.255.0
 default-router 192.168.70.1
 dns-server 192.168.90.2
exit
```

## Verifikasi dengan Ping tiap PC - Part  3
Hasil ping pc ke pc pada vlan40, vlan50, vlan60, vlan70:
![v](/images/v-p3.png)


Part 4 - Configure VLAN & Trunking in Core-RTR3 & SW3
-----------------------------------------------------

### Create VLAN & set the VLAN name as display

**Lakukan di Switch3:**

```
enable
configure terminal
vlan 80
name VLAN80
exit

vlan 90
name VLAN90
exit
```

### Configure portfast in all interfaces that connected to PCs

**Lakukan di Switch3:**

```
interface fastEthernet 0/2 #gunakan interface yang terhubung ke PC dan SVR
switchport mode access
switchport access vlan 80
spanning-tree portfast
exit
```

### Configure Trunking between Switch3 and Router3

**Lakukan di Switch3:**

```
interface fastEthernet 0/1 #gunakan interface yang menghubungkan switch dengan router
switchport mode trunk
switchport trunk allowed vlan 80,90
exit
```

### Configure Router on Stick & DHCP Server in Core-RTR3 for VLAN80

**Lakukan di Router3:**

```
enable
configure terminal
ip routing

interface fastEthernet 0/1.80
encapsulation dot1Q 80
ip address 192.168.80.1 255.255.255.0
exit

interface fastEthernet 0/1.90
encapsulation dot1Q 90
ip address 192.168.90.1 255.255.255.0
exit

ip dhcp excluded-address 192.168.80.1 192.168.80.10

ip dhcp pool VLAN80
network 192.168.80.0 255.255.255.0
default-router 192.168.80.1
dns-server 192.168.90.2
exit
```

---
### Configure IP Address on SVR1 & SVR2 manually

Atur IP secara manual:

1.  **SVR1** → 192.168.90.2

2.  **SVR2** → 192.168.90.3

Langkah:

-   Klik icon **SVR1** dan **SVR2**

-   Masuk ke **Desktop > IP Configuration**

-   Masukkan **IP Address**, **Subnet Mask**, **Gateway (192.168.90.1)**, dan **DNS Server (192.168.90.2)**

## Verifikasi Koneksi - Part 4

Pastikan semua PC dan server pada vlan80 dan vlan90 bisa **ping** satu sama lain:
![v](/images/v-p4.png)

Part 5 - Configure Point to Point Addressing Between Routers
------------------------------------------------------------

### Configure IP Address in all routers as display

**Lakukan di Router1:**

```
enable
configure terminal

# Interface ke RTR3
interface fastEthernet0/1
 ip address 10.10.10.5 255.255.255.252
 no shutdown
exit

# Interface ke RTR2
interface fastEthernet0/0
 ip address 10.10.10.1 255.255.255.252
 no shutdown
exit
```

**Lakukan di Router3:**

```
# Interface ke RTR1
interface fastEthernet0/0
 ip address 10.10.10.6 255.255.255.252
 no shutdown
exit
```

**Lakukan di Router2:**

```
# Interface ke RTR1
interface fastEthernet1/0
 ip address 10.10.10.2 255.255.255.252
 no shutdown
exit

# Interface ke Core-Switch2
interface fastEthernet0/1
 ip address 10.10.10.9 255.255.255.252
 no shutdown
exit

# Interface ke Core-Switch3
interface fastEthernet0/0
 ip address 10.10.10.13 255.255.255.252
 no shutdown
exit
```

**Lakukan di Core-Switch2:**

```
# Interface ke Router2
interface fastEthernet0/1
 no switchport
 ip address 10.10.10.10 255.255.255.252
 no shutdown
exit
```

**Lakukan di Core-Switch3:**

```
# Interface ke Router2
interface fastEthernet0/1
 no switchport
 ip address 10.10.10.14 255.255.255.252
 no shutdown
exit
```

## Verifikasi Koneksi - Part 5

Pastikan semua router bisa saling ping dan Core-Switch2 serta Core-Switch3 dapat melakukan ping ke Router2:
![v](/images/v-p5.png)

---

Part 6 - Configure OSPF in All of Routers
-----------------------------------------

### Configure OSPF in all of routers as display

**Lakukan di Router1:**

```
enable
configure terminal
router ospf 1
 network 10.10.10.0 0.0.0.3 area 0  # Koneksi ke Router 2
 network 10.10.10.4 0.0.0.3 area 0  # Koneksi ke Router 3
exit
```

**Lakukan di Router2:**

```
enable
configure terminal
router ospf 1
 network 10.10.10.0 0.0.0.3 area 0  # Koneksi ke Router 1
 network 10.10.10.12 0.0.0.3 area 0  # Koneksi ke Core-Switch1
 network 10.10.10.8 0.0.0.3 area 0  # Koneksi ke Core-Switch2
exit
```

**Lakukan di Router3:**

```
enable
configure terminal
router ospf 1
 network 10.10.10.4 0.0.0.3 area 0  # Koneksi ke Router 1
 network 192.168.80.0 0.0.0.255 area 0 # Koneksi ke switch 3
 network 192.168.90.0 0.0.0.255 area 0 # Koneksi ke switch 3
exit
```

**Lakukan di Core-Switch1:**

```
enable
configure terminal
ip routing
router ospf 1
 network 10.10.10.12 0.0.0.3 area 0  # Untuk koneksi ke Router 2
 network 192.168.40.0 0.0.0.255 area 0  # VLAN40
 network 192.168.50.0 0.0.0.255 area 0  # VLAN50
 network 192.168.60.0 0.0.0.255 area 0  # VLAN60
 network 192.168.70.0 0.0.0.255 area 0  # VLAN70
exit
```

**Lakukan di Core-Switch2:**

```
enable
configure terminal
ip routing
router ospf 1
 network 10.10.10.8 0.0.0.3 area 0 # Untuk koneksi ke Router 2
 network 192.168.10.0 0.0.0.255 area 0 # VLAN10
 network 192.168.20.0 0.0.0.255 area 0 # VLAN20
 network 192.168.30.0 0.0.0.255 area 0 # VLAN30
exit
```

## Verifikasi Ping semua PC - Part 6

Pastikan semua pc bisa saling melakukan **Ping**:
![v](/images/v-p6.png)

---

Part 7 - Configure DNS & Web Server
-----------------------------------

### Configure DNS Server in SVR1

-   Create A Record for agunacourse.com to 192.168.90.3

-   Create CNAME Record for www.agunacourse.com to agunacourse.com

**Lakukan di SVR1:**

1.  Klik Server (SVR1)

2.  Pilih Tab Services

3.  Pilih DNS

4.  Aktifkan DNS Service (On)

5.  Tambahkan A Record:

    -   Domain Name: agunacourse.com

    -   Address: 192.168.90.3

    -   Klik Add

6.  Tambahkan CNAME Record:

    -   Domain Name: www.agunacourse.com

    -   Alias: agunacourse.com

    -   Klik Add

7.  Simpan Konfigurasi

### Configure Web Server in SVR2 to display "Welcome to AgunaCourse.com"

**Lakukan di SVR2:**

1.  Klik Server (SVR2)

2.  Pilih Tab Services

3.  Pilih HTTP

4.  Pastikan HTTP Service dalam keadaan On

5.  Klik Tab index.html dan klik (edit)

6.  Ubah Konten Halaman Web untuk menampilkan kata "Welcome to AgunaCourse.com" contoh:

```
<html>
<head><title>AgunaCourse</title></head>
<body><h1>Welcome to AgunaCourse.com</h1></body>
</html>
```

**Simpan Perubahan**

---

## Verifikasi DNS & Web Server - Part 7

**Lakukan di semua PC:**

1.  Klik PC Client

2.  Pilih Tab Desktop

3.  Pilih Web Browser

4.  Ketik URL di Address Bar: `http://agunacourse.com`

5.  Tekan Enter

6.  Jika benar maka akan muncul tulisan: `Welcome to AgunaCourse.com`

![v](/images/v-p7.png)

---

Part 8 - Configure Access List
------------------------------

### Configure Access List in Core-RTR3 to prevent access from VLAN10 & VLAN60 to Server

**Lakukan di Router3:**

```
enable
configure terminal
access-list 101 deny ip 192.168.10.0 0.0.0.255 host 192.168.90.3 #Block VLAN 10
access-list 101 deny ip 192.168.60.0 0.0.0.255 host 192.168.90.3 #Block VLAN 60
access-list 101 permit ip any any #Traffic lain boleh masuk


#Terapkan ACL di Interface yang Mengarah ke Server
interface FastEthernet0/1
 ip access-group 101 out


#Terapkan ACL di Interface ke Switch
interface FastEthernet0/0
 ip access-group 101 in
```

## Verifikasi ACL - Part 8

Pastikan PC pada VLAN10 dan VLAN60 tidak bisa membuka web dan tidak bisa ping ke server.

PC pada VLAN10 dan VLAN60 ketika akses web:
![v](/images/v-p8.png)

PC pada VLAN10 dan VLAN60 ketika ping ke server:
![v](/images/v-p8-2.png)

## Hasil Akhir Keseluruhan

Dan yap, semua objective objective yang saya awal sudah saya eksekusi dengan sesuai, saya sudah berhasil melakukan:
1.  Mengonfigurasi hostname dan enable secret pada semua router & switch.

2.  Membangun VLAN & Trunking di berbagai switch dengan SVI dan DHCP Server.

3.  Mengimplementasikan Router on Stick pada router.

4.  Mengonfigurasi OSPF agar seluruh router dapat berkomunikasi.

5.  Mengimplementasikan layanan DNS dan Web Server.

6.  Menggunakan Access List untuk membatasi akses ke server dari VLAN tertentu.

Yang dimana hasil Verifikasi tiap part nya sudah saya berikan di akhir part masing masing.

## Kesimpulan

Setiap langkah dalam lab ini dirancang untuk memberikan pengalaman langsung dalam membangun dan mengelola jaringan berbasis Cisco. 
Dengan menerapkan konfigurasi ini, kita tidak hanya memahami konsep dasar networking tetapi juga mendapatkan keterampilan praktis yang dapat digunakan dalam lingkungan kerja nyata.
Semoga dokumentasi ini bermanfaat bagi siapa saja yang ingin memperdalam pemahaman mereka tentang konfigurasi jaringan menggunakan perangkat Cisco.

Terima kasih telah mengikuti dokumentasi ini, dan semoga sukses dalam perjalanan belajar jaringan Anda! 🚀💻