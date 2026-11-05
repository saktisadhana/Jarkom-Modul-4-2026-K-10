# Jarkom-Modul-4-2026-K-10

## Identitas Praktikan

| Nama                     | NRP        | Kelas |
| ------------------------ | ---------- | ----- |
| Putu Putra Sakti Sadhana | 5027251101 | B     |
| Danish Faeyza Rusmawan   | 5027251028 | B     |
## Reporting

Pada modul ini Lain membangun ulang The Wired dalam skala besar. Hanya Cyberia yang terhubung ke Cloud, lalu semua lalu lintas lewat Wired dan bercabang ke Navi, Tachibana, dan segmen bersama milik Psyche, Copland, dan Schumann. Di ujung paling dalam ada Tachibana, Deus, Protocol7, Knights, Phantoma, dan Shibuya.

Sesuai ketentuan soal, Cisco Packet Tracer (CPT) memakai CIDR dan GNS3 memakai VLSM. README ini membahas bagian CPT. Prefix IP kelompok K10 adalah `192.216`, sama seperti modul sebelumnya.

### Topologi

Topologi CPT terdiri dari 13 router 2811, 13 Switch-PT, 25 node, dan 1 Cloud. Jumlah host setiap node mengikuti gambar topologi dari soal.

<!-- Screenshot: topologi CPT lengkap dengan lingkaran CIDR, simpan di Asset/topologi - cpt.png (belum diambil) -->

Router 2811 bawaan hanya punya 2 port FastEthernet. Router yang butuh 3 atau 4 port (Cyberia, Wired, Navi, Copland, Schumann, Tachibana, Deus, Protocol7, Knights, Phantoma) kami beri modul `NM-2E2W`, sehingga ada tambahan port `Ethernet1/0` dan `Ethernet1/1`. Psyche, Accela, dan Shibuya cukup memakai 2 port bawaan.

Kabel router ke switch dan switch ke node memakai straight-through. Kabel router ke router dan router ke node tanpa switch (Juri, Akashic, Eiri) memakai cross-over. Cyberia terhubung ke Cloud0 lewat `FastEthernet0/0`.

### Perhitungan CIDR
#### Brief Overview

Menghitung kebutuhan IP setiap subnet, menggabungkan subnet dari router terdalam sampai Cyberia, lalu membagi IP dari blok gabungan terbesar ke bawah.

#### Step by Step

##### Menentukan subnet dan jumlah IP

Dalam step ini kami menamai setiap subnet A1 sampai A25 dan menghitung jumlah IP-nya. Sesuai soal, jumlah IP setiap subnet adalah jumlah host ditambah jumlah interface router di subnet itu. Netmask dipilih dari blok terkecil yang muat jumlah IP ditambah 2, yaitu network ID dan broadcast.

	A1 = JJ (95) + MyuMyu (30) + interface Cyberia (1) = 126 IP
	126 + 2 = 128, jadi netmask /25

Berfungsi sebagai contoh perhitungan satu subnet. Link antar router selalu berisi 2 IP (dua interface router), sehingga netmask-nya /30. Segmen Switch2 berisi 4 interface router (Wired, Psyche, Copland, Schumann), sehingga netmask-nya /29.

| Nama Subnet | Rute | Jumlah IP | Netmask |
| --- | --- | --- | --- |
| A1 | Cyberia > Switch1 > JJ, MyuMyu | 126 | /25 |
| A2 | Cyberia > Wired | 2 | /30 |
| A3 | Wired > Switch2 > Psyche, Copland, Schumann | 4 | /29 |
| A4 | Wired > Navi | 2 | /30 |
| A5 | Wired > Tachibana | 2 | /30 |
| A6 | Psyche > Switch3 > Karl, Lin | 881 | /22 |
| A7 | Copland > Switch4 > Masayuki | 301 | /23 |
| A8 | Copland > Juri | 32 | /26 |
| A9 | Schumann > Akashic | 4 | /29 |
| A10 | Schumann > Accela | 2 | /30 |
| A11 | Accela > Switch5 > Omnipresence, Denpa | 514 | /22 |
| A12 | Navi > Switch6 > Alice, Reika | 254 | /24 |
| A13 | Navi > Switch7 > Taro | 58 | /26 |
| A14 | Navi > Switch8 > Yasuo, Miho, Kuma | 143 | /24 |
| A15 | Tachibana > Switch9 > Collective, Ghost | 267 | /23 |
| A16 | Tachibana > Deus | 2 | /30 |
| A17 | Deus > Switch10 > Lilith, Specter | 99 | /25 |
| A18 | Deus > Protocol7 | 2 | /30 |
| A19 | Protocol7 > Switch11 > Chisa, Hodgeson | 33 | /26 |
| A20 | Protocol7 > Knights | 2 | /30 |
| A21 | Knights > Eiri | 3 | /29 |
| A22 | Knights > Phantoma | 2 | /30 |
| A23 | Phantoma > Switch12 > Semi | 15 | /27 |
| A24 | Phantoma > Shibuya | 2 | /30 |
| A25 | Shibuya > Switch13 > Lain, Mika | 62 | /26 |
| **Total** |  | **2814** | **/20** |

Total kebutuhan IP adalah 2814, sehingga secara jumlah seluruh jaringan cukup dengan /20. Hasil akhir CIDR tetap lebih besar dari itu karena setiap penggabungan membulatkan blok ke atas.

##### Penggabungan subnet

Dalam step ini kami menggabungkan subnet mulai dari router yang paling jauh dari Cyberia, yaitu Shibuya, lalu naik satu per satu sampai Cyberia. Setiap penggabungan mengambil dua subnet dan menghasilkan satu subnet baru.

	Netmask hasil = netmask terbesar dari kedua subnet - 1
	B1 = A25 (/26) + A24 (/30) = /25

Berfungsi untuk menentukan ukuran blok gabungan. Blok /25 cukup menampung dua blok di dalamnya, yaitu satu /26 dan satu /30.

Urutan penggabungan mengikuti pohon router. Di router yang punya banyak subnet, subnet yang kecil kami gabung lebih dulu supaya blok hasilnya tidak membesar tanpa perlu. Di Wired, domain yang letaknya berdekatan kami gabung lebih dulu: Psyche dengan Copland (C4), lalu Switch2 dengan Navi (D2). Ukuran akhir Wired tetap /18, dan urutan ini membuat lingkaran CIDR di CPT lebih mudah dibaca.

**Level I**

| Subnet | Subnet 1 | Netmask 1 | Subnet 2 | Netmask 2 | Netmask Akhir |
| --- | --- | --- | --- | --- | --- |
| B1 | A25 | /26 | A24 | /30 | /25 |
| B2 | A23 | /27 | A22 | /30 | /26 |
| B3 | A21 | /29 | A20 | /30 | /28 |
| B4 | A19 | /26 | A18 | /30 | /25 |
| B5 | A17 | /25 | A16 | /30 | /24 |
| B6 | A15 | /23 | A5 | /30 | /22 |
| B7 | A13 | /26 | A4 | /30 | /25 |
| B8 | A12 | /24 | A14 | /24 | /23 |
| B9 | A11 | /22 | A10 | /30 | /21 |
| B10 | A7 | /23 | A8 | /26 | /22 |
| B11 | A3 | /29 | A2 | /30 | /28 |

**Level II**

| Subnet | Subnet 1 | Netmask 1 | Subnet 2 | Netmask 2 | Netmask Akhir |
| --- | --- | --- | --- | --- | --- |
| C1 | B2 | /26 | B1 | /25 | /24 |
| C2 | B7 | /25 | B8 | /23 | /22 |
| C3 | A9 | /29 | B9 | /21 | /20 |
| C4 | A6 | /22 | B10 | /22 | /21 |

**Level III**

| Subnet | Subnet 1 | Netmask 1 | Subnet 2 | Netmask 2 | Netmask Akhir |
| --- | --- | --- | --- | --- | --- |
| D1 | B3 | /28 | C1 | /24 | /23 |
| D2 | B11 | /28 | C2 | /22 | /21 |

**Level IV**

| Subnet | Subnet 1 | Netmask 1 | Subnet 2 | Netmask 2 | Netmask Akhir |
| --- | --- | --- | --- | --- | --- |
| E1 | B4 | /25 | D1 | /23 | /22 |
| E2 | C4 | /21 | D2 | /21 | /20 |

**Level V**

| Subnet | Subnet 1 | Netmask 1 | Subnet 2 | Netmask 2 | Netmask Akhir |
| --- | --- | --- | --- | --- | --- |
| F1 | B5 | /24 | E1 | /22 | /21 |
| F2 | C3 | /20 | E2 | /20 | /19 |

**Level VI**

| Subnet | Subnet 1 | Netmask 1 | Subnet 2 | Netmask 2 | Netmask Akhir |
| --- | --- | --- | --- | --- | --- |
| G1 | B6 | /22 | F1 | /21 | /20 |

**Level VII**

| Subnet | Subnet 1 | Netmask 1 | Subnet 2 | Netmask 2 | Netmask Akhir |
| --- | --- | --- | --- | --- | --- |
| H1 | F2 | /19 | G1 | /20 | /18 |

**Level VIII**

| Subnet | Subnet 1 | Netmask 1 | Subnet 2 | Netmask 2 | Netmask Akhir |
| --- | --- | --- | --- | --- | --- |
| I1 | A1 | /25 | H1 | /18 | /17 |

Hasil akhirnya adalah I1 dengan netmask /17. Artinya seluruh The Wired versi CPT memakai blok `192.216.0.0/17`.

##### Pembagian IP

Dalam step ini kami membagi IP dari atas ke bawah. Blok I1 dimulai di `192.216.0.0/17`. Setiap blok gabungan dibagi dua sama besar: bagian pertama untuk anak yang lebih besar, bagian kedua untuk anak yang lebih kecil. Pembagian diulang sampai semua subnet A mendapat alamat.

	I1 = 192.216.0.0/17
	    H1 (/18) = 192.216.0.0
	    A1 (/25) = 192.216.64.0

Berfungsi sebagai contoh langkah pertama. H1 lebih besar sehingga mendapat setengah pertama, sedangkan A1 mendapat awal setengah kedua.

| Subnet | Rute | Network ID | Netmask | Broadcast | Range IP |
| --- | --- | --- | --- | --- | --- |
| A1 | Cyberia > Switch1 > JJ, MyuMyu | 192.216.64.0 | 255.255.255.128 | 192.216.64.127 | 192.216.64.1 - 192.216.64.126 |
| A2 | Cyberia > Wired | 192.216.28.8 | 255.255.255.252 | 192.216.28.11 | 192.216.28.9 - 192.216.28.10 |
| A3 | Wired > Switch2 > Psyche, Copland, Schumann | 192.216.28.0 | 255.255.255.248 | 192.216.28.7 | 192.216.28.1 - 192.216.28.6 |
| A4 | Wired > Navi | 192.216.26.64 | 255.255.255.252 | 192.216.26.67 | 192.216.26.65 - 192.216.26.66 |
| A5 | Wired > Tachibana | 192.216.42.0 | 255.255.255.252 | 192.216.42.3 | 192.216.42.1 - 192.216.42.2 |
| A6 | Psyche > Switch3 > Karl, Lin | 192.216.16.0 | 255.255.252.0 | 192.216.19.255 | 192.216.16.1 - 192.216.19.254 |
| A7 | Copland > Switch4 > Masayuki | 192.216.20.0 | 255.255.254.0 | 192.216.21.255 | 192.216.20.1 - 192.216.21.254 |
| A8 | Copland > Juri | 192.216.22.0 | 255.255.255.192 | 192.216.22.63 | 192.216.22.1 - 192.216.22.62 |
| A9 | Schumann > Akashic | 192.216.8.0 | 255.255.255.248 | 192.216.8.7 | 192.216.8.1 - 192.216.8.6 |
| A10 | Schumann > Accela | 192.216.4.0 | 255.255.255.252 | 192.216.4.3 | 192.216.4.1 - 192.216.4.2 |
| A11 | Accela > Switch5 > Omnipresence, Denpa | 192.216.0.0 | 255.255.252.0 | 192.216.3.255 | 192.216.0.1 - 192.216.3.254 |
| A12 | Navi > Switch6 > Alice, Reika | 192.216.24.0 | 255.255.255.0 | 192.216.24.255 | 192.216.24.1 - 192.216.24.254 |
| A13 | Navi > Switch7 > Taro | 192.216.26.0 | 255.255.255.192 | 192.216.26.63 | 192.216.26.1 - 192.216.26.62 |
| A14 | Navi > Switch8 > Yasuo, Miho, Kuma | 192.216.25.0 | 255.255.255.0 | 192.216.25.255 | 192.216.25.1 - 192.216.25.254 |
| A15 | Tachibana > Switch9 > Collective, Ghost | 192.216.40.0 | 255.255.254.0 | 192.216.41.255 | 192.216.40.1 - 192.216.41.254 |
| A16 | Tachibana > Deus | 192.216.36.128 | 255.255.255.252 | 192.216.36.131 | 192.216.36.129 - 192.216.36.130 |
| A17 | Deus > Switch10 > Lilith, Specter | 192.216.36.0 | 255.255.255.128 | 192.216.36.127 | 192.216.36.1 - 192.216.36.126 |
| A18 | Deus > Protocol7 | 192.216.34.64 | 255.255.255.252 | 192.216.34.67 | 192.216.34.65 - 192.216.34.66 |
| A19 | Protocol7 > Switch11 > Chisa, Hodgeson | 192.216.34.0 | 255.255.255.192 | 192.216.34.63 | 192.216.34.1 - 192.216.34.62 |
| A20 | Protocol7 > Knights | 192.216.33.8 | 255.255.255.252 | 192.216.33.11 | 192.216.33.9 - 192.216.33.10 |
| A21 | Knights > Eiri | 192.216.33.0 | 255.255.255.248 | 192.216.33.7 | 192.216.33.1 - 192.216.33.6 |
| A22 | Knights > Phantoma | 192.216.32.160 | 255.255.255.252 | 192.216.32.163 | 192.216.32.161 - 192.216.32.162 |
| A23 | Phantoma > Switch12 > Semi | 192.216.32.128 | 255.255.255.224 | 192.216.32.159 | 192.216.32.129 - 192.216.32.158 |
| A24 | Phantoma > Shibuya | 192.216.32.64 | 255.255.255.252 | 192.216.32.67 | 192.216.32.65 - 192.216.32.66 |
| A25 | Shibuya > Switch13 > Lain, Mika | 192.216.32.0 | 255.255.255.192 | 192.216.32.63 | 192.216.32.1 - 192.216.32.62 |

Di setiap subnet, router yang lebih dekat ke Cyberia memakai IP pertama dan router di bawahnya memakai IP kedua. Di Switch2, Wired memakai `.1`, Psyche `.2`, Copland `.3`, dan Schumann `.4`. Node memakai IP setelah IP router, dan gateway-nya adalah IP router di subnet itu.

##### Lingkaran CIDR di CPT

Dalam step ini kami menggambar satu elips untuk setiap hasil penggabungan (B1 sampai I1) di logical view CPT. Setiap elips melingkupi semua perangkat di subnet yang digabung, sehingga lingkaran level atas selalu memuat lingkaran level di bawahnya. Label setiap lingkaran berisi nama, netmask, dan network ID, misalnya `B1 /25 - 192.216.32.0`.

<!-- Screenshot: lingkaran CIDR di CPT, simpan di Asset/lingkaran cidr - cpt.png (belum diambil) -->

Jalur Tachibana sampai Shibuya mengarah serong ke kanan bawah, sedangkan CPT hanya bisa menggambar elips tegak. Karena itu lingkaran C1 sampai G1 ikut menutupi area Navi, walaupun isinya tidak mencakup Navi.

### Konfigurasi Router
#### Brief Overview

Memberi IP ke setiap interface router dan menyusun routing statis memakai blok gabungan CIDR. Router cabang cukup memakai satu default route ke router di atasnya. Router di atasnya memegang satu rute untuk setiap domain di bawahnya.

#### Step by Step

##### `Cyberia` : interface dan routing

```
enable
configure terminal
hostname Cyberia
no ip domain-lookup
interface FastEthernet0/0
 description ke Cloud0 (NAT)
 ip address dhcp
 no shutdown
 exit
interface FastEthernet0/1
 description A1 Cyberia > Switch1 > JJ, MyuMyu
 ip address 192.216.64.1 255.255.255.128
 no shutdown
 exit
interface Ethernet1/0
 description A2 Cyberia > Wired
 ip address 192.216.28.9 255.255.255.252
 no shutdown
 exit
ip route 192.216.0.0 255.255.192.0 192.216.28.10
end
write memory
```

Dalam step ini kami memberi IP ke interface `Cyberia` dan menambahkan satu rute ke seluruh domain Wired.

	ip address dhcp

Berfungsi untuk meminta IP dari Cloud0. Di CPT, Cloud tidak memberi DHCP, jadi interface ini memang tidak mendapat IP. Hal ini tidak memengaruhi koneksi antar node di topologi. Sedangkan:

	ip route 192.216.0.0 255.255.192.0 192.216.28.10

Berfungsi untuk mengarahkan semua tujuan di blok H1 `/18` ke Wired. Cyberia tidak butuh rute lain karena Switch1 dan link ke Wired sudah terhubung langsung.

##### `Wired` : interface dan routing

```
enable
configure terminal
hostname Wired
no ip domain-lookup
interface FastEthernet0/0
 description A2 Cyberia > Wired
 ip address 192.216.28.10 255.255.255.252
 no shutdown
 exit
interface FastEthernet0/1
 description A3 Wired > Switch2 > Psyche, Copland, Schumann
 ip address 192.216.28.1 255.255.255.248
 no shutdown
 exit
interface Ethernet1/0
 description A4 Wired > Navi
 ip address 192.216.26.65 255.255.255.252
 no shutdown
 exit
interface Ethernet1/1
 description A5 Wired > Tachibana
 ip address 192.216.42.1 255.255.255.252
 no shutdown
 exit
ip route 0.0.0.0 0.0.0.0 192.216.28.9
ip route 192.216.16.0 255.255.252.0 192.216.28.2
ip route 192.216.20.0 255.255.252.0 192.216.28.3
ip route 192.216.0.0 255.255.240.0 192.216.28.4
ip route 192.216.24.0 255.255.252.0 192.216.26.66
ip route 192.216.32.0 255.255.240.0 192.216.42.2
end
write memory
```

Dalam step ini kami memberi IP ke empat interface `Wired` dan menambahkan rute ke lima domain di bawahnya.

	ip route 0.0.0.0 0.0.0.0 192.216.28.9
	ip route 192.216.16.0 255.255.252.0 192.216.28.2
	ip route 192.216.20.0 255.255.252.0 192.216.28.3
	ip route 192.216.0.0 255.255.240.0 192.216.28.4
	ip route 192.216.24.0 255.255.252.0 192.216.26.66
	ip route 192.216.32.0 255.255.240.0 192.216.42.2

Berfungsi untuk mengirim lalu lintas tanpa tujuan khusus ke Cyberia, lalu mengarahkan setiap domain ke router pemiliknya. Domain Psyche, Copland, dan Schumann dicapai lewat Switch2, domain Navi dan Tachibana lewat link masing-masing.

##### `Psyche` : interface dan routing

```
enable
configure terminal
hostname Psyche
no ip domain-lookup
interface FastEthernet0/0
 description A3 Wired > Switch2 > Psyche, Copland, Schumann
 ip address 192.216.28.2 255.255.255.248
 no shutdown
 exit
interface FastEthernet0/1
 description A6 Psyche > Switch3 > Karl, Lin
 ip address 192.216.16.1 255.255.252.0
 no shutdown
 exit
ip route 0.0.0.0 0.0.0.0 192.216.28.1
end
write memory
```

Dalam step ini kami memberi IP ke 2 interface `Psyche` dan menambahkan default route.

	ip route 0.0.0.0 0.0.0.0 192.216.28.1

Berfungsi untuk mengirim semua tujuan yang tidak terhubung langsung ke Wired.

##### `Copland` : interface dan routing

```
enable
configure terminal
hostname Copland
no ip domain-lookup
interface FastEthernet0/0
 description A3 Wired > Switch2 > Psyche, Copland, Schumann
 ip address 192.216.28.3 255.255.255.248
 no shutdown
 exit
interface FastEthernet0/1
 description A7 Copland > Switch4 > Masayuki
 ip address 192.216.20.1 255.255.254.0
 no shutdown
 exit
interface Ethernet1/0
 description A8 Copland > Juri
 ip address 192.216.22.1 255.255.255.192
 no shutdown
 exit
ip route 0.0.0.0 0.0.0.0 192.216.28.1
end
write memory
```

Dalam step ini kami memberi IP ke 3 interface `Copland` dan menambahkan default route.

	ip route 0.0.0.0 0.0.0.0 192.216.28.1

Berfungsi untuk mengirim semua tujuan yang tidak terhubung langsung ke Wired.

##### `Schumann` : interface dan routing

```
enable
configure terminal
hostname Schumann
no ip domain-lookup
interface FastEthernet0/0
 description A3 Wired > Switch2 > Psyche, Copland, Schumann
 ip address 192.216.28.4 255.255.255.248
 no shutdown
 exit
interface FastEthernet0/1
 description A9 Schumann > Akashic
 ip address 192.216.8.1 255.255.255.248
 no shutdown
 exit
interface Ethernet1/0
 description A10 Schumann > Accela
 ip address 192.216.4.1 255.255.255.252
 no shutdown
 exit
ip route 0.0.0.0 0.0.0.0 192.216.28.1
ip route 192.216.0.0 255.255.248.0 192.216.4.2
end
write memory
```

Dalam step ini kami memberi IP ke 3 interface `Schumann` dan menambahkan default route serta rute ke domain di bawahnya.

	ip route 0.0.0.0 0.0.0.0 192.216.28.1

Berfungsi untuk mengirim semua tujuan yang tidak terhubung langsung ke Wired. Sedangkan:

	ip route 192.216.0.0 255.255.248.0 192.216.4.2

Berfungsi untuk mengarahkan semua subnet di domain Accela (B9) lewat `Accela`. Karena memakai satu blok gabungan CIDR, seluruh domain itu cukup ditulis dalam satu baris.

##### `Accela` : interface dan routing

```
enable
configure terminal
hostname Accela
no ip domain-lookup
interface FastEthernet0/0
 description A10 Schumann > Accela
 ip address 192.216.4.2 255.255.255.252
 no shutdown
 exit
interface FastEthernet0/1
 description A11 Accela > Switch5 > Omnipresence, Denpa
 ip address 192.216.0.1 255.255.252.0
 no shutdown
 exit
ip route 0.0.0.0 0.0.0.0 192.216.4.1
end
write memory
```

Dalam step ini kami memberi IP ke 2 interface `Accela` dan menambahkan default route.

	ip route 0.0.0.0 0.0.0.0 192.216.4.1

Berfungsi untuk mengirim semua tujuan yang tidak terhubung langsung ke Schumann.

##### `Navi` : interface dan routing

```
enable
configure terminal
hostname Navi
no ip domain-lookup
interface FastEthernet0/0
 description A4 Wired > Navi
 ip address 192.216.26.66 255.255.255.252
 no shutdown
 exit
interface FastEthernet0/1
 description A12 Navi > Switch6 > Alice, Reika
 ip address 192.216.24.1 255.255.255.0
 no shutdown
 exit
interface Ethernet1/0
 description A13 Navi > Switch7 > Taro
 ip address 192.216.26.1 255.255.255.192
 no shutdown
 exit
interface Ethernet1/1
 description A14 Navi > Switch8 > Yasuo, Miho, Kuma
 ip address 192.216.25.1 255.255.255.0
 no shutdown
 exit
ip route 0.0.0.0 0.0.0.0 192.216.26.65
end
write memory
```

Dalam step ini kami memberi IP ke 4 interface `Navi` dan menambahkan default route.

	ip route 0.0.0.0 0.0.0.0 192.216.26.65

Berfungsi untuk mengirim semua tujuan yang tidak terhubung langsung ke Wired.

##### `Tachibana` : interface dan routing

```
enable
configure terminal
hostname Tachibana
no ip domain-lookup
interface FastEthernet0/0
 description A5 Wired > Tachibana
 ip address 192.216.42.2 255.255.255.252
 no shutdown
 exit
interface FastEthernet0/1
 description A15 Tachibana > Switch9 > Collective, Ghost
 ip address 192.216.40.1 255.255.254.0
 no shutdown
 exit
interface Ethernet1/0
 description A16 Tachibana > Deus
 ip address 192.216.36.129 255.255.255.252
 no shutdown
 exit
ip route 0.0.0.0 0.0.0.0 192.216.42.1
ip route 192.216.32.0 255.255.248.0 192.216.36.130
end
write memory
```

Dalam step ini kami memberi IP ke 3 interface `Tachibana` dan menambahkan default route serta rute ke domain di bawahnya.

	ip route 0.0.0.0 0.0.0.0 192.216.42.1

Berfungsi untuk mengirim semua tujuan yang tidak terhubung langsung ke Wired. Sedangkan:

	ip route 192.216.32.0 255.255.248.0 192.216.36.130

Berfungsi untuk mengarahkan semua subnet di domain Deus (F1) lewat `Deus`. Karena memakai satu blok gabungan CIDR, seluruh domain itu cukup ditulis dalam satu baris.

##### `Deus` : interface dan routing

```
enable
configure terminal
hostname Deus
no ip domain-lookup
interface FastEthernet0/0
 description A16 Tachibana > Deus
 ip address 192.216.36.130 255.255.255.252
 no shutdown
 exit
interface FastEthernet0/1
 description A17 Deus > Switch10 > Lilith, Specter
 ip address 192.216.36.1 255.255.255.128
 no shutdown
 exit
interface Ethernet1/0
 description A18 Deus > Protocol7
 ip address 192.216.34.65 255.255.255.252
 no shutdown
 exit
ip route 0.0.0.0 0.0.0.0 192.216.36.129
ip route 192.216.32.0 255.255.252.0 192.216.34.66
end
write memory
```

Dalam step ini kami memberi IP ke 3 interface `Deus` dan menambahkan default route serta rute ke domain di bawahnya.

	ip route 0.0.0.0 0.0.0.0 192.216.36.129

Berfungsi untuk mengirim semua tujuan yang tidak terhubung langsung ke Tachibana. Sedangkan:

	ip route 192.216.32.0 255.255.252.0 192.216.34.66

Berfungsi untuk mengarahkan semua subnet di domain Protocol7 (E1) lewat `Protocol7`. Karena memakai satu blok gabungan CIDR, seluruh domain itu cukup ditulis dalam satu baris.

##### `Protocol7` : interface dan routing

```
enable
configure terminal
hostname Protocol7
no ip domain-lookup
interface FastEthernet0/0
 description A18 Deus > Protocol7
 ip address 192.216.34.66 255.255.255.252
 no shutdown
 exit
interface FastEthernet0/1
 description A19 Protocol7 > Switch11 > Chisa, Hodgeson
 ip address 192.216.34.1 255.255.255.192
 no shutdown
 exit
interface Ethernet1/0
 description A20 Protocol7 > Knights
 ip address 192.216.33.9 255.255.255.252
 no shutdown
 exit
ip route 0.0.0.0 0.0.0.0 192.216.34.65
ip route 192.216.32.0 255.255.254.0 192.216.33.10
end
write memory
```

Dalam step ini kami memberi IP ke 3 interface `Protocol7` dan menambahkan default route serta rute ke domain di bawahnya.

	ip route 0.0.0.0 0.0.0.0 192.216.34.65

Berfungsi untuk mengirim semua tujuan yang tidak terhubung langsung ke Deus. Sedangkan:

	ip route 192.216.32.0 255.255.254.0 192.216.33.10

Berfungsi untuk mengarahkan semua subnet di domain Knights (D1) lewat `Knights`. Karena memakai satu blok gabungan CIDR, seluruh domain itu cukup ditulis dalam satu baris.

##### `Knights` : interface dan routing

```
enable
configure terminal
hostname Knights
no ip domain-lookup
interface FastEthernet0/0
 description A20 Protocol7 > Knights
 ip address 192.216.33.10 255.255.255.252
 no shutdown
 exit
interface FastEthernet0/1
 description A22 Knights > Phantoma
 ip address 192.216.32.161 255.255.255.252
 no shutdown
 exit
interface Ethernet1/0
 description A21 Knights > Eiri
 ip address 192.216.33.1 255.255.255.248
 no shutdown
 exit
ip route 0.0.0.0 0.0.0.0 192.216.33.9
ip route 192.216.32.0 255.255.255.0 192.216.32.162
end
write memory
```

Dalam step ini kami memberi IP ke 3 interface `Knights` dan menambahkan default route serta rute ke domain di bawahnya.

	ip route 0.0.0.0 0.0.0.0 192.216.33.9

Berfungsi untuk mengirim semua tujuan yang tidak terhubung langsung ke Protocol7. Sedangkan:

	ip route 192.216.32.0 255.255.255.0 192.216.32.162

Berfungsi untuk mengarahkan semua subnet di domain Phantoma (C1) lewat `Phantoma`. Karena memakai satu blok gabungan CIDR, seluruh domain itu cukup ditulis dalam satu baris.

##### `Phantoma` : interface dan routing

```
enable
configure terminal
hostname Phantoma
no ip domain-lookup
interface FastEthernet0/0
 description A22 Knights > Phantoma
 ip address 192.216.32.162 255.255.255.252
 no shutdown
 exit
interface FastEthernet0/1
 description A23 Phantoma > Switch12 > Semi
 ip address 192.216.32.129 255.255.255.224
 no shutdown
 exit
interface Ethernet1/0
 description A24 Phantoma > Shibuya
 ip address 192.216.32.65 255.255.255.252
 no shutdown
 exit
ip route 0.0.0.0 0.0.0.0 192.216.32.161
ip route 192.216.32.0 255.255.255.128 192.216.32.66
end
write memory
```

Dalam step ini kami memberi IP ke 3 interface `Phantoma` dan menambahkan default route serta rute ke domain di bawahnya.

	ip route 0.0.0.0 0.0.0.0 192.216.32.161

Berfungsi untuk mengirim semua tujuan yang tidak terhubung langsung ke Knights. Sedangkan:

	ip route 192.216.32.0 255.255.255.128 192.216.32.66

Berfungsi untuk mengarahkan semua subnet di domain Shibuya (B1) lewat `Shibuya`. Karena memakai satu blok gabungan CIDR, seluruh domain itu cukup ditulis dalam satu baris.

##### `Shibuya` : interface dan routing

```
enable
configure terminal
hostname Shibuya
no ip domain-lookup
interface FastEthernet0/0
 description A24 Phantoma > Shibuya
 ip address 192.216.32.66 255.255.255.252
 no shutdown
 exit
interface FastEthernet0/1
 description A25 Shibuya > Switch13 > Lain, Mika
 ip address 192.216.32.1 255.255.255.192
 no shutdown
 exit
ip route 0.0.0.0 0.0.0.0 192.216.32.65
end
write memory
```

Dalam step ini kami memberi IP ke 2 interface `Shibuya` dan menambahkan default route.

	ip route 0.0.0.0 0.0.0.0 192.216.32.65

Berfungsi untuk mengirim semua tujuan yang tidak terhubung langsung ke Phantoma.

##### Switch

Semua Switch-PT memakai konfigurasi bawaan. Yang kami ubah hanya hostname, sesuai nama di topologi.

### Konfigurasi Node
#### Brief Overview

Memberi IP statis, netmask, dan gateway ke setiap node lewat Desktop > IP Configuration.

#### Step by Step

##### Semua node : IP statis

| Node | Subnet | IP | Netmask | Gateway |
| --- | --- | --- | --- | --- |
| JJ (95 host) | A1 | 192.216.64.2 | 255.255.255.128 | 192.216.64.1 |
| MyuMyu (30 host) | A1 | 192.216.64.3 | 255.255.255.128 | 192.216.64.1 |
| Karl (700 host) | A6 | 192.216.16.2 | 255.255.252.0 | 192.216.16.1 |
| Lin (180 host) | A6 | 192.216.16.3 | 255.255.252.0 | 192.216.16.1 |
| Masayuki (300 host) | A7 | 192.216.20.2 | 255.255.254.0 | 192.216.20.1 |
| Omnipresence (510 host) | A11 | 192.216.0.2 | 255.255.252.0 | 192.216.0.1 |
| Denpa (3 host) | A11 | 192.216.0.3 | 255.255.252.0 | 192.216.0.1 |
| Alice (208 host) | A12 | 192.216.24.2 | 255.255.255.0 | 192.216.24.1 |
| Reika (45 host) | A12 | 192.216.24.3 | 255.255.255.0 | 192.216.24.1 |
| Taro (57 host) | A13 | 192.216.26.2 | 255.255.255.192 | 192.216.26.1 |
| Yasuo (120 host) | A14 | 192.216.25.2 | 255.255.255.0 | 192.216.25.1 |
| Miho (18 host) | A14 | 192.216.25.3 | 255.255.255.0 | 192.216.25.1 |
| Kuma (4 host) | A14 | 192.216.25.4 | 255.255.255.0 | 192.216.25.1 |
| Collective (260 host) | A15 | 192.216.40.2 | 255.255.254.0 | 192.216.40.1 |
| Ghost (6 host) | A15 | 192.216.40.3 | 255.255.254.0 | 192.216.40.1 |
| Lilith (77 host) | A17 | 192.216.36.2 | 255.255.255.128 | 192.216.36.1 |
| Specter (21 host) | A17 | 192.216.36.3 | 255.255.255.128 | 192.216.36.1 |
| Chisa (23 host) | A19 | 192.216.34.2 | 255.255.255.192 | 192.216.34.1 |
| Hodgeson (9 host) | A19 | 192.216.34.3 | 255.255.255.192 | 192.216.34.1 |
| Semi (14 host) | A23 | 192.216.32.130 | 255.255.255.224 | 192.216.32.129 |
| Lain (57 host) | A25 | 192.216.32.2 | 255.255.255.192 | 192.216.32.1 |
| Mika (4 host) | A25 | 192.216.32.3 | 255.255.255.192 | 192.216.32.1 |
| Juri (31 host) | A8 | 192.216.22.2 | 255.255.255.192 | 192.216.22.1 |
| Akashic (3 host) | A9 | 192.216.8.2 | 255.255.255.248 | 192.216.8.1 |
| Eiri (2 host) | A21 | 192.216.33.2 | 255.255.255.248 | 192.216.33.1 |

Dalam step ini kami mengisi IP setiap node sesuai tabel pembagian IP. Gateway setiap node adalah IP router di subnet yang sama.

### Pengujian
#### Brief Overview

Menguji bahwa setiap node dapat melakukan ping ke node lain, sesuai ketentuan soal no. 5.

#### Step by Step

##### Simple PDU di CPT

Dalam step ini kami menyiapkan 27 scenario Simple PDU (ICMP) di panel simulasi CPT, dengan total 638 PDU.

| Scenario | Isi | Jumlah PDU |
| --- | --- | --- |
| 0 | Setiap node melakukan ping ke salah satu node terjauh darinya | 25 |
| 1 sampai 25 | Satu scenario per node, node tersebut melakukan ping ke 24 node lainnya | 24 per scenario |
| 26 | Lain melakukan ping ke satu interface di setiap router | 13 |

Scenario 1 sampai 25 mencakup semua pasangan node. Scenario dijalankan di mode Realtime, lalu kolom Last Status di daftar PDU menunjukkan hasilnya. Scenario 0 dipakai untuk menampilkan animasi paket di mode Simulation.

<!-- Screenshot: daftar PDU scenario 1 sampai 25 di mode Realtime, simpan di Asset/Pengujian/ (belum diambil) -->

##### Node : ping manual

| Dari | Perintah | Jalur yang diuji |
| --- | --- | --- |
| Lain | `ping 192.216.64.2` | Node terdalam ke Cyberia, lewat 8 router |
| Karl | `ping 192.216.33.2` | Psyche ke Knights, lewat Switch2 dan Tachibana |
| Denpa | `ping 192.216.25.2` | Accela ke Navi |
| Juri | `ping 192.216.8.2` | Copland ke Schumann, lewat Switch2 |
| Mika | `tracert 192.216.0.2` | Jalur terpanjang, menampilkan urutan router |

Dalam step ini kami menjalankan ping dari Command Prompt node untuk jalur-jalur terjauh. Ping pertama bisa timeout karena ARP, jadi ping diulang sekali.

<!-- Screenshot: hasil ping dan tracert di atas, simpan di Asset/Pengujian/ (belum diambil) -->

### Catatan

- Cyberia memakai `ip address dhcp` ke Cloud0. Di CPT, Cloud tidak memberi DHCP, jadi koneksi ke luar topologi tidak diuji di CPT.
