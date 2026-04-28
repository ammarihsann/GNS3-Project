# 🧾 WRITE-UP

## OSPF + DHCP MikroTik (2 Router, 2 LAN)

---

# 🧩 1. Topologi

```
PC1/PC2 ── R1 ── R2 ── PC3/PC4
            │
        10.10.10.0/24 (link antar router)

LAN R1: 192.168.1.0/24  
LAN R2: 192.168.2.0/24
```

---

# ⚙️ 2. Konfigurasi IP Address

## 🔵 Router 1 (CHR-1)

```bash
/ip address add address=10.10.10.1/24 interface=ether3
/ip address add address=192.168.1.1/24 interface=ether2
```

---

## 🔴 Router 2 (CHR-2)

```bash
/ip address add address=10.10.10.2/24 interface=ether3
/ip address add address=192.168.2.1/24 interface=ether2
```

---

# 🌐 3. Setup DHCP Server

## 🔵 R1 (LAN 192.168.1.0)

```bash
/ip pool add name=pool1 ranges=192.168.1.10-192.168.1.100

/ip dhcp-server add name=dhcp1 interface=ether2 address-pool=pool1 disabled=no

/ip dhcp-server network add address=192.168.1.0/24 gateway=192.168.1.1 dns-server=8.8.8.8
```

---

## 🔴 R2 (LAN 192.168.2.0)

```bash
/ip pool add name=pool2 ranges=192.168.2.10-192.168.2.100

/ip dhcp-server add name=dhcp2 interface=ether2 address-pool=pool2 disabled=no

/ip dhcp-server network add address=192.168.2.0/24 gateway=192.168.2.1 dns-server=8.8.8.8
```

---

# 💻 4. Konfigurasi PC (VPCS)

Semua PC cukup:

```bash
ip dhcp
```

---

# 🔁 5. Setup OSPF

## 5.1 Buat OSPF Instance

### R1

```bash
/routing ospf instance add name=ospf1 router-id=1.1.1.1
```

### R2

```bash
/routing ospf instance add name=ospf1 router-id=2.2.2.2
```

---

## 5.2 Buat Area Backbone

Di kedua router:

```bash
/routing ospf area add name=backbone area-id=0.0.0.0 instance=ospf1
```

---

## 5.3 Masukkan Network ke OSPF

### 🔵 R1

```bash
/routing ospf interface-template add networks=10.10.10.0/24 area=backbone
/routing ospf interface-template add networks=192.168.1.0/24 area=backbone
```

---

### 🔴 R2

```bash
/routing ospf interface-template add networks=10.10.10.0/24 area=backbone
/routing ospf interface-template add networks=192.168.2.0/24 area=backbone
```

---

## ⚠️ Opsional tapi sangat disarankan (biar stabil)

Bind langsung ke interface:

### R1 & R2:

```bash
/routing ospf interface-template add interfaces=ether3 area=backbone
/routing ospf interface-template add interfaces=ether2 area=backbone
```

---

# 🔍 6. Verifikasi

## ✅ Cek Neighbor

```bash
/routing ospf neighbor print
```

Harus:

```
state=Full
```

---

## ✅ Cek Routing Table

```bash
/ip route print
```

Harus muncul:

### Di R1:

```
DAo 192.168.2.0/24 via 10.10.10.2
```

### Di R2:

```
DAo 192.168.1.0/24 via 10.10.10.1
```

---

## ✅ Cek DHCP Lease

```bash
/ip dhcp-server lease print
```

---

## ✅ Cek di PC

```bash
ip
```

Harus dapat:

* IP (192.168.x.x)
* Gateway
* DNS

---

# 🧪 7. Testing

## Dari Router

```bash
ping 192.168.2.1
ping 192.168.2.x
```

## Dari PC

```bash
ping 192.168.2.x
```

👉 Harus reply

---

# ⚠️ 8. Troubleshooting (INI WAJIB DIINGAT)

---

## ❌ 1. OSPF tidak connect

* cek IP antar router
* cek network OSPF

---

## ❌ 2. Neighbor Full tapi route tidak ada

* network tidak match interface
* interface belum masuk OSPF

---

## ❌ 3. Route ada tapi tidak bisa ping

👉 cek:

* DHCP
* gateway PC

---

## ❌ 4. PC tidak dapat IP

👉 cek:

```bash
/ip dhcp-server print
/ip dhcp-server lease print
```

---

## ❌ 5. IP PC hilang

👉 solusi:

```bash
ip dhcp
```

---

## ❌ 6. Salah fatal (yang tadi kamu alami)

> DHCP aktif tapi tidak dicek → PC tidak punya IP/gateway

---

# 🚫 9. Hal yang TIDAK BOLEH dilakukan

❌ Jangan masukkan network ini ke OSPF:

```
192.168.205.0/24 (cloud / mgmt)
```

👉 bisa bikin:

* double route
* jalur tidak optimal

---

# 🧠 10. Insight Penting

* OSPF = komunikasi antar router
* DHCP = komunikasi router ke client
* Kalau client gagal → bukan selalu salah routing

---

# 🚀 11. Backup Config

```bash
/export file=ospf-lab
```

---

# ⚡ 12. Cheat Sheet Singkat

```bash
# IP
/ip address add address=X.X.X.X/24 interface=etherX

# DHCP
/ip pool add name=pool ranges=IP-IP
/ip dhcp-server add interface=etherX address-pool=pool disabled=no
/ip dhcp-server network add address=X.X.X.0/24 gateway=X.X.X.1

# OSPF
/routing ospf instance add name=ospf1 router-id=X.X.X.X
/routing ospf area add name=backbone area-id=0.0.0.0 instance=ospf1
/routing ospf interface-template add networks=X.X.X.0/24 area=backbone

# cek
/routing ospf neighbor print
/ip route print
```

---

# 🎯 Kesimpulan

✔ OSPF berhasil → routing antar router otomatis
✔ DHCP berhasil → PC dapat IP & gateway otomatis
✔ Ping antar LAN berhasil

---

# Topology

<p align="center">
  <img src="Network Office Simulation.png" alt="Scan In Logo" width="350"/>
</p>
