---
title: 'Memahami Kubernetes Object Dengan Mudah Menggunakan Analogi'
date: 2024-12-9
permalink: /posts/2024/12/kubernetes-objects/
tags:
  - Kubernetes
  - Analogy
  - Object
---

![kube-object](/images/OBJECT.png)

# **Memahami Object di Kubernetes dengan Mudah Menggunakan Analogi**

Halloooooo semuanya! 👋

Setelah kemarin kita membahas komponen utama Kubernetes menggunakan analogi, sekarang saya akan membahas mengenai Object API di Kubernetes, nah dengan mudah saya juga akan menggunakan analogi!!

sebelum itu, Object di Kubernetes itu apasih? dan ada apa saja?

# Kubernetes Object

![daftar-object](/images/daftar-object.png)
---
Menurut situs web kubernetes.io, Kubernetes Object adalah: 
> "Objek Kubernetes adalah entitas persisten dalam sistem Kubernetes. Kubernetes menggunakan entitas ini untuk merepresentasikan status cluster Anda"

Jadi Kubernetes Object adalah entitas persisten dalam sistem Kubernetes. Objek ini merepresentasikan "hal-hal" yang sedang berjalan di dalam cluster Anda, seperti aplikasi, beban kerja kontainer, konfigurasi jaringan, atau kebijakan cluster. Setiap objek adalah "rekaman niat" atau "Catatan berisi kondisi" Anda: setelah Anda membuat objek, Kubernetes akan terus bekerja untuk memastikan bahwa objek itu sesuai dengan keadaan yang Anda tentukan.

Setiap Kubernetes Object memiliki:
* Spesifikasi (spec): Mendefinisikan keadaan yang Anda inginkan.
* Status: Menunjukkan keadaan saat ini dari objek tersebut.

Secara simpel, **Kubernetes Object** adalah entitas yang dapat dikelola dalam cluster Kubernetes, masing-masing memiliki fungsi tertentu.

---
# API Object
![daftar-object](/images/api-objects.png)

## Pod

Pod adalah unit terkecil yang dapat dikelola dalam Kubernetes. Pod merepresentasikan satu atau lebih container (seperti Docker container) yang berjalan bersama-sama di dalam satu lingkungan komputasi. Pod dibuat untuk menjalankan aplikasi atau bagian dari aplikasi.

Dialam sebuah pod, biasanya ada **sidecar container** atau container pendukung yang bertugas membantu container utama, hal ini untuk memastikan pada sebuah pod aplikasi atau apapun yang berjalan didalamnya dapat bekerja dengan optimal. Pod memiliki beberapa ciri utama, yaitu: 
1. Unit Dasar Deploy Aplikasi: Satu Pod biasanya berisi satu aplikasi utama, tetapi bisa juga berisi beberapa container yang saling bekerja sama.
2. Berbagi Jaringan dan Penyimpanan: Container di dalam Pod berbagi IP address, namespace jaringan, dan storage (jika diatur).
3. Ephemeral (Sementara): Pod dirancang untuk bersifat sementara. Jika Pod gagal, Kubernetes Deployment atau ReplicaSet akan membuat Pod baru untuk menggantikannya.

Bayangkan Pod sebagai **meja** di restoran. Meja ini dapat digunakan oleh satu tamu (aplikasi utama) atau beberapa tamu (container pendukung) yang duduk bersama untuk bekerja sama. Mereka berbagi ruang yang sama (IP dan storage) agar lebih efisien. Jika meja rusak atau tidak bisa digunakan, pelayan (Deployment) akan menggantikannya dengan meja baru.

---
![pod](/images/pod.png)

### Contoh konfigurasi
![carbon](/images/pod-c.png)
---

## Service

Service adalah objek Kubernetes yang bertugas mengatur dan menyediakan akses jaringan ke sekelompok Pod. Service memungkinkan komunikasi antara Pod atau antara Pod dengan dunia luar tanpa perlu mengetahui lokasi (IP) Pod secara spesifik. Ini sangat penting karena Pod bersifat dinamis, sehingga IP-nya bisa berubah ketika Pod diganti atau dibuat ulang.

Service bertindak sebagai abstraksi yang stabil untuk mengakses aplikasi yang berjalan di dalam cluster, baik dari dalam cluster itu sendiri maupun dari luar.

Bayangkan cluster Kubernetes sebagai sekelompok pulau, di mana setiap pulau adalah Node, dan rumah-rumah (Pod) berada di atasnya. Penduduk di rumah (container) membutuhkan cara untuk saling berkomunikasi atau bahkan berinteraksi dengan dunia luar. **Service** adalah seperti jembatan penghubung yang menghubungkan pulau-pulau ini

---
![service](/images/service.png)

### Contoh Konfigurasi
![carbon](/images/service-c.png)
