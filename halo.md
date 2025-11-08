[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/e_s827HM)
| Name           | NRP        | Kelas     |
| ---            | ---        | ----------|
| Maleka Ghaniya | 5025241189 | Jarkom A |



## Put your topology config image here!

![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/e030bc42a3f3b0d0ea5baf06e6691c165c5370a5/Screenshot%202025-10-31%20012811.png)

## Put your GNS3 Project file here!

[link](https://github.com/Maleka0809/praktikum_jarkom_3/blob/499d02c6deb9bfbe13a530eb2d30694ab55dd731/praktikum_3.gns3project)

<br>

## Soal 1

> Setup Topo

> _Document the results of the subnet grouping that has been created._

**Answer:**

- Screenshot

  ![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/e030bc42a3f3b0d0ea5baf06e6691c165c5370a5/Screenshot%202025-10-31%20012811.png)

- Explanation
Topologi ini dibuat dengan beberapa subnet didalamnya, ada 4 subnet. setelah itu, atur ip static dengan cara menulisnya di konfigurasi setiap node dari subnet-subnet yang ada.
  ```
  auto eth0
  iface eth0 inet static
	      address [prefix].[subnet yang sudah ditentukan soal].[boleh apa saja(lebih baik urut)]
	      netmask 255.255.0.0 //netmask yang sudah ditentukan soal
	      gateway 10.58.0.254 //gateway 254 merupakan default lalu 10.58 merupakan prefix saya 
  ```
Berikut ini merupakan pengelompokan dan alamat ip yang sudah diatur.

subnet 1
```
    Lune		  10.58.2.1
    sciel		  10.58.2.2
    Gustave		10.58.2.3
```
subnet 2
```
    Renoir		10.58.3.1
    verso 		10.58.3.2
```
subnet 3
```
    Alicia		10.58.4.1
```
subnet 4
```
    Maelle		10.58.5.1
    Monocco		10.58.5.2
    Equie		  10.58.5.3
```
      

<br>

## Soal 2

> Buatlah konfigurasi untuk domain 
> **lune33.com** → ke IP node Lune , 
> **sciel33.com** → ke IP node Sciel ,
> **gustave33.com** → ke IP node Gustave 
> pada DNS Master Renoir. Kemudian konfigurasikan node Verso sebagai DNS Slave yang bekerja untuk DNS Master Renoir.

> _Dns Configuration , on  the DNS Master (Renoir)_
> _lune33.com → IP of node Lune ,_
> _sciel33.com → IP of node Sciel ,_
> _gustave33.com → IP of node Gustave_
> _Configure Verso as the DNS Slave that works with DNS Master Renoir._

**Answer:**

- Screenshot

  ![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/6c9f90d6dc0d15d1bab9d449450232f3be37cb11/Screenshot%202025-10-31%20025518.png)
  <br>
  ![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/88f590034883b17e9e832811017f8305bfd1a2e8/Screenshot%202025-10-31%20084128.png)

- Explanation
  ### Langkah-langkah :
  1. Pada node renoir dibuatlah direktori dan file dengan perintah `nano /etc/bind/named.conf.local`, setelah itu diisi dengan kode sebagai berikut.
     ```
     zone "lune33.com" {
          type master;
          file "/etc/bind/db.lune33";
          allow-transfer { 10.58.3.2; };
          notify yes;
      };
     ```
    `zone "lune33.com"`digunakan untuk mendefinisikan zona baru yaitu lune33.com, zona ini berisi semua record DNS (seperti A, MX, NS, CNAME) untuk domain tersebut,lalu type ditulis `master` karena node renoir harus diatur menjadi master. `allow-transfer { 10.58.3.2; };` Menentukan alamat IP DNS Slave (10.58.3.2). Setelah itu buat file db per node dari subnet 1
  ```
  nano  /etc/bind/db.sciel33
    $TTL 604800
    @       IN      SOA     renoir.lune33.com. admin.lune33.com. (
                            2025103001  ; Serial
                            604800      ; Refresh
                            86400       ; Retry
                            2419200     ; Expire
                            604800 )    ; Negative Cache TTL
    ;
    @       IN      NS      renoir.sciel33.com.
    @       IN      NS      verso.sciel33.com.
    
    renoir  IN      A       10.58.3.1
    verso   IN      A       10.58.3.2
    @       IN      A       10.58.2.2   ; IP Node Sciel

  ```
  ``
	  File zona di atas mengatur domain lune33.com dengan TTL 604800 detik (7 hari). Server utama (SOA) ditetapkan ke renoir.lune33.com dengan email admin admin@lune33.com, serta pengaturan serial dan waktu sinkronisasi untuk DNS slave. Domain ini memiliki dua nameserver, yaitu renoir.sciel33.com dan verso.sciel33.com, masing-masing memiliki alamat IP 10.58.3.1 dan 10.58.3.2. Sementara itu, domain utama lune33.com diarahkan ke alamat IP 10.58.2.2, yaitu IP node Lune. setelah sudah kita run dengan `named -g`.
	  
  2. Pada node verso dikonfigurasi menjadi slave.
     pada konfigurasin verso itu hampir sama dengan isi `/etc/bind/named.conf.local` hanya saja type nya diganti `slave` untuk mendeklarasikan bahwa verso slave.
     ```
      zone "lune33.com" {
          type slave;
          file "/var/cache/bind/db.lune33.slave";
          masters { 10.58.3.1; };
      };   
     ```
     `file "/var/cache/bind/db.lune33.slave";` ini menunjukkan lokasi file zona yang berisi semua pemetaan hostname ke alamat IP.
  3. Konfigurasi client
     pada konfigurasi client ditambahkan name server, agar client menuju ke master dan slave.
     ```
        up echo "nameserver 10.58.3.1" > /etc/resolv.conf  //ini ip addr Renoir
        up echo "nameserver 10.58.3.2" >> /etc/resolv.conf  //ini ip addr verso
     ```
  4. ping dengan menggunakan lune33.com, sciel33.com, dan gustave33.com pada console client. Jika berhasil maka kita sudah berhasil mengerjakan no.2.
     
<br>

## Soal 3

> Tambahkan subdomain alias berupa exp.lune33.com yang mengarah ke alamat lune33.com dan exp.sciel33.com yang mengarah ke alamat sciel33.com (HINT: CNAME). Selain itu, tambahkan konfigurasi untuk melakukan reverse DNS lookup untuk domain gustave33.com

> _Subdomain Configuration,_ 
> _Add alias subdomains (HINT: CNAME)._
> _exp.lune33.com → alias to lune33.com_
> _exp.sciel33.com → alias to sciel33.com_
> _Also, configure reverse DNS lookup for the domain gustave33.com._

**Answer:**

- Screenshot

  ![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/07e2ee2a68a8626d5b71a56e5ff367ccc1bf42f9/Screenshot%202025-10-31%20103433.png)
  <br>
  ![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/0cc535b4da1541507791169f7ce2475b828d78f8/Screenshot%202025-10-31%20103734.png)
- Explanation
  jika sudah bisa ping dari client dengan menggunakan exp.lune33.com dan exp.sciel33.com berarti sudah berhasil, reverse sendiri cara test nya yaitu

  ```
  dig -x [ip dari node yang direverse]
  ```
  arti dari outpu yang diisi diatas yaitu :
	`status: NOERROR` : Ini menunjukkan bahwa DNS Server (Renoir) menerima query Anda dan dapat menjawabnya tanpa masalah.
	
	`QUESTION SECTION` : Menunjukkan query Anda: mencari PTR Record untuk 3.2.58.10.in-addr.arpa. (yang merupakan representasi terbalik dari IP 10.58.2.3).
	
	`ANSWER SECTION` : Ini adalah buktinya!
	
	`3.2.58.10.in-addr.arpa. 604800 IN PTR gustave33.com.` : Baris ini mengonfirmasi bahwa IP 10.58.2.3 berhasil diresolve kembali ke nama domain gustave33.com. menggunakan PTR Record yang Anda buat di Reverse Zone (db.10.58).
	
	`SERVER: 10.58.3.1`  : Mengonfirmasi bahwa query berhasil dijawab oleh Renoir (DNS Master Anda).

	cara :
	1. Mengatur Alias :
		1. Menambahkan `exp     IN      CNAME   lune33.com.` pada `/etc/bind/db.lune33`.
	 	2. Menambahkan `exp     IN      CNAME   sciel33.com.` pada `/etc/bind/db.sciel33`.
	  CNAME digunakan untuk menambahkan alias jadi bisa input exp.

  	2. Mengatur reverse :
  	   1. Menambahkan kose pada `/etc/bind/named.conf.local`
  	      ```
  	      zone "58.10.in-addr.arpa" {
			    type master;
			    file "/etc/bind/db.10.58"; // Nama file database Reverse Zone
			    allow-transfer { 10.58.3.2; };
			    notify yes;
			};
  	      ```
  	   2. Membuat file baru yang berisi
  	      ```
  	       nano /etc/bind/db.10.58 (Reverse Zone for 10.58.0.0/16)
			
			$TTL 604800
			@       IN      SOA     renoir.lune33.com. admin.lune33.com. (
			                        2025103103  ; Serial (Nomor unik, harus lebih tinggi dari sebelumnya)
			                        604800      ; Refresh
			                        86400       ; Retry
			                        2419200     ; Expire
			                        604800 )    ; Negative Cache TTL
			;
			@       IN      NS      renoir.lune33.com.
			@       IN      NS      verso.lune33.com.
			
			; PTR Record untuk Gustave (IP: 10.58.2.3)
			; Format: Host.Subnet_ID  IN  PTR  Nama_Domain_Lengap
			3.2     IN      PTR     gustave33.com.

  	      ``` 
	
  

## Soal 4

> Buatlah subdomain berupa expedition.gustave33.com dan delegasikan subdomain tersebut dari Renoir ke Verso dengan alamat IP tujuan adalah node Gustave. Kemudian, matikan Renoir dan coba lakukan ping ke semua domain dan subdomain yang telah dikonfigurasikan pada nomor 2, 3, dan 4.

> _Create a subdomain expedition.gustave33.com and delegate it from Renoir to Verso, with the target IP being node Gustave.Then, turn off Renoir and try pinging all domains and subdomains configured in tasks 2, 3, and 4 to verify delegation works correctly._

**Answer:**

- Screenshot

  ![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/2dfdf3f57f35b4cf6e8005cd18a84c84e7cc13f3/Screenshot%202025-10-31%20125340.png)
  <br>
  ![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/2dfdf3f57f35b4cf6e8005cd18a84c84e7cc13f3/Screenshot%202025-10-31%20125353.png)
  <br>
  ![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/2dfdf3f57f35b4cf6e8005cd18a84c84e7cc13f3/Screenshot%202025-10-31%20125412.png)
  <br>
  ![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/2dfdf3f57f35b4cf6e8005cd18a84c84e7cc13f3/Screenshot%202025-10-31%20130644.png)
  <br>
  ![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/2dfdf3f57f35b4cf6e8005cd18a84c84e7cc13f3/Screenshot%202025-10-31%20130403.png)
  <br>
  ![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/2dfdf3f57f35b4cf6e8005cd18a84c84e7cc13f3/Screenshot%202025-10-31%20125432.png)

- Explanation
  Disini saya mengubah serial dan mengubah sedikit dari renoir dan verso.
  pada renoir saya mengubah serial pada `db.gustave3`,`db.lune33`, dan `db.sciel33` menjadi `2025103003`(tidak terlalu penting tetapi jika berbeda dengan serial yang ada di verso akan gagal). saya juga menambahakan `expedition IN  NS  verso.gustave33.com.` pada `db.gustave33`, ini merupakan baris konfigurasi penting dalam DNS yang melakukan tindakan yang disebut Delegasi Subdomain. Fungsinya adalah untuk memberi tahu dunia bahwa nameserver yang bertanggung jawab (authoritative) untuk subdomain
  
  pada verso saya menambahkan sedikit argumen untuk mengidetifikasi db.expedition di `/etc/bind/named.conf.local`
  ```
  zone "expedition.gustave33.com" {
	    type master;
	    file "/etc/bind/db.expedition"; 
		};

  ```
  Konfigurasi ini membuat Verso memiliki dua pekerjaan penting:
		- Pekerjaan 1 (Slave): Menjadi fotokopi data domain utama dari Renoir (Master).
		- Pekerjaan 2 (Master): Menjadi sumber asli informasi untuk subdomain khusus.
  dan menambah 1 file yaitu `db.expedition` yang berisi
  ```
  		$TTL 604800
		@       IN      SOA     verso.gustave33.com. admin.gustave33.com. (
		                        2025103101  ; Serial (untuk zone baru)
		                        604800      ; Refresh
		                        86400       ; Retry
		                        2419200     ; Expire
		                        604800 )    ; Negative Cache TTL
		;
		@       IN      NS      verso.gustave33.com.
		
		; A Record untuk expedition.gustave33.com
		@       IN      A       10.58.2.3
  ```
  File ini juga memiliki dua tujuan utama: Mendeklarasikan Verso sebagai Master dan Menyediakan Alamat IP Akhir untuk subdomain.

<br>

## Soal 5

> Konfigurasi node Lune, Sciel, dan Gustave agar berfungsi sebagai web server Nginx yang akan menyajikan halaman profil, dimana halaman profil akan berbeda untuk setiap node. Dari folder berikut, gunakan profile_lune.html untuk menyajikan halaman profil di node Lune, profile_sciel.html untuk menyajikan halaman profil di node Sciel, dan profile_gustave.html untuk menyajikan halaman profil di node Gustave. Konfigurasikan Nginx di setiap node untuk menyimpan custom access log ke file /tmp/access.log dan error log ke file /tmp/error.log. 

> _Configure Lune, Sciel, and Gustave as Nginx web servers serving profile pages, where each node has a unique profile page:_
> _- Use profile_lune.html for Lune_
> _- Use profile_sciel.html for Sciel_
> _- Use profile_gustave.html for Gustave_
> _In each web server, Configure Nginx to store custom logs:_
> _- Access log: /tmp/access.log_
> _- Error log: /tmp/error.log_

**Answer:**

- Screenshot
#### Lune
  ![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/c5ae8c25156e0ee373c9a4d4b5ee942a5c59bb48/Screenshot%202025-10-31%20175644.png)
#### Sciel
  ![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/c5ae8c25156e0ee373c9a4d4b5ee942a5c59bb48/Screenshot%202025-10-31%20175716.png)
#### Gustave
  ![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/c5ae8c25156e0ee373c9a4d4b5ee942a5c59bb48/Screenshot%202025-10-31%20180010.png)

- Explanation
  	1. Membuat file html di `/myscripts/myweb`.
	2. Membuat konfigurasi pada setiap node `nano /myscripts/myconfig/nginx.conf` yang berisi.
	```
	user www-data;
	worker_processes auto;
	worker_cpu_affinity auto;
	pid /tmp/nginx.pid;
	
	events { worker_connections 768; }
	
	http {
	    # Menyimpan log di /myscripts/tmp/ sesuai permintaan
	    access_log /myscripts/tmp/access.log;
	    error_log /myscripts/tmp/error.log;
	
	    server {
	        listen 80;
	        server_name gustave33.com;
	
	        root /myscripts/myweb;
	        
	        # Menyajikan profile_gustave.html sebagai halaman utama
	        index profile_gustave.html;
	
	        location / {
	            try_files $uri $uri/ =404;
	        }
	    }
	}
	```

     ``
			konfigurasi Nginx ini telah dimodifikasi untuk menjalankan web server Gustave di lingkungan GNS3, menyajikan halaman profil khusus, dan mengatur custom log. Blok http mendefinisikan pengaturan global, seperti penggunaan `user www-data` dan lokasi Access dan Error Log di /myscripts/tmp/. Di dalamnya, blok server mengatur Virtual Host untuk gustave33.com di Port 80. Konfigurasi ini secara spesifik mengatasi Soal 5 dengan menggunakan direktif index `profile_gustave.html;`, yang memastikan file `profile_gustave.html` disajikan sebagai halaman utama dari direktori `/myscripts/myweb`  ketika diakses, sekaligus menyiapkan server ini untuk domain yang diminta oleh Soal 2/3.

<br>

## Soal 6

> Setelah website berhasil dideploy pada masing-masing node web server dan halaman dapat menampilkan profil yang sesuai,  buatlah custom access log ke file /tmp/access.log di masing-masing node web server menggunakan format log tertentu seperti di bawah:
> - Tanggal dan waktu akses dalam format standar log.
> - Nama node yang sedang diakses.
> - Alamat IP klien yang mengakses website.
> - Metode HTTP dan URI yang diakses oleh klien.
> - Status respons HTTP yang diberikan oleh server.
> - Jumlah byte yang dikirimkan dalam respons.
> - Waktu yang dihabiskan oleh server untuk menangani permintaan.
> - Contoh format log yang sesuai:
>   [01/Oct/2024:11:30:45 +0000] Jarkom Node Lune Access from 192.168.1.15 using method "GET /resep/bayam HTTP/1.1" returned status 200 with 2567 bytes sent in 0.038 seconds

> _After successfully deploying each website and verifying the correct profile page is displayed, create a custom access log in /tmp/access.log on each web server using the following format:_
> _- Date and time of access (standard log format)_
> _- Name of the node being accessed_
> _- IP address of the client accessing the website_
> _- HTTP method and URI accessed by the client_
> _- HTTP response status code_
> _- Number of bytes sent in the response_
> _- Time taken by the server to process the request_
> _- Example Log Format:_
> _[01/Oct/2024:11:30:45 +0000] Jarkom Node Lune Access from 192.168.1.15 using method "GET /resep/bayam HTTP/1.1" returned status 200 with 2567 bytes sent in 0.038 seconds_

**Answer:**

- Screenshot

#### Lune
  ![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/531065832054689a87c2784585cfe1b7fae4daee/Screenshot%202025-11-08%20191050.png)

#### Sciel
  ![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/2d3f1ac157d385228bec467a938676da57b1b25b/Screenshot%202025-11-08%20191115.png)

#### Gustave
  ![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/aad91a3750b42b393677d814131502719a29b0f0/Screenshot%202025-11-08%20213906.png)


- Explanation

  Soal kali ini merupakan perintah mengatur format yang ada pada `access.log` caranya adalah mengatur format. Disini saya menambahkan kode pada `/myscripts/myconfig/nginx.conf`
   kode : 
```
  'log_format custom_lune \"[\$time_local] Jarkom Node Lune Access from \$remote_addr using method \"\$request\" returned status \$status with \$body_bytes_sent bytes sent in \$request_time seconds\";'
```  
`log_format custom_lune` digunakan untuk membuat format log khusus di Nginx dengan nama `custom_lune`. Bagian di dalam kutip menentukan isi log, seperti waktu akses `($time_local)`, IP klien `($remote_addr)`, metode dan URI permintaan `($request)`, status HTTP `($status)`, ukuran respons `($body_bytes_sent)`, dan waktu proses `($request_time)`. Tanda backslash (\) dipakai agar karakter khusus seperti $ dan " tidak dibaca oleh shell, tapi diteruskan ke Nginx

<br>

## Soal 7

> Gustave merupakan web server yang tidak disarankan untuk dilihat oleh publik. Maka dari itu, ubahlah konfigurasi nginx sehingga halaman profil Gustave menjadi hanya bisa di akses melalui port 8080 dan 8888.

> _The Gustave web server should not be publicly accessible.
Modify the Nginx configuration so that Gustave’s profile page can only be accessed through ports 8080 and 8888._

**Answer:**

- Screenshot
#### tanpa port
![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/4efb78bb645065415195ea30ae0aee21e058d98d/Screenshot%202025-11-08%20191647.png)
#### port 8080
![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/4efb78bb645065415195ea30ae0aee21e058d98d/Screenshot%202025-11-08%20191654.png)
#### port 8888
![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/4efb78bb645065415195ea30ae0aee21e058d98d/Screenshot%202025-11-08%20193222.png)

- Explanation
  Server block yang menyajikan halaman profil (profile_gustave.html) dikonfigurasi agar tidak mendengarkan port 80, dan hanya mendengarkan port 8080 dan 8888.
  cara nya adalah dengan menambahkan kode pada ` /myscripts/myconfig/nginx.conf` yang ada pada node web server.
  kode :
```
server {
    listen 8080;
    listen 8888;
    ...
}
```
dan jangan lupa menghapus `listen 80;` dengan begitu gustave hanya bisa mendengar port 8080 atau 8888 dan tidak mendengar 80 lagi
<br>

## Soal 8

> Untuk mempermudah program ekspedisi, maka node Lune, Sciel, Gustave sepakat untuk membuat halaman informasi dengan konten yang sama. Maka dari itu, buatlah lagi 1 server block di dalam konfigurasi nginx yang akan menyajikan file HTML ini. Namun, mereka ingin menyajikan halaman informasi tersebut di port yang berbeda-beda, yaitu Lune menggunakan port 8000, Sciel menggunakan port 8100, dan Gustave menggunakan port 8200.

> _To simplify coordination for the expedition program, Lune, Sciel, and Gustave agree to create a shared information page with the same content. Add one more server block in each node’s Nginx configuration that serves this HTML file 
Each node should serve the information page on a different port:_
> _- Lune → port 8000_
> _- Sciel → port 8100_
> _- Gustave → port 8200_

**Answer:**

- Screenshot

![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/b9715b29f21e11a5306b7c5af2dad33699a847af/Screenshot%202025-11-08%20221421.png)
![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/b9715b29f21e11a5306b7c5af2dad33699a847af/Screenshot%202025-11-08%20221440.png)
![alt](https://github.com/Maleka0809/praktikum_jarkom_3/blob/b9715b29f21e11a5306b7c5af2dad33699a847af/Screenshot%202025-11-08%20221536.png)

- Explanation

  Tugas ini diimplementasikan dengan membuat file info.html di setiap node dan menambahkan server block kedua di /etc/nginx/sites-available/default untuk menyajikan file tersebut di port yang berbeda.
![info.html](https://drive.google.com/file/d/1BLg3S22ldhL-wRYN-ivowEqh8cYwwfVS/view)

Konfigurasi listen (Server Block Kedua):

Lune: `listen 8000;`

```
server {
    listen 8000;

...
}
```

Sciel: `listen 8100;`
```
server {
    listen 8100;

...
}
```

Gustave: `listen 8200;`

```
server {
    listen 8200;
    
    root /var/www/gustave;
    index info.html;
}
```
`server { ... }` Mendefinisikan server block baru (Virtual Host) yang dapat diakses melalui jaringan. `listen 8200;` Ini adalah fungsi kunci untuk Soal 8. Nginx diperintahkan untuk mendengarkan koneksi masuk hanya pada TCP port 8200. Node Lune menggunakan 8000 dan Sciel menggunakan 8100, sehingga Gustave mendapat 8200. `root /var/www/gustave;` Menetapkan direktori dasar (root directory) di mana Nginx akan mencari file yang diminta. `index info.html;` Menentukan bahwa ketika klien mengakses server hanya dengan port (misalnya, http://gustave33.com:8200), file info.html yang akan disajikan secara default.

<br>

## Soal 9

> Untuk mempermudah akses ke profil tiap anggota ekspedisi, buatlah 1 domain lagi yaitu "expeditioners.com" yang akan mengarah ke Alicia. Lalu, untuk mencegah overload dari salah satu web server, konfigurasikan reverse proxy Alicia agar bisa forward request ke server yang sesuai berdasarkan URL profile yang diminta oleh klien dengan ketentuan sebagai berikut:
> -  Request untuk “expeditioners.com/profil_lune” harus dialihkan ke halaman profil web server Lune.
> -  Request untuk “expeditioners.com/profil_sciel” harus dialihkan ke halaman profil web server Sciel.
> -  Request untuk “expeditioners.com/profil_gustave” harus dialihkan ke halaman profil web server Gustave.
> Jika terdapat request ke URL selain profil yang ditentukan, reverse proxy akan mengalihkan ke halaman informasi pada web server Lune.

> _To make it easier to access each member’s profile, create a new domain “expeditioners.com” that points to Alicia. "
Configure Alicia’s reverse proxy (Nginx) to forward requests to the correct web server based on the requested URL, with the following rules:_
> _- Request URL expeditioners.com/profil_lune, Forward To Lune’s profile page_
> _- Request URL expeditioners.com/profil_sciel, Forward To Sciel’s profile page_
> _- Request URL expeditioners.com/profil_gustave, Forward To Gustave’s profile page_
> _- Any other URL, Forward To Lune’s information page_

**Answer:**

- Screenshot

  `Put your screenshot in here`

- Explanation

  `Put your explanation in here`

<br>

## Soal 10

> Untuk mendistribusikan traffic halaman informasi, atur Reverse Proxy Alicia agar dapat membagi pekerjaan kepada web server Lune, Sciel, dan Gustave secara optimal menggunakan algoritma Round-robin. Pastikan target pembagian load merupakan halaman informasi, bukan halaman profil masing-masing web server.

> _To distribute traffic for the information page, configure the reverse proxy (Alicia) to use Round-robin load balancing between the three web servers: Lune, Sciel, and Gustave.
Ensure that only the information page is included in the load-balancing configuration - not the profile pages._

**Answer:**

- Screenshot

  `Put your screenshot in here`

- Explanation

  `Put your explanation in here`

<br>
  
## Problems

## Revisions (if any)

