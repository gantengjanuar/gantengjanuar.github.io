---
title: 'How To Monitoring Instance with Prometheus'
date: 2024-11-15
permalink: /posts/2024/11/prometheus-instance-monitoring/
tags:
  - Openstack
  - Deployment
  - Tutorial
  - Prometheus
  - Cloud-init
  - Grafana
---

![kolla openstack](/images/prome-grafana.png)

# **Instance Monitoring With Prometheus & Grafana**
Halloo semua!! sebelumnya kita udah monitoring Instance Openstack menggunakkan ELK Stack, sekarang kita akan monitoring menggunakan Prometheus!!. Blog ini merupakan dokumentasi sekaligus panduan lengkap mengenai cara Monitoring sebuah instance yang berada di Openstack cluster menggunakan Prometheus dan Grafana.

oh ya, agar mudah dan efisien, saya disini memakai **cloud-init** yang isinya memungkinkan sebuah instance yang baru di launching langsung terpasang node expoerter sehingga kita bisa langsung memvisualisasikan data metrics nya lewat grafana.

selain itu, saya juga menggunakan **template dashboard** grafana sehingga kita bisa monitoring node secara full tanpa memikirkan query nya. menarik bukan... ikuti caranya!!

---

## Latar Belakang

Dalam era teknologi informasi yang semakin berkembang pesat, kebutuhan akan infrastruktur komputasi yang fleksibel, efisien, dan aman menjadi prioritas utama bagi organisasi. Cloud computing hadir sebagai solusi utama, memungkinkan penggunaan sumber daya komputasi secara dinamis dan efisien. Namun, pemanfaatan teknologi cloud memerlukan pendekatan yang lebih terintegrasi untuk memastikan kelancaran operasional, termasuk dalam memonitoring instance yang berjalan.

Monitoring instance adalah proses penting untuk menjaga performa, keamanan, dan ketersediaan sistem. Dengan memonitoring, administrator dapat mendeteksi permasalahan lebih awal, mencegah downtime yang tidak diinginkan, serta memastikan sistem berjalan sesuai dengan kebutuhan. Monitoring mencakup pengumpulan data performa, log, dan metrik penting lainnya, yang memberikan wawasan untuk pengambilan keputusan secara proaktif.

Selain itu, pengelolaan instance dalam cloud juga membutuhkan cara otomatisasi yang efisien, salah satunya melalui penggunaan cloud-init. Cloud-init adalah alat yang dirancang untuk mengkonfigurasi instance saat pertama kali diluncurkan, seperti menginstal paket, mengatur jaringan, atau menambahkan layanan monitoring. Dengan cloud-init, konfigurasi instance dapat dilakukan secara konsisten dan otomatis, mengurangi risiko kesalahan manual serta meningkatkan produktivitas tim operasional.

Oleh karena itu, blog ini akan membahas pentingnya monitoring instance untuk menjaga kestabilan operasional serta peran strategis cloud-init dalam otomatisasi konfigurasi instance di lingkungan cloud. Penjelasan ini diharapkan memberikan gambaran yang lebih jelas mengenai bagaimana kombinasi monitoring dan cloud-init dapat mendukung manajemen infrastruktur modern secara efektif.

---
# Langkah Pengerjaan | Implementasi
Pada kesempatan kali ini, saya akan Launching Instance yang nantinya dipasangkan cloud init berisi instalasi dan konfigurasi node exporter lalu memonitoringnya menggunakan Prometheus dan Grafana. 

> **Note** : Seluruh langkah pengerjaan disini adalah pengerjaan sendiri, sehingga mungkin jika anda ingin mencoba melakukannya juga, anda bisa menyesuaikan dengan environment anda sendiri, seperti penamaan ataupun IP dari VM.

## 1. Openstack
Pertama-tama, pastikan sudah melakukan Instalasi atau deployment Openstack terlebih dahulu, jika belum bisa ikuti panduan ini [How To Deploy Openstack With Kolla-Ansible](https://gantengjanuar.github.io//posts/2024/11/deploy-openstack/)

## 2. Prometheus
Lakukan instalasi Prometheus Server di node controller, seperti ini:

1.pindah ke root dan unduh Prometheus Server di node controller.
```
$ sudo su -
# cd /opt
# wget https://github.com/prometheus/prometheus/releases/download/v2.48.1/prometheus-2.48.1.linux-amd64.tar.gz
# tar xvfz prometheus-2.48.1.linux-amd64.tar.gz
```

2.Konfigurasi config.yml supaya bisa menerima metrics instance.
```
# cd prometheus-2.48.1.linux-amd64
# vim config.yml
```

```
global:
  scrape_interval:     10s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus-controller'
    static_configs:
    - targets: ['10.13.13.10:9090']

  - job_name: 'openstack-sd'
    openstack_sd_configs:
      - identity_endpoint: http://10.13.13.100:5000 #URL keystone
        username: admin
        project_name: admin
        password: xVVXVxFhl60w1GCEbJTEi3EHmG2BHilByawIV337 #ganti dengan password di /etc/kolla/admin-openrc.sh
        region: RegionOne
        role: instance
        domain_name: Default
        port: 9100
    relabel_configs:
      - source_labels: [__meta_openstack_public_ip]
        action: replace
        target_label: __address__
        regex: (.+)
        replacement: ${1}:9100

      - source_labels: [__meta_openstack_instance_name]
        target_label: instance
```
> **Note**: sesuaikan dengan konfigurasi cluster sendiri!

3.Jalankan Prometheus sebagai service.
```
# vim /etc/systemd/system/prometheus_server.service

[Unit]
Description=Prometheus Server

[Service]
User=root
ExecStart=/opt/prometheus-2.48.1.linux-amd64/prometheus --config.file=/opt/prometheus-2.48.1.linux-amd64/config.yml --web.external-url=http://10.13.13.10:9090/ #IP controller

[Install]
WantedBy=default.target
```

4.Aktifkan Prometheus
```
# systemctl daemon-reload
# systemctl enable prometheus_server.service
# systemctl start prometheus_server.service
# systemctl status prometheus_server.service
# journalctl -u prometheus_server
```

5.Untuk verifikasi, bisa coba akses melalui browser:
```
Metrics: http://10.13.13.10:9090/metrics
Graph  : http://10.13.13.10:9090/
Target : http://10.13.13.10:9090/targets
```
> **Note:** Ganti dengan IP Controller!

## 3. Grafana
Lakukan juga instalasi grafana untuk kita membuat visualisasinya.

1.Install grafana di node Controller
```
$ sudo su -
# cd /opt
# wget https://dl.grafana.com/enterprise/release/grafana-enterprise-11.3.0.linux-amd64.tar.gz
# tar -zxvf grafana-enterprise-11.3.0.linux-amd64.tar.gz
```

2.Jalankan Grafana sebagai service.
```
# vim /etc/systemd/system/grafana.service

[Unit]
Description=Grafana

[Service]
User=root
ExecStart=/opt/grafana-v11.3.0/bin/grafana-server -homepath /opt/grafana-v11.3.0/ web

[Install]
WantedBy=default.target
```

3.Jalankan service grafana.
```
# systemctl daemon-reload
# systemctl enable grafana.service
# systemctl start grafana.service
# systemctl status grafana.service
# journalctl -u grafana
```

3.Akses http://10.13.13.10:3000/login di browser.

4.Tambahkan Prometheus data source. Login ke grafana **dashboard > Configuration > Data sources > Add data source.**
```
Type: Prometheus
Name: Prometheus-username
URL: http://10.13.13.10:9090
save & test
```
---

## Launching Instance
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
```
```
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
$ openstack router create router-ganteng
$ openstack router set --external-gateway ganteng-external-net router-ganteng
$ openstack router add subnet router-ganteng ganteng-internal-subnet

$ openstack router list
```

6.Buat security group yang spesifikasi rule nya:
```
$ openstack security group create gan-security-group
```

| Direction | Ether Type | IP Protocol | Port Range | Remote IP Prefix | 
|-----------|------------|-------------|------------|-------------------|
| Egress    | IPv4       | Any         | Any        | 0.0.0.0/0         | 
| Egress    | IPv6       | Any         | Any        | ::/0              |
| Ingress   | IPv4       | ICMP        | Any        | 0.0.0.0/0         | 
| Ingress   | IPv4       | TCP         | 9100       | 0.0.0.0/0         | 


```
$ openstack security group list
```

7.Buat keypair dan verifikasi
```
$ openstack keypair create --public-key ~/.ssh/id_rsa.pub controller-key

$ openstack keypair list
```

8.Buat flavor dan verifikasi.
```
$ openstack flavor create --ram 2048 --disk 8 --vcpus 1 --public mid-spec

$ openstack flavor list
```

9.Buat **Cloud-init** yang berisi node exporter.
```
#cloud-config
write_files:
  - path: /etc/systemd/system/node_exporter.service
    permissions: '0644'
    owner: root:root
    content: |
      [Unit]
      Description=Node Exporter
      Wants=network-online.target
      After=network-online.target

      [Service]
      User=node_exporter
      ExecStart=/usr/local/bin/node_exporter
      Restart=always

      [Install]
      WantedBy=default.target

runcmd:
  - mkdir -p /opt/node_exporter
  - cd /opt/node_exporter
  - wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
  - tar -xzf node_exporter-1.6.1.linux-amd64.tar.gz
  - cp node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
  - useradd -rs /bin/false node_exporter
  - systemctl daemon-reload
  - systemctl enable node_exporter
  - systemctl start node_exporter

```

10.Launching instance.
```
$ openstack server create --flavor mid-spec \
--image ubuntu-image \
--key-name controller-key \
--security-group gan-security-group \
--network ganteng-internal-net \
--user-data cloud-config.yml \
gan-node1
```

11.Pasang floating ip ke instance dan verifikasi.
```
$ openstack floating ip create --floating-ip-address 20.13.13.132 ganteng-external-net
$ openstack server add floating ip gan-node1 20.13.13.132

$ openstack server list
```
## Verifikasi metrics Instance

1.Coba akses instance.
```
$ ssh ubuntu@20.13.13.132
```

2.Verifikasi node exporter udah berjalan.
```
$ sudo systemctl status node_exporter.service
```

3.Verifikasi metrics berhasil terkirim ke Prometheus server
* Buka browser > akses **http://10.13.13.10:9090/targets** dan pastikan instance yang di launching tampil dengan keadaan up.

![prome](/images/prome-1.png)

---

## Visualisasi dengan Grafana menggunakan Template

1.Akses Grafana lewat port 3000 di browser **10.13.13.10:3000** 
> Sesuaikan ip nya.

2.Buka browser, cari **node exporter full grafana dashboard** dan klik link paling atas.
![grafana](/images/prome-2.png)

3.Scroll dikit kebawah, lalu di bagian kanan ada **Copy ID Dashboard** lalu klik dan simpan id Template dashboard.

4.Buka kembali grafana. klik **Home > Dashboard > New > New Dashboard**
![grafana](/images/prome-3.png)

5.Klik **Import a Dashboard** lalu masukkan ID yang sudah di copy sebelumnya dan klik **load**.
![grafana](/images/prome-4.png)

6.Sesuaikan Nama, UID, data source, dll. lalu klik **import**

7.Dan ya, sekarang kita memiliki Dashboard yang sangat lengkap dan dapat memonitoring instance kita.
![grafana](/images/prome-5.png)
