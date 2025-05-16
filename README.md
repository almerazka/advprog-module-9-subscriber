**Tutorial 9 Pemrograman Lanjut (Advanced Programming) 2024/2025 Genap**
* Nama    : Muhammad Almerazka Yocendra
* NPM     : 2306241745
* Kelas   : Pemrograman Lanjut - A

## Apa itu amqp?
**AMQP** (_Advanced Message Queuing Protocol_) adalah protokol komunikasi terbuka yang digunakan untuk mengirimkan pesan antara aplikasi, organisasi atau komponen yang berbeda dalam sebuah sistem terdistribusi. Protokol ini memungkinkan aplikasi untuk berkomunikasi secara asinkron dengan cara yang dapat diandalkan, aman, dan _interoperable_. **AMQP** mendefinisikan format pesan, aturan pertukaran pesan, dan operasi yang dapat dilakukan pada pesan tersebut, sehingga mendukung skenario seperti _message queuing_, _publish-subscribe messaging_, dan _request-response messaging_.

Dalam arsitektur berbasis _event_, **AMQP** berfungsi sebagai penghubung antara pengirim (_publisher_) dan penerima (_subscriber_) pesan. Pesan yang dikirimkan oleh pengirim tidak langsung diterima oleh penerima, melainkan dikirimkan terlebih dahulu ke message broker, seperti **RabbitMQ**. _Broker_ ini kemudian mendistribusikan pesan tersebut ke antrian (_queue_) yang sesuai untuk diproses oleh penerima. Salah satu contoh implementasinya adalah penggunaan AMQP pada file `main.rs` di mana `CrosstownBus` menghubungkan `UserCreatedHandler` dengan _queue_ yang memiliki topik `user_created`. Dengan demikian, `UserCreatedHandler` dapat menerima dan memproses pesan yang dikirimkan. ke antrian tersebut.

> Secara singkat, **AMQP** berfungsi sebagai "jembatan" yang memfasilitasi komunikasi antara berbagai aplikasi atau layanan dengan cara yang asinkron dan terpisah.

## Apa arti dari guest:guest@localhost:5672 ?
String `guest:guest@localhost:5672` adalah format URL koneksi yang digunakan dalam **AMQP** (_Advanced Message Queuing Protocol_) untuk menghubungkan ke _message broker_ seperti **RabbitMQ**.
- `guest` pertama : Merujuk pada nama pengguna (_username_) yang digunakan untuk autentikasi saat menghubungkan ke server **RabbitMQ**. Di sini, nama penggunanya adalah `guest`.
- `guest` kedua : Merujuk pada kata sandi (_password_) yang terkait dengan nama pengguna. Di sini, kata sandinya juga `guest`.
- `localhost` : Menunjukkan _hostname_ tempat **RabbitMQ** berjalan. Dalam hal ini, `localhost` berarti **RabbitMQ** berjalan di mesin yang sama dengan tempat klien mencoba untuk terhubung.
- `:5672` : 5672 adalah _default port server_ **AMQP** dan akan digunakan oleh _server_ **RabbitMQ** untuk menerima koneksi dari _client_.

## Message Broker dengan RabbitMQ
### Simulation slow subscriber
![image](https://github.com/user-attachments/assets/59c49bb8-949a-4503-9c66-10b68cff810c)

![image](https://github.com/user-attachments/assets/0d7995eb-1fbf-4217-a127-e950f29db81d)

Pada gambar di atas, terlihat bahwa ketika saya mencoba untuk memberikan _delay_ 2 detik untuk setiap proses pada _subscriber_, terjadi penumpukan pesan di message broker (**RabbitMQ**). Hal ini karena _publisher_ mengirimkan pesan lebih cepat daripada _subscriber_ yang memproses pesan. Setiap kali _publisher_ dijalankan, pesan-pesan yang dikirimkan akan menumpuk di antrian (_queued messages_) di **RabbitMQ**, sementara _subscriber_ hanya dapat memproses pesan secara bertahap dengan _delay_ yang telah ditentukan. Dalam kasus ini, saya menjalankan `cargo run` pada _publisher_ sebanyak 3 kali, yang menyebabkan **RabbitMQ** memiliki 13 pesan dalam antrian. Antrian pesan ini semakin banyak seiring dengan bertambahnya _delay_ pada _subscriber_, karena setiap proses yang diterima oleh _subscriber_ harus menunggu proses sebelumnya selesai terlebih dahulu. Oleh karena itu, meskipun _publisher_ terus mengirimkan pesan, _subscriber_ membutuhkan waktu lebih lama untuk memprosesnya, yang menyebabkan _queued messages_ terus bertambah.

### One publisher and three subscribers
![image](https://github.com/user-attachments/assets/cfeef612-83e7-4de1-9961-e72540918460)

![Dijalankan sekali](https://github.com/user-attachments/assets/fa0d4748-9184-4737-870c-4d1b438e802d)

![Dijalankan beberapa kali](https://github.com/user-attachments/assets/ff5721e9-9456-4fff-9b21-f1033f93680d)

Berdasarkan gambar diatas, terlihat bahwa ketika saya menjalankan tiga _subscriber_ secara bersamaan, proses pemrosesan pesan terbagi di antara ketiga _subscriber_ tersebut. Ini tercermin dalam grafik _message rates_ yang menunjukkan tiga grafik garis berbeda, satu untuk setiap _subscriber_. Meskipun _publisher_ dijalankan beberapa kali, antrian pesan (_queued messages_) pada **RabbitMQ** jauh lebih sedikit dibandingkan dengan saat hanya satu _subscriber_ yang dijalankan. Dengan lebih dari satu _subscriber_, _publisher_ dapat mengirimkan pesan dengan lebih cepat dan lebih efisien, karena pesan-pesan tersebut akan dibagi di antara _subscriber-subcriber_ yang terhubung. Pada kasus saya, tidak ada pesan yang masuk ke dalam _queued messages_ ketika dijalankan sekali karena pesan langsung diproses oleh _subscriber_ yang terhubung. Hal ini terjadi karena _message broker_ mendistribusikan data yang diterima dari _publisher_ ke banyak _subscriber_ tanpa harus menunggu satu _subscriber_ untuk memproses pesan sebelumnya, sehingga antrian pesan di **RabbitMQ** sangat sedikit.

> Penambahan jumlah _subscriber_ ini menunjukkan bagaimana dengan hanya mengubah jumlah _subscriber_ yang menerima pesan, kita dapat memperoleh hasil yang berbeda tanpa perlu mengubah kode program. Dengan kata lain, ini adalah contoh dari `event-driven architecture`, di mana perubahan dalam konfigurasi _message broker_ atau jumlah _subscriber_ yang aktif dapat mempengaruhi cara pesan diproses tanpa mengubah logika kode.