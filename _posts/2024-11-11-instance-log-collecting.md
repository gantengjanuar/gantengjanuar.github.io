---
title: 'Openstack Instance Log and Metric Collection using ELK Stack'
date: 2024-11-11
permalink: /posts/2024/11/instance-log-collecting/
tags:
  - Openstack
  - ELK Stack
  - Documentation
---

![Openstack-ElkStack](/images/Openstack-ELkStack.png)

---
Hallo Semuanya! Blog ini merupakan dokumentasi sekaligus panduan lengkap mengenai cara mengumpulkan Log dan Metric dari sebuah Instance yang berada di Openstack Cluster. Disini, kita akan memanfaatkan Tools **ELK Stack** ( Filebeat, MetricBeat, Elasticsearch dan Kibana) untuk pengelolaan log dan metric yang cukup kompleks.

Selain itu, disini juga akan dijelaskan dari awal dari mulai Deploy Openstack menggunakan Kolla-Ansible hingga Monitoring Instance menggunakan Dashboard yang datanya diambil dari Log dan juga Metric dua Instance. Dengan menggunakan Dashboard, memungkinkan kita untuk Monitoring Instance secara efektif.

Oh ya, disini saya menggunakan 3 Virtual Machine, 1 untuk controller atau pilotnya, sedangkan duanya untuk compute

---
## Latar Belakang
Dalam era digital saat ini, manajemen infrastruktur TI yang efisien menjadi sangat penting bagi keberlangsungan operasi perusahaan. OpenStack, sebagai platform cloud computing yang open-source, memungkinkan perusahaan untuk membangun dan mengelola infrastruktur cloud secara fleksibel dan scalable.

Namun, dengan kompleksitas yang dihadapi dalam pengoperasian cluster OpenStack, tantangan muncul dalam hal pemantauan dan pengumpulan log dari setiap instance. Tim operasional sering kali dihadapkan pada kesulitan dalam mengakses dan menganalisis log dari masing-masing instance, yang dapat menghambat respons terhadap isu-isu yang mungkin terjadi. Untuk mengatasi tantangan ini, penting untuk memiliki sistem monitoring yang terpusat, yang tidak hanya mengumpulkan log tetapi juga memfasilitasi pengukuran metrik kinerja dan pengaturan alerting.

Dengan menerapkan alat seperti ElasticSearch, perusahaan dapat mengkonsolidasikan log dari berbagai instance ke dalam satu platform. Ini memungkinkan tim operasional untuk memantau kondisi sistem secara real-time, menganalisis data log dengan lebih efisien, dan mengambil tindakan yang diperlukan berdasarkan informasi yang diperoleh. Selain itu, penggunaan metrik dan alerting akan meningkatkan kemampuan pemantauan, memungkinkan deteksi masalah lebih awal dan pengambilan keputusan yang lebih cepat.

## Tools yang Digunakan:
* OpenStack	 	      - v7.2.1
* kolla-ansible 		- v2023.1
* Elasticsearch 		- v 8.15.3
* Kibana  		      - v 8.15.3
* Filebeat	      	-v 8.15.3
* Metricbeat		    -v 8.15.3

## Topologi Saya
![Topologi](/images/Topologi-1.png)

Dari Openstack Cluster, deploy 2 instance dan keduanya dipasangkan FIlebeat dan Metricbeat, log dan metric dari kedua instance dikirim ke logstash untuk parsing lalu ke elasticsearch untuk dikelola lebih lanjut. Pada elasticsearch, dipasangkan X-PACK untuk mengamankan koneksi serta menambah fitur alerting. Lalu log dan metrics akan di visualisasikan menggunakan Kibana serta dibuat Alerting agar monitoring lebih aman.

## Alur Kerja Saya
![Topologi](/images/Workflow-1.png)

---
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
Kolla-Ansible adalah proyek open-source yang menyediakan alat untuk deployment (penyebaran) OpenStack menggunakan Docker container dan Ansible playbooks. Ini merupakan bagian dari ekosistem OpenStack Kolla, yang berfokus pada pengemasan layanan OpenStack ke dalam container untuk penyebaran yang lebih mudah dan konsisten.

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


---
# Langkah Persiapan | Instalasi
Sebelum masuk ke pengerjaan, lakukan instalasi / deployment terkait tools yang nantinya kita akan gunakan.

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
ganteng-controller

[network]
ganteng-controller

[compute]
ganteng-compute1
ganteng-compute2

[monitoring]
ganteng-controller

[storage]
ganteng-controller
ganteng-compute1
ganteng-compute2

[deployment]
localhost ansible_connection=local
```


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

---
## buat Cinder-volume VG di all node
```
$sudo pvcreate /dev/vdb 
$ sudo vgcreate cinder-volumes /dev/vdb 
$ sudo vgs
```
> **Note:** jalankan step ini di all node 


10.Lakukan Booststrap , pre deploy , Deployment dan Post deploy.
```
$ kolla-ansible -i ./multinode bootstrap-servers
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

---
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

---
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

---
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

---
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


---
# Langkah Pengerjaan | Implementasi
Selesai melakukan tahap persiapan yaitu instalasi, barulah disini kita akan mulai launching 2 instance yang nantinya akan dikelola data log dan metrics nya untuk dibuat Dashboard untuk mengimplementasikan **Monitoring**.


## Launching Instance 1 dan 2
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
  --file ./focal-server-cloudimg-amd64.img focal-image-ubuntu

$ openstack image list
```

3.Buat external network serta subnetnya dan verifikasi.
```
$ openstack network create --share --external \
  --provider-physical-network physnet1 \
  --provider-network-type flat external-net-ganteng

$ openstack network list
```
```
$ openstack subnet create --network external-net-ganteng \
  --gateway 20.13.13.1 --no-dhcp \
  --subnet-range 20.13.13.0/24 external-subnet-ganteng

$ openstack subnet list
```

4.Buat Internal network serta subnetnya dan verifikasi.
```
$ openstack network create internal-net-ganteng

$ openstack subnet create --network internal-net-ganteng \
  --allocation-pool start=10.100.13.10,end=10.100.13.254 \
  --dns-nameserver 8.8.8.8 --gateway 10.100.13.1 \
  --subnet-range 10.100.13.0/24 internal-subnet-ganteng

$ openstack network list
$ openstack subnet list
```

5.Buat router dan verifikasi.
```
$ openstack router create router-ganteng
$ openstack router set --external-gateway external-net-ganteng router-ganteng
$ openstack router add subnet router-ganteng internal-subnet-ganteng

$ openstack router list
```

6.Buat security group yang mengizinkan protokol  ICMP dan TCP ssh dan verifikasi.
```
$ openstack security group create gan-security-group
$ openstack security group rule create --protocol icmp gan-security-group
$ openstack security group rule create --protocol tcp --ingress --dst-port 22 gan-security-group

$ openstack security group list
$ openstack security group rule list gan-security-group
```

7.Buat keypair dan verifikasi
```
$ openstack keypair create --public-key ~/.ssh/id_rsa.pub ubuntu-key

$ openstack keypair list
```

8.Buat flavor dan verifikasi.
```
$ openstack flavor create --ram 2048 --disk 8 --vcpus 1 --public mid-spec
$ openstack flavor create --ram 3072 --disk 8 --vcpus 1 --public mid-spec-2

$ openstack flavor list
```

9.Launching instance node1-gan
```
$ openstack server create --flavor mid-spec \
--image focal-image-ubuntu \
--key-name ubuntu-key \
--security-group gan-security-group \
--network internal-net-ganteng \
node1-gan
```

10.Pasang floating ip ke instance node1-gan dan verifikasi.
```
$ openstack floating ip create --floating-ip-address 20.13.13.34 external-net-ganteng
$ openstack server add floating ip nodel-gan 20.13.13.34

$ openstack server list
```

11.Coba akses instance node1-gan.
```
$ ssh -o 'PubkeyAcceptedKeyTypes +ssh-rsa' ubuntu@20.13.13.34
```

12. Launching instance node2-gan
```
$ openstack server create --flavor mid-spec-2 \
--image focal-image-ubuntu \
--key-name ubuntu-key \
--security-group gan-security-group \
--network internal-net-ganteng \
node2-gan
```

13.Pasang floating ip ke instance node2-gan dan verifikasi.
```
$ openstack floating ip create --floating-ip-address 20.13.13.119 external-net-ganteng
$ openstack server add floating ip node2-gan 20.13.13.119

$ openstack server list
```

14.Coba akses instance node2-gan.
```
$ ssh -o 'PubkeyAcceptedKeyTypes +ssh-rsa' ubuntu@20.13.13.119
```

## Install Filebeat di kedua Instance
1.Tambahkan Kunci GPG untuk Elastic.
```
$ wget -qo - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

2.Tambahkan repository Elastic ke source list
```
$ echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-8.x.list
```

3.Update dan install Filebeat.
```
$ sudo apt-get update && sudo apt-get upgrade
$ sudo apt-get install filebeat
```

4.Jalankan Filebeat dan verifikasi.
```
$ sudo systemctl enable filebeat
$ sudo systemctl start filebeat
$ sudo systemctl status filebeat
```

## Install Metricbeat di kedua Instance
1.Update dan install Metricbeat.
```
$ sudo apt-get update && sudo apt-get upgrade
$ sudo apt-get install metricbeat
```

2.Enable modul system agar metricbeat bisa mengambil metrics system.
```
$ sudo metricbeat modules enable system
```

3.Konfigurasi supaya bisa menarik metrik yang kita inginkan.
```
$ sudo nano /etc/metricbeat/modules.d/system.yml

- module: system
  period: 10s
  metricsets:
  - cpu
  - load
  - memory
  - network
  - process
  - process_summary
  - socket_summary
  - users
```

4.Jalankan dan verifikasi.
```
$ sudo systemctl enable metricbeat
$ sudo systemctl start metricbeat
$ sudo systemctl enable metricbeat
```

## Konfigurasi Filebeat, Metricbeat, dan Logstash
Selesai konfigurasi Tools tools yang akan digunakan, barulah kita akan mulai mengelola log dan metric dengan cara mengambilnya dari kedua instance, dengan cara:

1.Edit konfigurasi filebeat di node1-gan dan node2-gan
```
$ nano /etc/filebeat/filebeat.yml

filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/syslog
    fields:
    log_name: syslog

  - type: log
    enabled: true
    paths:
      - /var/log/auth. log
    fields:
    log_name: auth_log

output. logstash:
hosts: ["10.13.13.10:5044"]
```

2.Edit konfigurasi metricbeat di node1-gan dan node2-gan
```
$ sudo nano /etc/metricbeat/metricbeat.yml

# Global configurations for Metricbeat
metricbeat.config.modules:
  path: ${path.config}/modules.d/ *. yml
  reload. enabled: false

path.data: /var/lib/metricbeat


output.elasticsearch:
  # Set Elasticsearch host
  hosts: ["http://10.13.13.10:9200"]
  username: "elastic"
  password: "elastic"

# Configuring ILM for data stream management
setup.ilm.enabled: true
setup. ilm.rollover_alias: "metricbeat-(node1/node2)"
setup.ilm.pattern: "{now/d}-000001"


# Add custom fields to help identify data from this instance
processors:
  - add_fields:
    target: '
    fields:
    instance_id: "(node1-gan/node2-gan)"


# Set up Metricbeat dashboard in Kibana (recommended for visualization)
setup.kibana:
host: "http://10.13.13.10:5601"
```

3.Edit konfigurasi logstash di vm ganteng-controller.
```
$ sudo nano /etc/logstash/conf.d/filter-syslog.conf
```
```
input {
  beats {
    port => 5044
  }
}

filter {
  if [type] == "syslog" {
    grok {
      match => { "message" => [
        "%{SYSLOGBASE} %{GREEDYDATA:syslog_message}"
      ] }
      # Jika parsing gagal, masukkan log ke tag "_grokparsefailure"
      tag_on_failure => ["_grokparsefailure"]
    }

    date {
      match => ["syslog_timestamp", "MMM d HH:mm:ss", "MMM dd HH:mm:ss"]
      timezone => "UTC"
    }

    # Field-field tambahan untuk Klarifikasi dan visualisasi
    mutate {
      rename => { "syslog_hostname" => "host.name" }
      rename => { "syslog_program" => "process.name" }
      rename => { "syslog_pid" => "process.pid" }
      rename => { "syslog_message" => "message" }

      remove_field => ["syslog_timestamp"]
    }

    # Tentukan ECS-compatible fields
    mutate {
      add_field => { "[host][ip]" => "%{[host][ip]}" }
    }
  }

  # Auth log filter
  if [fields][log_name] == "auth_log" {
    grok {
      match => { "message" => "Invalid user %{USERNAME:ssh.invalid_user} from %{IP:ssh.client_ip}" }
      add_tag => ["ssh_fail", "auth_log"]
    }

    if [fields][log_name] == "auth_log" and "_grokparsefailure" in [tags] {
      drop { }
    }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    manage_template => false
    index => "%{[fields][log_name]}-%{[agent][name]}-%{+YYYY.MM}"
    user => "elastic"
    password => "elastic"
  }
}
```

4.Restart filebeat, metricbeat, logstash, dan elasticsearch
```
$ sudo systemctl restart filebeat
$ sudo systemctl restart metricbeat
$ sudo systemctl restart logstash
$ sudo systemctl restart elasticsearch
```

5.Verifikasi bahwa semua service running tanpa masalah.
```
$ sudo systemctl status filebeat
$ sudo systemctl status metricbeat
$ sudo systemctl status logstash
$ sudo systemctl status elasticsearch
```

6.Verifikasi index sudah terbuat.
```
$ curl -X GET -u elastic:elastic 10.13.13.10:9200/_cat/indices?v
```

---
## Buat Data Views
Setelah index terbuat, buat data view yang nantinya akan menjadi sumber / source data untuk kita buat Visualisasi

1.Login ke Kibana lalu buka **side bar > stack management > Data views**.

2.Buat data view dengan cara klik **Create Data View** dan mulai pembuatan, data view yang dibutuhkan: 
![Dataview1](/images/dataview.png)

* Name : Syslog-node1   | Index pattern: syslog_node1* | Timestamp field: @timestamp
* Name : Syslog-node2   | Index pattern: syslog_node2* | Timestamp field: @timestamp
* Name : Auth-node1  | Index pattern: auth_log_node1* | Timestamp field: @timestamp
* Name : Auth-node2  | Index pattern: auth_log_node2* | Timestamp field: @timestamp
* Name : Syslog dan Auth-node1  | Index pattern: syslog_node1*, auth_log_node1* | Timestamp field: @timestamp
* Name : Syslog dan Auth-node2  | Index pattern: syslog_node2*, auth_log_node2* | Timestamp field: @timestamp
* Name : Metrics-node 1&2  | Index pattern: metricbeat* | Timestamp field: @timestamp

2.Verifikasi Data views yang sudah terbuat.
![Dataview2](/images/dataview-2.png)

---
## Buat Visualisasi Dashboard Node1
Sekarang kita akan membuat Visualisasi untuk Dashboard node 1 yang nantinya akan digunakan untuk Monitoring Instance node1-gan
1.Buka **Sidebar > Dashboard > create dashboard**.
![Visualisasi](/images/visualisasi1.png)

2.Untuk membuat visualisasi, klik **Create visualization**.
> **Note:** Disini saya membuat banyak Visualisasi, caranya mirip namun ketentuannya berbeda.

3.Buat Visualisasi yang akan digunakan untuk monitoring Instance node1-gan, ketentuan Visualisasinya:
- Visualisasi: Hostname  
  - Data source	        	: Syslog-node1
  - Fields		          	: host.name.keyword
  - Visualization Type	  : Tag cloud

---
- Visualisasi: Total RAM
  - Data source           : Metrics node 1&2
  - Fields                : system.memory.total
  - Visualization Type    : Legacy Metrics
  - Functions             : Median
  - Filter by             : instance_id: "node1-gan"
  - Name                  : Total RAM
  - Value format          : Bytes (1024)

---
- Visualisasi: RAM Used
  - Data source           : Metrics node 1&2
  - Metric Fields         : system.memory.actual.used
  - Maximum Value fields  : system.memory.total
  - Visualization Type    : Semi-circular Gauge
  - Functions             : Last value
  - Name                  : RAM Used
  - Value format          : Bytes (1024)

---
- Visualisasi: RAM Free
  - Data source           : Metrics node 1&2
  - Fields                : system.memory.actual.free
  - Visualization Type    : Legacy Metrics
  - Functions             : Last value
  - Filter by             : instance_id: "node1-gan"
  - Name                  : RAM Free
  - Value format          : Bytes (1024)

---
- Visualisasi: Total Disk
  - Data source           : Metrics node 1&2
  - Fields                : system.fsstat.total_size.total
  - Visualization Type    : Legacy Metrics
  - Functions             : Last value
  - Filter by             : instance_id: "node1-gan"
  - Name                  : Node 1 Disk Total
  - Value format          : Bytes (1024)

---
- Visualisasi: Disk Free
  - Data source           : Metrics node 1&2
  - Fields                : system.fsstat.total_size.free
  - Visualization Type    : Legacy Metrics
  - Functions             : Last value
  - Filter by             : "system.fsstat.total_size.free": * and instance_id : "node1-gan"
  - Name                  : Total Disk Free
  - Value format          : Bytes (1024)

---
- Visualisasi: Disk Used
  - Data source           : Metrics node 1&2
  - Metrics Fields        : system.fsstat.total_size.used
  - Maximum Field         : system.fsstat.total_size.total
  - Visualization Type    : Legacy Metrics
  - Functions             : Last value
  - Filter by             : "system.fsstat.total_size.used": * and instance_id : "node1-gan"
  - Name                  : Total Disk Used
  - Value format          : Bytes (1024)

---
- Visualisasi: Event Syslog
  - Data source           : Syslog-node1
  - Fields                : event.original.keyword
  - Visualization Type    : Table
  - Name Rows             : Syslog Event

---
- Visualisasi: Log Types
  - Data source           : Syslog dan Auth-node1
  - Fields                : log.file.path_keyword
  - Visualization Type    : Table
  - Name Rows             : Log Types in Node01

---
- Visualisasi: SSH Invalid User
  - Data source           : Auth-node1
  - Fields                : ssh_invalid_user
  - Visualization Type    : Table
  - Rows 1                : @timestamp
  - Rows 2                : Top values of ssh_invalid_user.keyword
  - Number of values      : 10
  - Metrics               : Count of ssh_invalid_user.keyword

---
- Visualisasi: CPU Usage
  - Data source           : Metrics node 1&2
  - Metrics Fields        : system.fsstat.total_size.used
  - Maximum Field         : system.fsstat.total_size.total
  - Visualization Type    : Horizontal Bullet
  - Functions             : Last value
  - Filter by             : "system.cpu.total.pct": * and instance_id : "node1-gan"
  - Name                  : CPU Usage
  - Value format          : Percent

4.Selesai membuat visualisasi tersebut, rapihkan dan **Save** Dashboard.
![Dashboard](/images/Dashboard-1.png)

---
## Buat Dashboard Node2
1.Buat Dashboard dengan nama “Dashboard-node2” dan save.

2.Buka Dashboard yang sudah dibuat sebelumnya.

3.Klik **... pada visualisasi Hostname > more >**
![Dashboard](/images/dashboard2-1.png)

4.Klik **Copy to Dashboard > Existing dashboard > Dashboard-Node2 >Copy and go to dashboard**.
![Dashboard](/images/dashboard2-2.png)

![Dashboard](/images/dashboard2-3.png)

5.Kembali lagi ke dashboard sebelumnya, lalu lakukan hal yang sama ke semua visualisasi yang sudah dibuat sebelumnya.

6.Pastikan seluruh visualisasi yang dibuat sudah dicopy ke Dashboard Node2

7.Ganti Data view pada seluruh visualisasi agar menampilkan node2
- Semua visualisasi yang menggunakan Data View / source **Auth-node1** ubah menjadi > **Auth-node2
- Semua visualisasi yang menggunakan Data View / source **Syslog-node1** ubah menjadi > **Syslog-node2**
- Semua visualisasi yang menggunakan Data View / source **Metrics-node** 1 & 2 ubah **filter by** menjadi > **Instance_id : node2-gan**

8.Setelah semua selesai diubah, rapihkan Dashboard dan klik **Save**.
![Dashboard Final](/images/dashboard2-final.png)

---
## Buat Alerting 
Setelah membuat dua Dashboard untuk memonitoring kedua Instance, tentu saja ga lengkap kalo kita ga ngebuat Alerting yang dapat membantu untuk mendeteksi masalah lebih dini, gimana caranya? gini:

1.Di node controller, buat encryption key yang berisi 32 karakter dan tambahkan kunci x-pack yang sudah dibuat ke konfigurasi kibana.
```
$ sudo nano /etc/kibana/kibana.yml
```
```
xpack.encryptedSavedObjects.encryptionkey: "2qlWjw69B8xKr8Gof4KNmN8JSGWqWEIF"
```

2.Buka **Side bar > observabillity > alerts > manage rules > create rules**.
![Alert](/images/alert-1.png)

3.Pilih **Metrics Threshold**
![Alert](/images/alert-2.png)

4.Buat Alert untuk kedua Instance, ketentuannya:
- Name                : CPU USAGE node1-gan
  - Tags              : node1-gan, cpu-usage
  - Conditions        : When Average of system.cpu.total.pct is above 80% Alert, is above 70% Warning for the last 2 minutes
  - Filter            : instance_id : "node1-gan"
  - Check Every       : 1 minute
  - Action            : Server Log
  - **Save Rules**

---
- Name                : CPU USAGE node2-gan
  - Tags              : node2-gan, cpu-usage
  - Conditions        : When Average of system.cpu.total.pct is above 80% Alert, is above 70% Warning for the last 2 minutes
  - Filter            : instance_id : "node2-gan"
  - Check Every       : 1 minute
  - Action            : Server Log
  - **Save Rules**

---
- Name                : RAM USAGE node1-gan
  - Tags              : node1-gan, ram-usage
  - Conditions        : When Average of system.memory.actual.used.pct is above 70% Alert, is above 60% Warning for the last 2 minutes
  - Filter            : instance_id : "node1-gan"
  - Check Every       : 1 minute
  - Action            : Server Log
  - **Save Rules**

---
- Name                : RAM USAGE node2-gan
  - Tags              : node2-gan, ram-usage
  - Conditions        : When Average of system.memory.actual.used.pct is above 70% Alert, is above 60% Warning for the last 2 minutes
  - Filter            : instance_id : "node2-gan"
  - Check Every       : 1 minute
  - Action            : Server Log (Tambahkan connector bila tidak ada)
  - **Save Rules**

---
- Name                : DISK USAGE node1-gan
  - Tags              : node1-gan, disk-usage
  - Conditions        : When Average of system.filesystem.used.pct is above 85% Alert, is above 80% Warning for the last 2 minutes
  - Filter            : instance_id : "node1-gan"
  - Check Every       : 1 minute
  - Action            : Server Log
  - **Save Rules**

---
- Name                : DISK USAGE node2-gan
  - Tags              : node2-gan, disk-usage
  - Conditions        : When Average of system.filesystem.used.pct is above 85% Alert, is above 80% Warning for the last 2 minutes
  - Filter            : instance_id : "node2-gan"
  - Check Every       : 1 minute
  - Action            : Server Log
  - **Save Rules**

---
- Name                : INVALID SSH node1-gan
  - Tags              : node1-gan, invalid-ssh
  - Data view         : Auth-node1
  - Aggregation       : ssh_invalid_user.keyword : *
  - Threshold         : equation A, IS Above 25, for the last 30 days
  - Action            : Server Log
  - **Save Rules**

---
- Name                : INVALID SSH node2-gan
  - Tags              : node2-gan, invalid-ssh
  - Data view         : Auth-node2
  - Aggregation       : ssh_invalid_user.keyword : *
  - Threshold         : equation A, IS Above 25, for the last 30 days
  - Action            : Server Log
  - **Save Rules**

5.Verifikasi bahwa semua alert sudah terbuat.
![Alert](/images/alert-final.png)

---
# Verifikasi Hasil Pengerjaan
Sekarang semua sudah terbuat, kita sudah berhasil melakukan:
- **Openstack**
  - Deploy Openstack menggunakan Kolla-Ansible
  - Membuat serta launching dua Instance, yaitu **node1-gan** dan **node2-gan**

---
- **ELK Stack**
  - Instalasi Logstash
  - Instalasi Elasticsearch
  - Instalasi Kibana
  - Menerapkan Xpack 
  - Instalasi Filebeat pada kedua Instance
  - Instalasi Metricbeat pada kedua Instance
  - Mengirim log **syslog** dan **Authlog** untuk diproses
  - Membuat Data view dan memvisualisasikannya
  - Membuat dua Dashboard untuk monitoring kedua Instance
  - Membuat Alert untuk kedua Instance

---
# Hasil Akhir Project

## Dashboard | node1-gan
![hasil Akhir](/images/Dashboard-1.png)

---
## Dashboard | node2-gan
![hasil Akhir](/images/dashboard2-final.png)

## Alerting | node1-gan & node2-gan
![Alert](/images/alert-final.png)


---
## Referensi
* [Openstack](https://docs.openstack.org/2024.2/)

* [Elasticsearch, Instalasi Filebeat](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation-configuration.html)

* [Medium, Install and Enables Modules Metricbeat](https://medium.com/@luisalbertotaveras9/how-to-install-metricbeat-and-enable-their-modules-4bf242b61ed5)

* [Btech.id, Openstack](https://btech.id/en/news/layanan-layanan-yang-ada-di-openstack/)

* [Btech.id,  Elastic](https://btech.id/en/news/elastic-definisi-fungsi-dan-keuntungannya/)

* [Youtube, How to Configure x-pack](https://youtu.be/E-kwK88Vxzk)

* [revou, Apa itu Log?](https://revou.co/kosakata/log)

* [pypi.org, Kolla-Ansible](https://pypi.org/project/kolla-ansible/)