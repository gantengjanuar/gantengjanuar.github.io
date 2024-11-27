---
title: 'Memahami Kubernetes Dengan Mudah Menggunakan Analogi'
date: 2024-11-27
permalink: /posts/2024/11/kubernetes-architecture/
tags:
  - Kubernetes
  - Analogy
  - Architecture
---

![alert](/images/kube-banner.png)

# **Memahami Arsitektur Kubernetes dengan Mudah Menggunakan Analogi**

Hellooooooww! 👋

disini saya akan ngebahas arsitektur dari salah satu alat orkestrasi kontainer yang sangat terkenal dan powerfull nih, yaitu **Kubernetes**. Sebelum ke praktik kubernetes, baiknya kita memahami terlebih dahulu apa aja sih komponen penting yang perlu kita kenali agar lebih memahami mengenai Kubernetes.

Nah supaya mudah, disini saya akan menjelaskan komponen yang ada di **Kubernetes Arsitektur** menggunakan **Analogi**!! Langsung aja kita kenalan terlebih dahulu sama Kubernetes, sebenernya Kubernetes itu apa sih?

---

# Kubernetes

![kube](/images/kube-logo.png)

Menurut situs web kubernetes.io, Kubernetes adalah: 
> "Open Source System untuk mengotomatiskan penyebaran, penskalaan, dan manajemen aplikasi yang terkontainerisasi".

Kubernetes, sering disingkat **K8s**, adalah platform open-source yang dirancang untuk mengelola dan mengatur container yang berisi aplikasi beserta semua dependensinya. Dengan analogi, kita bisa membayangkan **aplikasi sebagai produk makanan cepat saji**, maka **Kubernetes adalah sistem dapur** yang memastikan semua burger, kentang goreng, dan minuman siap dengan cepat, dalam jumlah yang tepat, dikirim ke pelanggan tanpa gangguan dan tentu saja rasa yang nikmat😋✨.

Awalnya dikembangkan oleh Google dan kini dikelola oleh **Cloud Native Computing Foundation (CNCF)**, Kubernetes ini membantu perusahaan menjalankan aplikasi mereka di berbagai lingkungan dari data center pribadi hingga layanan cloud seperti AWS, Azure, atau Google Cloud.

## Mengapa Kubernetes Sangat Penting?

Dalam dunia teknologi modern, aplikasi itu harus:

1. **Cepat dan fleksibel** untuk memenuhi kebutuhan pengguna.
2. **Dapat diandalkan**, meskipun terjadi gangguan.
3. **Efisien** dalam menggunakan sumber daya, sehingga menghemat biaya.

Kubernetes mempermudah pengelolaan aplikasi modern yang sering terdiri dari puluhan atau bahkan ratusan container. Tanpa Kubernetes, mengelola container ini seperti mencoba mengatur ribuan pemain di sebuah konser musik tanpa instruksi yang jelas dan tentunya akan menyebabkan kekacauan, nah ini Kubernetes seperti **konduktor** yang memastikan semuanya berjalan harmonis.

![kube](/images/analogi-1.png)

## Manfaat utama Kubernetes:
* **Otomasi**: Semua tugas manual seperti menyalakan, mematikan, atau memindahkan aplikasi dapat diotomatiskan.
* **Portabilitas**: Anda bisa menjalankan aplikasi di lingkungan apa pun tanpa banyak perubahan.
* **Efisiensi**: Kubernetes mengoptimalkan penggunaan server, menghindari pemborosan sumber daya.

---

# Arsitektur Kubernetes

Sebelum ke satu persatu komponen yang ada didalam arsitektur Kubernetes, perhatikan terlebih dahulu ini:

![kube](/images/kube-arsitektur.jpg)

dalam arsitektur kubernetes, agar mudah pembahasannya disini saya bagi dua, yaitu Master Node dan Worker Node yang dimana keduanya memiliki fungsi berbeda.

## Master Node
Master Node adalah pusat kendali dari kluster Kubernetes. Komponen-komponen di dalamnya bertugas mengelola, memonitor, dan mengatur semua operasi yang terjadi pada Worker Node. Komponen penting pada Master Node itu apa aja sih?

**1.Kube-API Server**

API Server adalah pintu gerbang utama atau **central communication point** untuk semua interaksi di dalam kluster Kubernetes. Tugasnya adalah menerima perintah dari pengguna (melalui kubectl atau UI lain), kemudian mendistribusikan tugas tersebut ke komponen lainnya. API Server berperan sebagai **"penghubung"** antara pengguna, Master Node, dan Worker Node.

Bayangkan kluster Kubernetes sebagai sebuah hotel besar dengan banyak kamar (nodes), tempat tamu (aplikasi atau pod) tinggal dan beraktivitas. Dalam analogi ini, Kube-API Server adalah **resepsionis hotel**, yang menjadi penghubung antara tamu, staf hotel, dan manajemen. Ketika resepsionis menerima permintaan tamu, maka resepsionis akan berkomunikasi langsung ke staf dan manajemen hotel untuk memenuhi permintaan tersebut. Selain itu, resepsionis juga akan mencatat informasi setiap aktivitas ataupun permintaan yang nantinya akan disimpan dan dikumentasikan secara baik.

![kube](/images/analogi-2.png)

---
**2.Kube-Scheduler**

Kube-Scheduler bertugas menentukan di mana (pada Worker Node mana) aplikasi/container akan dijalankan. Scheduler juga memonitor sumber daya yang tersedia pada node dan memilih node terbaik untuk menjalankan Pod berdasarkan kebutuhan aplikasi. Selain mengatur dimana pod akan ditempatkan, kita juga bisa mengatur bagaimana pod ingin dijalankan seperti berdasarkan label ataupun lainnya.

Bayangkan Kube-Scheduler sebagai **sistem parkir cerdas** yang bertugas menempatkan mobil (Pod) di tempat parkir (Node) yang paling sesuai. Dalam analogi ini, tugas Kube-Sheduler beragam, seperti Menganalisis ketersediaan tempat parkir, menentukan tempat parkir yang tepat berdasarkan analisa, serta akan menangani konflik ketika ada beberapa kendaraan yang membutuhkan tempat yang sama.

![kube](/images/analogi-3.png)

---
**3.Kube-Controller Manager**

Controller Manager bertanggung jawab untuk memonitor status kluster dan mengambil tindakan otomatis jika terjadi perubahan. Jadi ia akan memastikan bahwa status dari Cluster Kubernetes kita sesuai dengan yang kita inginkan / tentukan. Misal terjadi masalah dimana Pod gagal berjalan, Controller Manager akan memicu proses untuk membuat ulang Pod tersebut.

Controller Manager dalam Kubernetes dapat dianalogikan seperti seorang **pengawas proyek** konstruksi di sebuah lokasi pembangunan. Pengawas proyek ini memiliki tanggung jawab untuk memastikan semua aspek proyek berjalan sesuai rencana. Jika ada sesuatu yang tidak sesuai, ia segera mengambil tindakan untuk memperbaikinya.

Misalnya, ketika seorang pekerja tidak hadir (diibaratkan sebagai sebuah Pod yang gagal), pengawas proyek akan segera mencari pengganti untuk memastikan pekerjaan tetap berjalan. Jika ada peralatan atau bahan bangunan yang rusak (ibarat Node yang bermasalah), ia akan melaporkannya dan mengatur perbaikan atau penggantian. Dengan perannya yang terus mengawasi dan menjaga keseimbangan, Controller Manager memastikan bahwa "proyek" Kubernetes, yakni seluruh sistem cluster, berjalan dengan lancar dan sesuai rencana.

![kube](/images/analogi-4.png)

---
**4.etcd**

etcd adalah sebuah database key-value yang menjadi pusat penyimpanan data dalam Kubernetes. Ia bertugas menyimpan semua informasi penting mengenai keadaan cluster, seperti konfigurasi, status node, daftar Pod, dan informasi lainnya yang dibutuhkan untuk menjalankan Kubernetes dengan baik.Salah satu alasan etcd penting adalah karena Kubernetes bergantung pada data yang tersimpan di dalamnya untuk menjaga koordinasi antar komponen. Jika ada perubahan dalam cluster, seperti penambahan Pod baru atau penghapusan layanan, etcd akan langsung diperbarui agar semua bagian dari sistem tetap sinkron.

Singkatnya, etcd adalah pusat data yang memastikan Kubernetes dapat berjalan secara konsisten dan efisien, karena setiap tindakan dalam cluster selalu dicatat dan dapat diakses dengan cepat.

Bisa dianalogikan, etcd sebagai **arsip perusahaan** yang mencatat semua dokumen penting terkait operasional perusahaan secara terorganisir. Arsip ini menyimpan informasi seperti daftar karyawan, jadwal proyek, inventaris aset, hingga status pekerjaan yang sedang berlangsung.

Ketika seorang manajer atau karyawan membutuhkan informasi misalnya, siapa yang bertanggung jawab atas sebuah proyek atau di mana inventaris tertentu disimpan, mereka bisa langsung merujuk ke arsip tersebut untuk mendapatkan data yang akurat dan terkini. Jika ada perubahan, seperti karyawan baru yang bergabung atau proyek yang selesai, arsip ini diperbarui secara real-time sehingga semua pihak tetap memiliki informasi yang konsisten.

![kube](/images/analogi-5.png)

---


## Worker Node
Worker Node adalah tempat di mana aplikasi dijalankan. Node ini menjalankan perintah yang diterima dari Master Node. Nah komponen utama pada Worker Node itu emangnya apa aja?

**1.Kubelet**

Kubelet seperti asisten pribadi di setiap node dalam cluster Kubernetes. Tugas utamanya adalah memastikan semua Pod dan kontainer yang berjalan di node tersebut sesuai dengan perintah yang diberikan oleh API Server. Kubelet secara terus-menerus memonitor status Pod dan kontainer, melaporkan kondisinya kembali ke API Server, dan memastikan semuanya tetap berjalan sebagaimana mestinya. Jika ada masalah, misalnya kontainer berhenti tiba-tiba, Kubelet akan mencoba menjalankan ulang kontainer tersebut berdasarkan definisi konfigurasi.

Bisa di analogikan seperti ini, setiap kantor memiliki seorang **supervisor** (kubelet) yang memastikan bahwa semua **karyawan** (pod) menjalankan tugas mereka sesuai dengan instruksi manajer pusat. Supervisor ini terus memeriksa pekerjaan karyawan, memastikan mereka bekerja sesuai dengan rencana, dan melaporkan progresnya ke manajer. Jika ada masalah, seperti karyawan yang tidak hadir atau tugas yang tidak selesai, supervisor akan mengambil tindakan untuk memperbaikinya.

![kube](/images/analogi-6.png)

---
**2.Kube-Proxy**

Kube Proxy itu seperti juru lalu lintas di dalam cluster Kubernetes. Ia bertanggung jawab untuk mengatur lalu lintas jaringan antara Pod, layanan (Service), dan pengguna yang mengakses aplikasi dalam cluster. Ketika sebuah permintaan datang, Kube Proxy memastikan permintaan tersebut diarahkan ke Pod yang benar dengan cara yang efisien. Misalnya, jika sebuah layanan memiliki banyak Pod yang mendukungnya, Kube Proxy akan melakukan load balancing untuk membagi permintaan ke berbagai Pod tersebut.

Nah dengan begitu, Kube Proxy bisa dianalogikan seperti **juru lalu lintas udara** di sebuah bandara besar. Tentu saja lalu lintas masuk dan keluar itu ramai donng, nah disini tugasnya juru lalu lintas udara, ia akan mengaturur lintasan supaya tidak ada tabrakan, menjaga dan memastikan lalu lintas aman, serta bisa juga menyesuaikan jalur penerbangan supaya seluruh lalu lintas antar apapun itu aman terkendali.

![kube](/images/analogi-7.png)

---
**3.Container Runtime**

Container Runtime adalah bagian dari infrastruktur yang memungkinkan kontainer berjalan di dalam node. Ia seperti mesin penggerak kontainer. Kubernetes mendukung berbagai container runtime seperti **Docker**, **containerd**, atau **CRI-O**. Tugas utama container runtime adalah menjalankan kontainer, menghentikannya, dan mengelola image kontainer. Ketika Kubelet memerintahkan untuk menjalankan sebuah Pod, container runtime-lah yang bertindak langsung untuk memulai kontainer sesuai dengan perintah tersebut. Tanpa container runtime, Kubernetes tidak bisa menjalankan aplikasi apa pun.

Anggaplah ada sebuah **mesin otomatis di pabrik** yang bertugas mengambil bahan baku, mengolahnya, dan menghasilkan produk jadi. Mesin ini bekerja berdasarkan perintah dan spesifikasi yang diberikan oleh **supervisor** (kubelet) . Dalam Kubernetes, container runtime adalah mesin yang bertanggung jawab untuk menjalankan kontainer, mengelola image, dan memastikan kontainer beroperasi sesuai dengan permintaan. Tanpa mesin ini, tidak akan ada aktivitas di pabrik, dan produk tidak bisa dihasilkan.

![kube](/images/analogi-8.png)

---

Nah gimana nih? memahami komponen utama yang ada di arsitektur Kubernetes ini cukup mudah kan dengan adanya analogi? karena dengan analogi kita bisa memudahkan pemahaman terhadap sesuatu yang cukup ribet, namun untuk pendalaman lebih dalam lagi disarankan untuk baca dokumentasi resmi dari dokumentasi resminya ya!

## Referensi

- [Kubernetes.io](https://kubernetes.io/docs/home/)