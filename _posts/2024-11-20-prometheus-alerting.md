---
title: 'How To Send Alerts  from Prometheus to Multiple Target'
date: 2024-11-20
permalink: /posts/2024/11/send-alert-to-multiple/
tags:
  - Openstack
  - Tutorial
  - Prometheus
  - Alerting
---

![alert](/images/alert.png)

# **Sending Alert from Prometheus to Multiple Target**
Hallo halloo, sebelumnya saya udah berhasil monitoring Instance yang berada di Openstack cluster menggunakan Prometheus dan Grafana sebagai visualisasinya yang saya dokumentasikan [ disini](https://gantengjanuar.github.io//posts/2024/11/prometheus-instance-monitoring/), nah sekarang merupakan tahap lanjutan dari monitoring nih, yaitu **Alerting**

Disini, saya mengirim alert ke berbagai platform yaitu **Gmail**, **Discord**, dan juga **Slack** yang tentunya akan ngebikin alert kita makin lengkap.

Sebelum lanjut, sebenernya Alerting itu apasih?

---

## Latar Belakang
Dalam era digital, infrastruktur IT menjadi tulang punggung bagi operasional bisnis dan layanan modern. Keandalan, ketersediaan, dan performa sistem menjadi faktor utama yang memengaruhi pengalaman pengguna dan produktivitas organisasi. Namun, kompleksitas sistem IT, terutama dalam lingkungan cloud dan DevOps, sering kali menghadirkan tantangan dalam mendeteksi dan merespons masalah secara cepat.

**Alerting** adalah solusi yang dirancang untuk memberikan notifikasi otomatis kepada tim IT atau DevOps saat terjadi anomali atau kondisi tertentu dalam sistem. Proses ini melibatkan pemantauan metrik, log, atau event, serta pengiriman notifikasi ketika ambang batas (threshold) telah terlampaui.

## Alerting

Simpelnya, **Alerting** adalah proses pemberitahuan atau pengiriman notifikasi secara otomatis saat kondisi tertentu terpenuhi dalam suatu sistem atau infrastruktur IT. Proses ini biasanya menggunakan alat monitoring seperti Prometheus, Grafana, Kibana, atau sistem serupa yang mendeteksi kejadian atau metrik tertentu, seperti lonjakan penggunaan CPU, kegagalan disk, atau akses tak sah. Alerting dirancang untuk memberitahukan tim atau individu yang bertanggung jawab, melalui berbagai saluran seperti email, Slack, SMS, atau dashboard.

---

# Langkah Pengerjaan | Implementasi
Kali ini, saya melanjutkan blog sebelumnya yang dimana saya udah launching Instance Openstack dan memonitoring menggunakan Prometheus dan Grafana. Jika penasaran dengan blog sebelumnya, [baca disini](https://gantengjanuar.github.io//posts/2024/11/prometheus-instance-monitoring/)

Pertama, saya harus lakukan Instalasi AlertManager di vm Controller.

1.Pindah ke user root, download AlertManager.
```
$ sudo su -
# cd /opt
# wget https://github.com/prometheus/alertmanager/releases/download/v0.26.0/alertmanager-0.26.0.linux-amd64.tar.gz
# tar xvfz alertmanager-0.26.0.linux-amd64.tar.gz
```
2.Dapatkan password aplikasi google, Webhook Discord dan juga Slack
* Gmail
  1. Kita memerlukan password aplikasi google, yang bisa didapatkan [disini](https://myaccount.google.com/u/0/apppasswords) 

* Discord
  1. Buat sebuah channel "Alert" lalu klik **edit channel**
  ![alerting](/images/alerting-1.png)
  
  2. Tambahkan intergasi webhook 
  ![alerting](/images/alerting-2.png)

  3. Buat webhook, Copy URL Webhook
  ![alerting](/images/alerting-3.png)

* Slack
  1. Tambahkan Webhook ke slack, klik **Garis 3 > File > Settings & Administration > Manage App**
  ![alerting](/images/alerting-5.png)

  2. di pencarian, cari **incoming Webhook** lalu klik **add to slack** 
  ![alerting](/images/alerting-4.png)

  3. Atur konfigurasi webhook agar mengirim ke channel alerts, lalu copy URL webhook dan **Save Settings**
  ![alerting](/images/alerting-6.png)

3.Konfigurasi AlertManager
```
# cd alertmanager-0.26.0.linux-amd64
# vim config.yml
```

```
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  receiver: 'Gmail-Notif'
  routes:
    - receiver: 'Gmail-Notif'
      continue: true
    - receiver: 'Discord-Notif'
      continue: true
    - receiver: 'Slack-Notif'

receivers:
  - name: 'Gmail-Notif'
    email_configs:
      - to: 'your@email'
        from: 'your@email'
        smarthost: 'smtp.gmail.com:587'
        auth_username: 'your@email'
        auth_password: 'yourpassword' # Password aplikasi Google
        send_resolved: true

  - name: 'Discord-Notif'
    discord_configs:
      - webhook_url: 'https://discordapp.com/api/webhooks/1308461062278090772/ALO3z_SJfB6-9T0slbSBwSnWzhhHsdUstejXNz2P4mUA_Q9qJF04mDSX02Pp47j34Ybc' #URL Webhook Discord Discord Anda
        send_resolved: true

  - name: 'Slack-Notif'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/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' #URL Webhook Slack
        channel: '#alerts'
        send_resolved: true
        text: |
          :rotating_light: *Alert:* {{ .CommonAnnotations.summary }}
          *Description:* {{ .CommonAnnotations.description }}
          *Instance:* {{ .CommonLabels.instance }}
```

4.Jalankan AlertManager sebagai service     
```
# vim /etc/systemd/system/alert_manager.service
```
```
[Unit]
Description=Alert Manager

[Service]
User=root
ExecStart=/opt/alertmanager-0.26.0.linux-amd64/alertmanager --config.file=/opt/alertmanager-0.26.0.linux-amd64/config.yml --web.external-url=http://10.13.13.10:9093/ --log.level=debug #IP controller

[Install]
WantedBy=default.target
```

5.Cek konfigurasi dan start AlertManager
```
# ./amtool check-config config.yml
# systemctl daemon-reload
# systemctl enable alert_manager.service
# systemctl start alert_manager.service
# systemctl status alert_manager.service
```
6.Buat alert untuk mendeteksi ketika intance mengalami **downtime**
```
# cd /opt/prometheus-2.48.0.linux-amd64/
```
```
# vim alerts_rules.yml 

groups:
- name: Instance.rules
  rules:
  - alert: node1-ganDown
    expr: up{instance="gan-node1", job="openstack-sd"} == 0
    for: 2m
    annotations:
      summary: "Instance node1-gan mati"
      description: "Instance 'node1-gan' has been down for more than 2 minute."
```

7.Tambahkan konfigurasi alerts di config.yml
```
# vim config.yml
```
```
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - 10.13.13.10:9093 #IP Controller

rule_files:
  - "alerts_rules.yml"
```

8.Cek konfigurasi dan restart prometheus server.
```
# ./promtool check config config.yml
# systemctl restart prometheus_server
```

9.Cek Alerts apakah sudah terbuat.
```
http://10.13.13.10:9090/config
http://10.13.13.10:9090/rules
http://10.13.13.10:9090/alerts
http://10.13.13.10:9093/#/status
```
![alerting](/images/alerting-7.png)

---

# Tes Alerting
Karena alert sudah terbuat, sekarang saatnya melakukan pengujiannya nih, kita bisa ngecek apakah alert sesuai yang diinginkan dengan mematikan sebuah Instance.

1.Matikan Instance Openstack.
![alerting](/images/matiin-instance.png)

2.Tunggu hingga status alert menjadi **pending** lalu **Firing**.
![alerting](/images/matiin-instance-1.png)

![alerting](/images/matiin-instance-2.png)

3.Periksa notif Alert.
  * Gmail
  ![alerting](/images/matiin-instance-3.png)

  * Discord
  ![alerting](/images/matiin-instance-4.png)

  * Slack
  ![alerting](/images/matiin-instance-5.png)

4.Jika ingin tes notif resolve, bisa tinggal nyalakan kembali instance dan liat notifikasi yang didapat.
![alerting](/images/matiin-instance-6.png)

