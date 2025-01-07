---
title: 'Memahami Kubernetes Object Dengan Mudah Menggunakan Analogi'
date: 2024-12-9
permalink: /posts/2024/12/kubernetes-objects/
tags:
  - Kubernetes
  - Analogy
  - Object
---

![kube-object](/images/object-analogy.png)

# **Memahami Object di Kubernetes dengan Mudah Menggunakan Analogi**

Halloooooo semuanya! 👋

Setelah kemarin kita membahas komponen utama Kubernetes menggunakan analogi, sekarang saya akan membahas mengenai Object API di Kubernetes, nah dengan mudah saya juga akan menggunakan analogi!!

sebelum itu, Object di Kubernetes itu apasih? dan ada apa saja?

# Kubernetes Object

![daftar-object](/images/daftar-object.png)
---
Menurut situs web kubernetes.io, Kubernetes Object adalah: 
> "Objek Kubernetes adalah entitas persisten dalam sistem Kubernetes. Kubernetes menggunakan entitas ini untuk merepresentasikan status cluster Anda"

Jadi Kubernetes Object adalah entitas persisten dalam sistem Kubernetes. Objek ini merepresentasikan "hal-hal" yang sedang berjalan di dalam cluster, seperti aplikasi, beban kerja kontainer, konfigurasi jaringan, atau kebijakan cluster. Setiap objek adalah "rekaman niat" atau "Catatan berisi kondisi".setelah kita membuat objek, Kubernetes akan terus bekerja untuk memastikan bahwa objek itu sesuai dengan keadaan yang ditentukan.

Setiap Kubernetes Object memiliki:
* Spesifikasi (spec): Mendefinisikan keadaan yang diinginkan.
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

Service adalah objek Kubernetes yang bertugas mengatur dan menyediakan akses jaringan ke sekelompok Pod. Service memungkinkan komunikasi antara Pod atau antara Pod dengan dunia luar tanpa perlu mengetahui lokasi (IP) Pod secara spesifik. Ini sangat penting karena Pod bersifat dinamis, sehingga IP-nya bisa berubah ketika Pod diganti atau dibuat ulang. Service bertindak sebagai abstraksi yang stabil untuk mengakses aplikasi yang berjalan di dalam cluster, baik dari dalam cluster itu sendiri maupun dari luar.

Ada tiga tipe service, yaitu:
* **ClusterIP**: adalah tipe Service yang paling sering digunakan secara default. Tipe ini membuat aplikasi dapat diakses hanya dari dalam cluster. ClusterIP sangat berguna untuk komunikasi internal antar aplikasi, seperti frontend yang ingin berbicara dengan backend.

* **NodePort** adalah tipe Service yang membuka akses aplikasi melalui port tetap pada setiap Node dalam cluster. Dengan NodePort, pengguna dapat mengakses aplikasi dari luar cluster menggunakan alamat IP Node dan port tertentu.

* **LoadBalancer** adalah tipe Service yang secara otomatis membuat load balancer eksternal menggunakan fitur yang disediakan oleh cloud provider. Service ini mengarahkan trafik dari load balancer eksternal ke NodePort dan ClusterIP yang sesuai.

Bayangkan cluster Kubernetes sebagai sekelompok pulau, di mana setiap pulau adalah Node, dan rumah-rumah (Pod) berada di atasnya. Penduduk di rumah (container) membutuhkan cara untuk saling berkomunikasi atau bahkan berinteraksi dengan dunia luar. **Service** adalah seperti jembatan penghubung yang menghubungkan pulau-pulau ini

---
![service](/images/service.png)

### Contoh Konfigurasi
![carbon](/images/service-c.png)

## Ingress

Ingress dalam Kubernetes adalah komponen yang mengatur lalu lintas masuk (traffic) ke dalam aplikasi yang berjalan di cluster. Dengan Ingress, Anda bisa menentukan aturan bagaimana pengguna atau klien eksternal dapat mengakses aplikasi Anda, seperti menentukan jalur URL tertentu atau mengatur domain yang digunakan. Ingress biasanya digunakan untuk mengarahkan traffic HTTP dan HTTPS dengan lebih fleksibel dibandingkan Service biasa.

Bayangkan sebuah perusahaan besar dengan berbagai departemen di dalamnya, seperti HR, IT, dan Keuangan. Semua orang yang ingin mengakses departemen ini harus masuk melalui gerbang utama. Ingress dalam analogi ini adalah **gerbang utama** perusahaan. Gerbang ini memiliki aturan jelas dimana orang yang datang harus diarahkan ke departemen tertentu berdasarkan tujuan mereka. Misalnya, permintaan ke "hr.perusahaan.com" akan diarahkan ke departemen HR

Pada ingress, ada yang disebut sebagai **ingress controller**. Agar gerbang utama dapat bekerja dengan baik, dibutuhkan seseorang yang memastikan setiap tamu diarahkan ke tempat yang tepat. Di sinilah Ingress Controller berperan. Ingress Controller adalah **resepsionis perusahaan** yang menerima setiap tamu di gerbang utama, membaca informasi tujuan mereka, dan memastikan mereka sampai ke departemen yang benar.

---
![service](/images/ingress.png)

### Contoh Konfigurasi
![service](/images/ingress-c.png)
---

## Volume
Dalam Kubernetes, **Volume** adalah mekanisme untuk menyimpan data yang digunakan oleh Pod. Tidak seperti penyimpanan dalam kontainer yang bersifat sementara (ephemeral), Volume memastikan data tetap tersedia meskipun Pod atau kontainer dihentikan atau diganti. Volume ini dapat digunakan untuk berbagi data antar kontainer dalam satu Pod atau menyimpan data secara persisten.

Bayangkan Volume sebagai gudang pada sebuah mansion. **Gudang** adalah tempat penyimpanan yang tidak terpengaruh oleh aktivitas yang ada (kontainer). Meski ada yang keluar atau berganti, data di gudang tetap ada dan dapat diakses oleh siapa pun yang membutuhkannya. Sama seperti itu, Volume di Kubernetes memastikan data tetap tersedia meskipun Pod berhenti bekerja atau diganti.

---
![service](/images/volume.png)

### Contoh Konfigurasi
![carbon](/images/volume-c.png)

## Persistent Volume (PV) & Persistent Volume Claim (PVC)
Persistent Volume (PV) dan Persistent Volume Claim (PVC) adalah dua komponen dalam Kubernetes yang bekerja bersama untuk menyediakan penyimpanan persisten bagi Pod. PV adalah sumber daya penyimpanan yang disediakan oleh administrator cluster, sementara PVC adalah permintaan untuk sumber daya tersebut oleh pengguna.

Bayangkan sebuah PV sebagai gudang besar yang dimiliki oleh sebuah perusahaan. Gudang ini dapat menampung banyak barang dan menyediakan ruang untuk berbagai kebutuhan. Namun, agar dapat menggunakan gudang ini, tim atau departemen tertentu perlu mengajukan permintaan ruang dengan menentukan ukuran dan kebutuhan mereka. Permintaan ini adalah PVC, yang merupakan sebagian dari gudang yang dikhususkan untuk mereka.

---
![pv](/images/pv.png)

---
### Contoh Konfigurasi PV dan PVC
![pv](/images/pvc.png)

## Namespace
Namespace adalah mekanisme Kubernetes untuk mengisolasi dan mengelola resource dalam suatu cluster. Namespace memungkinkan kita membagi cluster besar menjadi bagian-bagian kecil yang lebih terorganisir dan terisolasi. Dengan Namespace, Anda kita memastikan bahwa resource tidak saling tumpang tindih dan memberikan kontrol lebih baik pada pengaturan akses.

Bayangkan sebuah gedung perkantoran besar yang memiliki banyak ruangan, di mana setiap ruangan digunakan oleh tim atau departemen yang berbeda. Disini namespace adalah **Ruangan** tersebut, ruangan ruangan yang ada mengisolasi segala sesuatu yang ada dilamnya sehingga membuat gedung menjadi lebih terorganisir. Seperti ruangan produksi yang berisi mesin mesin produksi dan ruangan arsip yang berisi dokumen dokumen.

---
![namespace](/images/ns.png)

### Contoh Konfigurasi
![ns](/images/ns-c.png)

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: example-namespace
```