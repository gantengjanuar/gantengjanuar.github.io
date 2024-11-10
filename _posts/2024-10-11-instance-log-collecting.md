---
title: 'Openstack Instance Log and Metric Collection using ELK Stack'
date: 2024-10-11
permalink: /posts/2024/11/instance-log-collecting/
tags:
  - Openstack
  - ELK Stack
  - Documentation
---

![Openstack-ElkStack](/images/Openstack-ELkStack.png)

# Instance Log and Metric Collecting for Monitoring With ELK Stack
Hallo Semuanya!, Blog ini merupakan panduan lengkap mengenai cara mengumpulkan Log dan Metric dari sebuah Instance yang berada di Openstack Cluster. Disini, kita akan memanfaatkan Tools **ELK Stack** ( Filebeat, MetricBeat, Elasticsearch dan Kibana) untuk pengelolaan log dan metric yang cukup kompleks.

Selain itu, disini juga akan dijelaskan dari awal dari mulai Deploy Openstack menggunakan Kolla-Ansible hingga Monitoring Instance menggunakan Dashboard yang datanya diambil dari Log dan juga Metric dua Instance. Dengan menggunakan Dashboard, memungkinkan kita untuk Monitoring Instance secara efektif 

## Latar Belakang
Dalam era digital saat ini, manajemen infrastruktur TI yang efisien menjadi sangat penting bagi keberlangsungan operasi perusahaan. OpenStack, sebagai platform cloud computing yang open-source, memungkinkan perusahaan untuk membangun dan mengelola infrastruktur cloud secara fleksibel dan scalable.

Namun, dengan kompleksitas yang dihadapi dalam pengoperasian cluster OpenStack, tantangan muncul dalam hal pemantauan dan pengumpulan log dari setiap instance. Tim operasional sering kali dihadapkan pada kesulitan dalam mengakses dan menganalisis log dari masing-masing instance, yang dapat menghambat respons terhadap isu-isu yang mungkin terjadi. Untuk mengatasi tantangan ini, penting untuk memiliki sistem monitoring yang terpusat, yang tidak hanya mengumpulkan log tetapi juga memfasilitasi pengukuran metrik kinerja dan pengaturan alerting.

Dengan menerapkan alat seperti ElasticSearch, perusahaan dapat mengkonsolidasikan log dari berbagai instance ke dalam satu platform. Ini memungkinkan tim operasional untuk memantau kondisi sistem secara real-time, menganalisis data log dengan lebih efisien, dan mengambil tindakan yang diperlukan berdasarkan informasi yang diperoleh. Selain itu, penggunaan metrik dan alerting akan meningkatkan kemampuan pemantauan, memungkinkan deteksi masalah lebih awal dan pengambilan keputusan yang lebih cepat.

## Tools yang Digunakan*
* OpenStack	 	      - v7.2.1
* kolla-ansible 		- v2023.1
* Elasticsearch 		- v 8.15.3
* Kibana  		      - v 8.15.3
* Filebeat	      	-v 8.15.3
* Metricbeat		    -v 8.15.3

## Topologi Saya
![Topologi](/images/Topologi-1.png)

## Alur Kerja Saya
![Topologi](/images/Workflow-1.png)


# Teori
Sebelum masuk jauh ke panduan, ada beberapa hal dulu yang sangat penting untuk kita pahami supaya pengerjaan nantinya menjadi lebih mudah, apa saja itu?

## 1. Log
Log adalah catatan atau rekaman peristiwa yang terjadi pada suatu sistem atau aplikasi. Log ini biasanya berupa teks yang memuat informasi penting mengenai aktivitas, kesalahan, atau status dari sistem atau aplikasi. Log sering disimpan dalam bentuk file dan dapat dianalisis untuk mengidentifikasi masalah atau memahami pola penggunaan.

## 2. Metric
Metric adalah data numerik yang dikumpulkan dari sistem atau aplikasi dan diukur  untuk memberikan informasi tentang performa atau kondisi dari suatu sistem. Berbeda dengan log yang berupa teks, metric biasanya berupa data kuantitatif yang memberikan gambaran mengenai status sistem secara real-time.

## 3. Alerting
Alerting adalah proses pemberian peringatan atau notifikasi ketika terjadi kondisi tertentu yang dianggap kritis atau abnormal di sistem. Proses ini menggunakan data dari log dan metric untuk memantau ambang batas yang telah ditentukan (thresholds). Jika nilai yang diamati melampaui ambang batas tersebut, sistem alert akan mengirimkan peringatan kepada administrator.

## 4. Monitoring
Monitoring adalah proses pemantauan secara aktif dan real-time terhadap kondisi dan performa sistem atau aplikasi untuk memastikan bahwa sebuah sistem bekerja sebagaimana mestinya. Dalam monitoring, data dari log dan metric dikumpulkan, dianalisis, dan divisualisasikan dalam bentuk grafik atau dasbor (dashboard) sehingga tim operasional dapat dengan mudah melihat status terkini.

## 5. OpenStack

OpenStack adalah platform komputasi awan (cloud computing) open-source yang dirancang untuk membangun dan mengelola cloud publik atau privat. OpenStack menyediakan infrastruktur sebagai layanan (IaaS) dengan menyediakan berbagai layanan utama seperti:
* **Nova** untuk pengelolaan komputasi (virtual machine instances).
* **Neutron** untuk jaringan virtual.
* **Cinder** untuk manajemen storage atau penyimpanan blok.
* **Swift** untuk penyimpanan objek.
* **Horizon** sebagai dashboard grafis bagi pengguna untuk mengelola resource.
* **Keystone** untuk layanan autentikasi dan otorisasi.
OpenStack memungkinkan pengguna untuk mengelola resource komputasi, jaringan, dan storage dengan mudah dalam skala besar. Pengguna bisa melakukan otomatisasi pada penyebaran server, memantau kinerja, dan mengelola resource sesuai kebutuhan mereka.

## 6. Kolla-Ansible
Kolla-Ansible adalah proyek open-source yang menyediakan alat untuk deployment (penyebaran) OpenStack menggunakan Docker container dan Ansible playbooks. Ini merupakan bagian dari ekosistem OpenStack Kolla, yang berfokus pada pengemasan layanan OpenStack ke dalam container untuk penyebaran yang lebih mudah dan konsisten. Beberapa fitur dan konsep penting dalam Kolla-Ansible adalah:

## 7. Instance
Instance di OpenStack adalah sebuah virtual machine (VM) yang dijalankan di dalam lingkungan cloud OpenStack. Setiap instance merupakan unit komputasi yang di-boot menggunakan gambar (image) tertentu dan dijalankan di atas hypervisor yang dikelola oleh layanan Nova di OpenStack.

## 8. Filebeat
Filebeat adalah agen ringan yang digunakan untuk mengirimkan log dari server ke sistem pengelolaan log seperti Logstash atau Elasticsearch. Filebeat membaca log dari file log yang ada di dalam server (misalnya syslog, auth log) dan mengirimkannya ke target tujuan untuk dianalisis lebih lanjut.

## 9. Metricbeat
Metricbeat adalah agen yang mengumpulkan metric dari sistem dan aplikasi, lalu mengirimkannya ke penyimpanan data seperti Elasticsearch atau Logstash. Data metricbeat ini kemudian bisa divisualisasikan di dashboard seperti Kibana.

## 10. Logstash
Logstash adalah alat pengumpulan, pemrosesan, dan penguraian data yang biasanya digunakan dalam pipeline ELK Stack. Logstash menerima data dari berbagai sumber, seperti Filebeat dan Metricbeat, lalu memprosesnya dan mengirimkannya ke Elasticsearch untuk disimpan dan dianalisis.

## 11. Elasticsearch
Elasticsearch adalah mesin pencarian dan analisis terdistribusi yang digunakan untuk menyimpan, mencari, dan menganalisis data besar secara real-time. Dalam ELK Stack, Elasticsearch adalah komponen utama untuk menyimpan data log dan metric yang kemudian bisa dianalisis dan divisualisasikan.

## 12. X-Pack
X-Pack adalah sekumpulan plugin yang dikembangkan oleh Elastic untuk menambahkan fitur keamanan, monitoring, alerting, dan manajemen data ke dalam Elasticsearch dan Kibana.

## 13. Kibana
Kibana adalah alat visualisasi dan dashboard dalam ELK Stack yang digunakan untuk memvisualisasikan data yang disimpan di Elasticsearch. Kibana memungkinkan pengguna untuk membuat grafik, peta, dan visualisasi lain untuk memahami data log dan metric secara lebih mendalam.


# Langkah pengerjaan / Implementasi


## Deploy Openstack pada Controller

1.Install sependensi yang dibutuhkan.
```
$ sudo apt update
$ sudo apt-get install python3-dev libffi-dev gcc libssl-dev python3-selinux python3-setuptools python3-venv -y
```
2.Buat virtual envrionment dan aktifkan.
``` 
$ python3 -m venv kolla-venv
$ source kolla-venv/bin/activate
```

3.Install Ansible dan  Kolla-Ansible.
```
$ pip install -U pip
$ pip install 'ansible>=6,<8'
$ pip install git+https://opendev.org/openstack/kolla-ansible@stable/2023.1
$ kolla-ansible install-deps
```
4.Buat direktori /etc/kolla dan copy konfigurasi yang dibutuhkan.

```
$ sudo mkdir -p /etc/kolla
$ sudo chown $USER:$USER /etc/kolla
$ cp -r kolla-venv/share/kolla-ansible/etc_examples/kolla/* /etc/kolla
```

5.Edit inventory Multinode seperti dibawah:
```
$ cp kolla-venv/share/kolla-ansible/ansible/inventory/* .
$ vim ~/multinode
``` 
```
[control]
hostname-controller

[network]
hostname-controller

[compute]
hostname-compute1
hostname-compute2

[monitoring]
hostname-controller

[storage]
hostname-controller
hostname-compute1
hostname-compute2

[deployment]
localhost ansible_connection=local
```


> **Note:** ganti **hostname** dengan hostname controller dan compute anda.

6.Konfigurasi ansible.cfg.
```
$ sudo mkdir -p /etc/ansible
$ sudo vim /etc/ansible/ansible.cfg
```
```
[defaults]
host_key_checking=False
pipelining=True
forks=100
```
7.Uji konektivitas Ansible
```
$ ansible -i multinode all -m ping
```
8.Generate kolla passwd
```
$ kolla-genpwd
$ cat /etc/kolla/passwords.yml
```
9.konfigurasi globals.yml.
```
$ sudo vim /etc/kolla/globals.yml

kolla_base_distro: "ubuntu"
openstack_release: "2023.1"
network_interface: "ens3"
neutron_external_interface: "ens4"
kolla_internal_vip_address: "10.10.10.11" #ganti dengan IP VIP anda
neutron_plugin_agent: "openvswitch"
enable_keystone: "yes"
enable_horizon: "yes"
enable_neutron_provider_networks: "yes"
enable_cinder: "yes"
enable_cinder_backend_lvm: "yes"
```
> **Note:** ganti "**yes**" dengan "**no**" jika ingin menonaktifkan service nya.

10.Lakukan Booststrap , pre deploy , Deployment dan Post deploy.
```
$ kolla-ansible -i ./multinode prechecks
$ kolla-ansible -i ./multinode deploy
$ kolla-ansible -i ./multinode post-deploy
``` 
> **Note:** Proses ini memakan waktu berapa menit jadi harap sabar.
11.Lakukan Instalasi openstackclient dan verifikasi Openstack
```
$ pip install openstackclient
$ source /etc/kolla/admin-openrc.sh
```
```
$ openstack service list 
```
> **Note:** Pastikan Service inti yang diinginkan sudah active.

## Install Elasticsearch pada Controller
1.Update repo dan install dependensi yang dibutuhkan.
```
$ sudo apt update
$ sudo apt install -y openjdk-17-jre apt-transport-https curl wget
```

2.Download dan install public sign key serta tambahkan repositori elastic.
```
$ wget -qo - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```
```
$ echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-8.x.list
```
3.Install Elasticsearch dan jalankan.
```
$ sudo apt update && sudo apt -y install elasticsearch

$ sudo systemctl start elasticsearch. service
$ sudo systemctl enable elasticsearch.service
```
4.Konfigurasi Elasticsearch agar mendengarkan semua ipv4 network.
```
$ sudo vim /etc/elasticsearch/elasticsearch.yml
```
```
network.host: 0.0.0.0
http.port: 9200

discovery.seed_hosts: ["127.0.0.1", "[ :: 1]"]
# Enable security features
xpack. security. enabled: false
```

5.Restart Elasticsearch.
$ sudo systemctl restart elasticsearch
$ sudo systemctl status elasticsearch

## Install Kibana pada Controller
1.Install Kibana dan jalankan.
```
$ sudo apt install -y kibana

$ sudo systemctl start kibana
$ sudo systemctl enable kibana
$ sudo systemctl status kibana
```

2.Konfigurasi Kibana untuk mendengarkan semua network IPv4.
```
$ sudo vim /etc/kibana/kibana.yml

server.host: "0.0.0.0"
```
3.Restart Kibana.
```
$ sudo systemctl restart kibana
```


## Install Logstash pada Controller
1.Install logstash.
```
$ sudo apt-get install -y logstash
```

2.Jalankan dan verifikasi.
```
$ sudo systemctl enable logstash
$ sudo systemctl start logstash
$ sudo systemctl status logstash
```


## Menerapkan keamanan X-Pack
1.Stop Kibana dan Elasticsearch.
```
$ sudo systemctl stop kibana && elasticsearch
```

2.Nyalakan X-pack di konfigurasi elasticsearch.yml
```
$ sudo nano /etc/elasticsearch/elasticsearch.yml

# Enable security features
xpack.security.enabled: true

# Disable
xpack. security.http.ssl:
enabled: false
keystore.path: certs/http.p12
```

3.Setup password user elastic untuk autentikasi
``` 
$ cd /usr/share/elasticsearch/bin
$ sudo ./elasticsearch-reset-password -u elastic -i
```
> **Note:** isi dengan password “elastic”

4.Setup password user kibana 
``` 
$ sudo ./elasticsearch-reset-password -u kibana_system -i
```
> **Note:** isi dengan password “kibana”

5.Simpan user dan password di catatan.

6.Konfigurasi kibana.yml
```
$ sudo nano /etc/kibana/kibana.yml

elasticsearch.username: "kibana_system"
elasticsearch.password: "kibana"
```

7.Jalankan kembali Elasticsearch dan Kibana lalu Akses kibana dan Login dengan user elastic.
Akses browser: http://10.13.13.10:5601/

