# Dedsecc

Project ini terdiri, atas :
Ketua : Muhammad Ariq Darmawan 22.83.0856 (Firewall, VPN Server)
Anggota :
Mukhlis Nur Arif 22.83.0837 (Hot/Cold Backup, Monitoring Grafana)
Zakiya Fazila Salsabila 22.83.0850 (SSH, Load Balancing)
Tiara Citra Mustika 22.83.0864 (DNS, Load Balancing)
Intan Utami 22.83.0874 (SSL, Monitoring Prometheus)
Alfira Yuniar 22.83.0880 (Web Server, Database)

**DESKRIPSI KONSEP :**
(topologi)

Proyek ini bertujuan untuk merancang, membangun, dan mengelola infrastruktur web terdistribusi yang menggabungkan beberapa layanan kunci pada virtual machine (VM). Setiap VM memiliki peran khusus dalam menjaga ketersediaan tinggi, meningkatkan performa, dan memastikan keamanan sistem secara menyeluruh.

**LAYANAN YANG DISEDIAKAN :**

**1. VM 1: Server Aplikasi:**
- WordPress sebagai aplikasi utama.
- Database (seperti MySQL) untuk manajemen data aplikasi.
- Fokus pada performa aplikasi web.

**2. VM 2: Load Balancer (HAProxy):**
- Mengatur distribusi lalu lintas ke server aplikasi (VM 1).
- Potensi untuk menambah instance WordPress pada VM lain.

**3. VM 3: Keamanan dan Ketersediaan Tinggi:**
- OpenVPN untuk akses jaringan lokal secara aman dari luar.
- Solusi backup dengan opsi "hot" (online) dan "cold" (offline).

**4. VM 4: Monitoring dan Visualisasi:**
- Prometheus untuk pengumpulan metrik dari semua VM.
- Grafana untuk visualisasi data metrik dari Prometheus.

**TUJUAN :**
- Ketersediaan Tinggi (HA): Memastikan aplikasi web selalu tersedia dengan mengoptimalkan infrastruktur terdistribusi.
- Keamanan Terintegrasi: Memanfaatkan OpenVPN untuk akses yang aman dan solusi backup untuk perlindungan data.
- Pengawasan Terpusat: Menggunakan Prometheus dan Grafana untuk memantau performa sistem secara menyeluruh.

**KONFIGURASI YANG DITERAPKAN :**

# VM 1

## 1. Instalasi dan Konfigurasi Web Server

**Langkah 1: Instalasi Paket Apache2**
```
apt update
apt-get install apache2
```
**Langkah 2: Konfigurasi Apache2**
```
nano /etc/apache2/sites-available/000-default.conf
```
**Langkah 3: Sesuaikan Konfigurasi ini dengan domain yang anda gunakan**
```
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin admin@example.com
        ServerName example.com
        DocumentRoot /var/www/new_html
```
**Lengkah 4: Restart Layanan Apache2**
```
systemctl restart apache2
```
**Langkah 5: Cek Apache2**

Pastikan layanan web default muncul dengan membuka domain Anda di browser.

**Langkah 6: Melakukan Instalasi PHP**
```
apt-get update
apt-get install php php-mysql
```
**Langkah 7: Melakukan Instalasi Database Server**

Silakan lihat instruksi pada bagian yang relevan untuk instalasi dan konfigurasi database server. Disini : 

**Langkah 8: Buat Database untuk WordPress**

login ke Database Terlebih dahulu:
```
mysql -u root -p
```
buat database untuk Wordpress dan berikan password sesuai keinginan anda
```
CREATE DATABASE wordpress;
CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
**Langkah 9: Download dan Extract Paket Wordpress**
```
curl -O https://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
```
**Langkah 10: Pindahkan isi direktori Wordpress**

saya menyimpan file unduhan di /tmp,jadi diseusikan lokasi nya
```
mv /tmp/wordpress/* /var/www/html/
```
**Langkah 11: Copy file Konfigurasi utama WordPress**
```
cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
```
Buka Kofigurasi file nya
```
nano /var/www/html/wp-config.php
```
**Langkah 12: Sesuaikan isi Database dalam wp-config.php**
```
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** Database username */
define( 'DB_USER', 'ariq' );

/** Database password */
define( 'DB_PASSWORD', 'root' );

/** Database hostname */
define( 'DB_HOST', 'localhost' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );
```
**Langkah 13: Set hak akses**
```
chown -R www-data:www-data /var/www/html
```
**Langkah 14: Instalasi Admin Wordpress**

Ikuti langkah-langkah dalam tampilan yang muncul saat mengakses domain Anda untuk menyelesaikan instalasi WordPress.

**Langkah 15: Uji Konfigurasi Apache2**

Pastikan untuk membuat postingan dan melihat tampilan pada browser untuk memverifikasi konfigurasi.


## 2. Instalasi dan Konfigurasi SSH Server

**Langkah 1: Lakukan Instalasi Paket SSH Server**
```
apt-get install openssh-server
```
**Langkah 2: Buka Direktori konfigurasi ssh dengan text editor(disini saya menggunakan nano)**
```
nano /etc/ssh/sshd_config
```
**Langkah 3: Edit Konfigurasi seperti dibawah ini**
```
Include /etc/ssh/sshd_config.d/*.conf

Port 2222
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
#LogLevel INFO

# Authentication:

#LoginGraceTime 2m
PermitRootLogin yes
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

#PubkeyAuthentication yes

# Expect .ssh/authorized_keys2 to be disregarded by default in future.
#AuthorizedKeysFile	.ssh/authorized_keys .ssh/authorized_keys2

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody
```
Port telah diubah dari 22 ke 2222 untuk meningkatkan keamanan.
Untuk "PermitRootLogin", jika tidak mengizinkan root untuk login dengan remote SSH, ubah menjadi "prohibit-password"

**Langkah 4: Restart layanan SSH Server**
```
systemctl restart sshd
```
**Langkah 5: Uji Konfigurasi**
```
ssh root@IPADDR -p 2222
```
Pastikan untuk mengganti IPADDR dengan alamat IP yang sesuai dan 2222 dengan port yang telah dikonfigurasi sebelumnya.

## 3. Instalasi dan Konfigurasi DNS Server

**Langkah 1: Instalasi Paket BIND9**
```
apt update
apt-get install bind9
```
**Langkah 2: Salin File Konfigurasi "Forward" dan "Reverse"**
```
cd /etc/bind
root@finalprojectspj:/etc/bind# root@finalprojectspj:/etc/bind# ls
bind.keys  db.127  db.empty  named.conf                named.conf.local    rndc.key
db.0       db.255  db.local  named.conf.default-zones  named.conf.options  zones.rfc1918
cp db.local db.forward
cp db.127 db.reverse
```
**Langkah 3: Mengatur File Konfigurasi db.forward**
```
nano db.forward
```
Sesuaikan konfigurasi agar cocok dengan domain Anda:
```
;
; BIND data file for local loopback interface
;
$TTL	604800
@	IN	SOA	finalprojectspj.com. root.finalprojectspj.com. (
			      2		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
@	IN	NS	finalprojectspj.com.
@	IN	A	10.10.10.2
ns      IN      A       10.10.10.2
www     IN      A       10.10.10.2
```
**Langkah 4: Konfigurasi file db.reverse**
```
nano db.reverse
```
Ubah konfigurasi untuk reverse lookup:
```
;
; BIND reverse data file for local loopback interface
;
$TTL	604800
@	IN	SOA	finalprojectspj.com. root.finalprojectspj.com. (
			      1		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
@	IN	NS	finalprojectspj.com.
2	IN	PTR	finalprojectspj.com.
```

**Langkah 5: Atur Zona DNS di named.conf.local**
```
nano named.conf.local
```
Konfigurasikan zona DNS:
```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "finalprojectspj.com" {
        type master;
        file "/etc/bind/db.forward";
};
zone "20.168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/db.reverse";
};
```
**Langkah 6: Konfigurasi DNS Forwarders di named.conf.options**
```
nano named.conf.options
```
tambahkan forwarders DNS:
```
options {
	directory "/var/cache/bind";

	// If there is a firewall between you and nameservers you want
	// to talk to, you may need to fix the firewall to allow multiple
	// ports to talk.  See http://www.kb.cert.org/vuls/id/800113

	// If your ISP provided one or more IP addresses for stable 
	// nameservers, you probably want to use them as forwarders.  
	// Uncomment the following block, and insert the addresses replacing 
	// the all-0's placeholder.

	// forwarders {
	//	10.10.10.2;
	// 	8.8.8.8;
	// };

	//========================================================================
	// If BIND logs error messages about the root key being expired,
	// you will need to update your keys.  See https://www.isc.org/bind-keys
	//========================================================================
	dnssec-validation auto;

	listen-on-v6 { any; };
};
```
**Langkah 7: Konfigurasi DNS pada Perangkat Server**
```
nano /etc/resolv.conf
```
Atur DNS pada server:
```
domain finalprojectspj.com
search finalprojectspj.com
nameserver 192.168.20.2
nameserver 10.10.10.2
nameserver 8.8.8.8
```
**Langkah 8: Restart Layanan Bind9**
```
systemctl restart bind9
```
**Langkah 9: Instalasi Paket DNS Resolver**
```
apt-get install dnsutils
```
1. Lakukan Nslookup untuk menguji Domain
2. Uji pada sisi ClientGunakan perintah seperti nslookup pada sistem klien untuk memastikan DNS berfungsi dengan baik dan sistem klien dapat menemukan domain yang diperlukan dengan benar.

# VM 2

## 1. Instalasi dan Konfigurasi Haproxy Load Balancing 
 	
```
                         +-------------------+
                         |    10.10.10.1     |
                         |       [vm1]       |
                         +-------------------+
                                  |
                                  |
                                  v
+----------+          +----------------+          +-----------------+
|          |  :80     |                |   :80    |                 |
|  Client  +--------->+    HAProxy     +--------->    Backend      |
|          |          |                |          |    Servers      |
+----------+          +----------------+          +-----------------+
      ^                	      ^                          ^
      |                       |                          |
   +----+                   +----+                     +----+
      |                       |                          |
 +---------+             +---------+                 +---------+
|  10.10.10.3  | 	| 10.10.10.1  |   	    | 10.10.10.3 |
|   [vm3]     |  	 |   [vm1]    |  	    |   [vm3]   |
 +---------+               +---------+   	     +---------+
```
**Langkah 1: Instalasi Paket HAProxy**
```
root@dlp:~# apt -y install haproxy
```

**Langkah 2: Mengonfigurasi Paket HAProxy**
```
root@dlp:~# vi /etc/haproxy/haproxy.cfg
```
**Langkah 3: Tambah Konfigurasi**
```
global
    log /dev/log local0
    log /dev/log local1 notice
    log-send-hostname
    daemon

defaults
    log global
    mode http
    option httplog
    option dontlognull
    retries 3
    timeout connect 5000
    timeout client 50000
    timeout server 50000

frontend main
    bind *:80
    default_backend servers

backend servers
    balance roundrobin
    server vm1 10.10.10.1:80 check
    server vm3 10.10.10.3:80 check
# Log settings
log 127.0.0.1 local0 debug
log 127.0.0.1 local1 notice
```
**Langkah 4: Restart layanan HAProxy**
```
root@dlp:~# systemctl restart haproxy
```

**Langkah 5: Mengubah pengaturan pada backend Web servers untuk mencatat header X-Forwarded-For.**

Untuk kasus pengaturan Apache2:
```
root@node01:~# a2enmod remoteip
root@node01:~# vi /etc/apache2/apache2.conf
```
Ganti seperti berikut:
```
RemoteIPHeader X-Forwarded-For
RemoteIPInternalProxy 10.0.0.30
LogFormat "%v:%p %a %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" vhost_combined
LogFormat "%a %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" combined
```
**Langkah 6: Restart layanan Apache2**
```
root@node01:~# systemctl restart apache2
```
**Langkah 7: Verifikasi bahwa semuanya berfungsi normal dengan mengakses frontend Server HAProxy.**

## 2. Instalasi dan Konfigurasi Prometheus

Untuk mengonfigurasi Pengekspor Blackbox, memungkinkan untuk melakukan pengecekan pada endpoint melalui HTTP, HTTPS, DNS, TCP, dan ICMP.

**Langkah 1: Pada Node yang ingin Anda monitor dengan Pengekspor Blackbox, lakukan instalasi.**

```
root@node02:~# apt -y install prometheus-blackbox-exporter
```

**Langkah 2: Ini adalah berkas pengaturan dari Pengekspor Blackbox. (Pada contoh ini, biarkan default)**

```
root@node02:~# vi /etc/prometheus/blackbox.yml
```

```
modules:
  http_2xx:
    prober: http
  http_post_2xx:
    prober: http
    http:
      method: POST
```

**Langkah 3: Restart layanan Blackbox Exporter**

```
root@node02:~# systemctl enable prometheus-blackbox-exporter

```

**Langkah 4: Tambahkan pengaturan pada Node Server Prometheus.**

```
root@dlp:~# vi /etc/prometheus/prometheus.yml
```
```
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s
    scrape_timeout: 5s

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ['localhost:9090']

  - job_name: node
    # If prometheus-node-exporter is installed, grab stats about the local
    # machine by default.
    static_configs:
      - targets: ['localhost:9100']

  - job_name:  'Blackbox_http'
    metrics_path:  /probe
    params:
      module:  [http_2xx]
    static_configs:
      -  targets:
         -  10.10.10.2
    relabel_configs:
      -  source_labels:  [__address__]
         target_label:  __param_target
      -  source_labels:  [__param_target]
         target_label:  instance
      -  target_label:  __address__

         replacement:  10.10.10.2:9115
```
**Langkah 5: Restart layanan Prometheus**
```
root@dlp:~# systemctl restart prometheus
```
