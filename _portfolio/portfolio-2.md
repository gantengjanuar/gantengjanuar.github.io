---
title: "Project: Collecting Metrics Instance Openstack With Prometheus dan Grafana"
excerpt: " <br/><img src='/images/metrics.png' width='300'>"
collection: portfolio
---
![Openstack-ElkStack](/images/metrics.png)

--- 
# Project: **Mengumpulkan Metrics Instance pada Openstack Cluster Menggunakan Prometheus dan Grafana**

## Deskripsi Project: 

Pada project ini, saya mengatasi masalah dimana tim operasi membutuhkan tools atau alat Monitoring seperti Dashboard untuk memantau status dan kondisi dari sebuah instance supaya bisa mendeteksi masalah lebih dini. Metrics Instance saya ambil menggunakan **Prometheus** dan nanti akan di visualisasikan di **Grafana** dengan bentuk Dashboard. Saya juga membuat Alert supaya ketika instance mati akan ada peringatan yang dikirim melalui **AlertManager** ke **Gmail**, **Discord**, dan juga **Slack**

Untuk launching Instance nya saya juga menambahkan **cloud-init** supaya ketika di launching nanti instance sudah terpasang **node exporter**

---
## Hasil Akhir Project
Saya berhasil membuat Dashboard yang akan digunakan untuk memantau kondisi sebuah Instance secara full, dimulai dari kondisi Disk, CPU, Memory, Network, dll, Selain itu saya juga membuat Alert untuk Instance yang akan dikirim via Gmail, Discord, dan Slack. Saya juga membuat Blog berisi Dokumentasi yang bisa dibaca dan dilihat [disini | Instance Monitoring](https://gantengjanuar.github.io//posts/2024/11/prometheus-instance-monitoring/) dan [disini | Alerting](https://gantengjanuar.github.io//posts/2024/11/prometheus-instance-monitoring/)

---
## Dashboard | Basic Info
![hasil Akhir](/images/basic-info.png)

---
## Dashboard | CPU-Memory-Network-Disk
![hasil Akhir](/images/cpu-memory.png)

---

## Alerting | Instance Down
![Alert](/images/prome-alert.png)

# Tes Alerting  | Gmail-Discord-Slack
![alerting](/images/matiin-instance-3.png)

![alerting](/images/matiin-instance-4.png)

![alerting](/images/matiin-instance-5.png)