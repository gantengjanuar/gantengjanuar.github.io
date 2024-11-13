---
title: 'How To Deploy OpenStack With Kolla-Ansible'
date: 2024-11-10
permalink: /posts/2024/11/deploy-openstack/
tags:
  - Openstack
  - Deployment
  - Tutorial
---

![kolla openstack](/images/kolla-openstack.png)

# **OpenStack Deployment With Kolla-Ansible**
Hallo semuanya!! Disini saya akan memberikan langkah-langkah deploy openstack menggunakan Kolla-Ansible.
Sebelum masuk ke langkah langkah, kita harus kenalan dulu nih sama yang namanya Openstack dan Kolla ansible, sebenernya kedua itu apa sih?


## OpenStack

OpenStack adalah platform komputasi awan (cloud computing) open-source yang dirancang untuk membangun dan mengelola cloud publik atau privat. OpenStack menyediakan infrastruktur sebagai layanan (IaaS) dengan menyediakan berbagai layanan utama seperti:

**Nova** untuk pengelolaan komputasi (virtual machine instances).

**Neutron** untuk jaringan virtual.

**Cinder** untuk manajemen storage atau penyimpanan blok.

**Swift** untuk penyimpanan objek.

**Horizon** sebagai dashboard grafis bagi pengguna untuk mengelola resource.

**Keystone** untuk layanan autentikasi dan otorisasi.

OpenStack memungkinkan pengguna untuk mengelola resource komputasi, jaringan, dan storage dengan mudah dalam skala besar. Pengguna bisa melakukan otomatisasi pada penyebaran server, memantau kinerja, dan mengelola resource sesuai kebutuhan mereka.

## Kolla-Ansible
Kolla-Ansible adalah proyek open-source yang menyediakan alat untuk deployment (penyebaran) OpenStack menggunakan Docker container dan Ansible playbooks. Ini merupakan bagian dari ekosistem OpenStack Kolla, yang berfokus pada pengemasan layanan OpenStack ke dalam container untuk penyebaran yang lebih mudah dan konsisten. Beberapa fitur dan konsep penting dalam Kolla-Ansible adalah:

**Containerized OpenStack Services**: Kolla-Ansible menjalankan layanan OpenStack di dalam container Docker. Ini membuat setiap layanan (seperti Nova, Neutron, dan Cinder) berjalan secara terisolas.

**Ansible Playbooks**: Kolla-Ansible menggunakan playbook Ansible untuk mengotomatisasi penyebaran dan konfigurasi OpenStack. Playbook ini mencakup berbagai tugas, seperti instalasi, konfigurasi, dan manajemen layanan OpenStack di lingkungan yang telah ditentukan.

**Modular dan Fleksibel**: Kolla-Ansible mendukung berbagai arsitektur, seperti single-node (semua layanan di satu server) atau multi-node (layanan terdistribusi di beberapa server) untuk kebutuhan produksi yang besar.

Kolla-Ansible sangat memudahkan penyebaran OpenStack bagi perusahaan atau tim yang ingin memiliki lingkungan cloud OpenStack dengan konfigurasi cepat, efisien, dan stabil.

---
# Langkah pengerjaan / Implementasi

1.Install dependensi yang dibutuhkan.
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

---
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
---

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
---

11.Lakukan Instalasi openstackclient dan verifikasi Openstack
```
$ pip install openstackclient
$ source /etc/kolla/admin-openrc.sh
```
```
$ openstack service list 
```
> **Note:** Pastikan Service inti yang diinginkan sudah active.