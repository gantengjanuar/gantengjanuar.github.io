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
---

![kolla openstack](/images/prome-grafana.png)

# **Instance Monitoring With Prometheus & Grafana**
Blog ini merupakan dokumentasi sekaligus panduan lengkap mengenai cara Monitoring sebuah instance yang berada di Openstack cluster menggunakan Prometheus dan Grafana.

oh ya, agar mudah dan efisien, saya disini memakai **cloud-init** yang isinya memungkinkan sebuah instance yang baru di launching langsung terpasang node expoerter sehingga kita bisa langsung memvisualisasikan data metrics nya lewat grafana.

selain itu, saya juga menggunakan **template dashboard** grafana sehingga kita bisa monitoring node secara full tanpa memikirkan query nya. menarik bukan... ikuti caranya!!

---

## Latar Belakang

Dalam era teknologi informasi yang semakin berkembang pesat, kebutuhan akan infrastruktur komputasi yang fleksibel, efisien, dan aman menjadi prioritas utama bagi organisasi. Cloud computing hadir sebagai solusi utama, memungkinkan penggunaan sumber daya komputasi secara dinamis dan efisien. Namun, pemanfaatan teknologi cloud memerlukan pendekatan yang lebih terintegrasi untuk memastikan kelancaran operasional, termasuk dalam memonitoring instance yang berjalan.

Monitoring instance adalah proses penting untuk menjaga performa, keamanan, dan ketersediaan sistem. Dengan memonitoring, administrator dapat mendeteksi permasalahan lebih awal, mencegah downtime yang tidak diinginkan, serta memastikan sistem berjalan sesuai dengan kebutuhan. Monitoring mencakup pengumpulan data performa, log, dan metrik penting lainnya, yang memberikan wawasan untuk pengambilan keputusan secara proaktif.

Selain itu, pengelolaan instance dalam cloud juga membutuhkan cara otomatisasi yang efisien, salah satunya melalui penggunaan cloud-init. Cloud-init adalah alat yang dirancang untuk mengkonfigurasi instance saat pertama kali diluncurkan, seperti menginstal paket, mengatur jaringan, atau menambahkan layanan monitoring. Dengan cloud-init, konfigurasi instance dapat dilakukan secara konsisten dan otomatis, mengurangi risiko kesalahan manual serta meningkatkan produktivitas tim operasional.

Oleh karena itu, blog ini akan membahas pentingnya monitoring instance untuk menjaga kestabilan operasional serta peran strategis cloud-init dalam otomatisasi konfigurasi instance di lingkungan cloud. Penjelasan ini diharapkan memberikan gambaran yang lebih jelas mengenai bagaimana kombinasi monitoring dan cloud-init dapat mendukung manajemen infrastruktur modern secara efektif.

# Langkah Pengerjaan | Implementasi
Pada kesempatan kali ini, saya akan Launching Instance yang nantinya dipasangkan cloud config berisi instalasi dan konfigurasi node exporter lalu memonitoringnya menggunakan Prometheus dan Grafana. 

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
        password: xVVXVxFhl60w1GCEbJTEi3EHmG2BHilByawIV337 #ganti dengan password di /etc/kolla/
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
