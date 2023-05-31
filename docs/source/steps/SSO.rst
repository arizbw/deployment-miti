SSO
===

.. table::
   :widths: auto
   :align: center

   +----------------------+------------------------------------------------------------------+
   |**Person in charge**  | M Sendi Siradj `@sendisiradj <https://github.com/SendiSiradj>`_  |
   +----------------------+------------------------------------------------------------------+
   |**Service**           | SSO Integration                                                  |
   +----------------------+------------------------------------------------------------------+
   |**Aplikasi**          | Keycloak                                                         |
   +----------------------+------------------------------------------------------------------+

Aplikasi integrasi SSO yang digunakan dalam tugas ini adalah Keycloak. Kami memilih Keycloak karena merupakan solusi *open-source* yang paling sering digunakan dan relatif mudah digunakan. Dengan Keycloak, kami dapat membuat 2 jenis protokol untuk authorization yaitu OpenID atau SAML 2.0. Namun untuk instalasi pada kali ini, kami memilih protokol OpenID yang akan diterapkan.

Pada proses instalasi Keycloak, kami menggunakan Helm sebagai *package manager* untuk menginstall dan mengelola aplikasi di dalam cluster Kubernetes kami. Helm memungkinkan kami untuk dengan mudah mendefinisikan dan menyebarkan konfigurasi Keycloak serta komponen-komponen terkaitnya dalam lingkungan Kubernetes. Selain itu, Helm juga memungkinkan kami untuk melakukan *upgrade* dan *rollback* secara mudah serta memastikan bahwa kami memiliki versi Keycloak yang konsisten.

Setelah itu kami akan mengintegrasikan aplikasi Wordpress yang sudah dipasang sebelumnya agar proses *authentication* atau *log in* dapat menggunakan akun Keycloak yang sudah di *define* di konfigurasi *console* Keycloak.

Prerequisites
-------------
Instalasi Keycloak dapat dilakukan jika kluster sudah memenuhi beberapa hal berikut:

- Sudah memiliki `host` database
- Kluster Kubernetes aktif
- Wordpress beserta Persistent Volume Claim (PVC) yang sesuai




Proses Instalasi
----------------
Pemasangan fitur SSO pada kluster ini setidaknya memiliki 5 langkah utama, yaitu mulai dari mempersiapkan *database*, instalasi Helm, instalasi Keycloak, konfigurasi Keycloak, dan terakhir konfigurasi WordPress.

Deployment Keycloak kami luncurkan sebagai sebuah service tersendiri dan memiliki external IP yang dapat diakses melalui http://34.101.223.88 . Semua *resource* yang dipasang untuk deployment Keycloak berada pada *namespace* "default".

Berikut ini adalah detail dari langkah-langkah yang dilakukan untuk instalasi SSO menggunakan Keycloak :

Mempersiapkan *Database* Keycloak
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Pada kluster ini sudah terinstall host database dari MariaDB. Saya hanya perlu membuat database dan user Keycloak di dalam host tersebut. Masuk ke dalam *console* MariaDB melalui terminal dan sambungkan MariaDB dengan menjalankan perintah :

.. code-block:: sql

   CREATE DATABASE keycloak;
   GRANT ALL PRIVILEGES ON keycloak.* TO 'keycloak'@'%' IDENTIFIED BY 'password';
   FLUSH PRIVILEGES;
   EXIT;

.. figure:: ../assets/keycloak-images/keycloak-image5.png
   :align: center

   membuat database dan user baru pada MariaDB

Setelah itu, Cek nama service MariaDB yang sudah diinstal.

.. code-block:: bash

   kubectl get svc

.. figure:: ../assets/keycloak-images/keycloak-image10.png
   :align: center

   mendapatkan nama service host database

Instalasi *Package Manager* Helm
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Helm memudahkan kita untuk melakukan instalasi Keycloak karena Helm sudah melakukan pre-konfigurasi dan Helm juga menyediakan segala dependency yang dibutuhkan untuk menjalankan Keycloak.
Saya menggunakan Helm chart yang di `mantain` oleh Codecentric. Sebelum kita menginstal Helm kita perlu mempersiapkan `repository` dan file konfigurasi terlebih dahulu. Pertama, kita menambahkan repository ``codecentric`` yang isinya adalah `collection` chart Helm.

.. code-block:: bash

   helm repo add codecentric https://codecentric.github.io/helm-charts

.. figure:: ../assets/keycloak-images/keycloak-image19.png
   :align: center

   Helm chart dari codecentric

Kemudian, buat sebuah file yaml bernama ``values.yaml`` untuk chart Keycloak Helm. File ini menyimpan konfigurasi Keycloak Helm yang akan digunakan pada projek ini.

.. code-block:: yaml

 keycloak:
  persistence:
    dbVendor: mariadb
    dbName: keycloak
    dbHost: mariadb-1-mariadb
    dbUser: keycloak
    dbPassword: password
  username: admin
  password: admin
  ingress:
    enabled: false
  extraEnv: |
    - name: KEYCLOAK_FRONTEND_URL
      value: "http://<External-IP>/auth"

.. figure:: ../assets/keycloak-images/keycloak-image21.png
   :align: center

   isi values.yaml

value <External-IP> akan diganti nanti. Untuk saat ini simpan ``values.yaml`` terlebih dahulu.

Sekarang kita dapat menginstal `chart` Keycloak dengan menggunakan konfigurasi yang sudah disimpan di ``values.yaml``.

.. code-block:: bash

   helm install keycloak codecentric/keycloak -f values.yaml --namespace default

.. figure:: ../assets/keycloak-images/keycloak-image13.png
   :align: center

   instal chart Helm

Setelah proses instal selesai, cek apakah `service` Keycloak sudah berjalan.

.. code-block:: bash

   kubectl get svc --namespace default

.. figure:: ../assets/keycloak-images/keycloak-image4.png
   :align: center

   daftar `service` yang ada pada `namespace` ``default``

Pada tahap ini, `service` Keycloak sudah dapat dijalankan namun hanya bisa dilakukan di dalam kluser. Artinya kita harus melakukan `port-forwarding` terlebih dahulu melalui terminal kluster. Setelah itu, dashboard `console` admin Keycloak bisa dibuka.
Tentu saja ini membuat pengalaman memakai `console` admin akan menjadi terbatas. Oleh karena itu, kita butuh melakukan `expose` agar `console` admin dapat diakses dari luar.

*Deployment* Keycloak via Helm
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`Service` keycloak sudah terdaftar dan berjalan. Selanjutnya kita perlu melakukan `expose` ke `pod` yang memiliki `service` Keycloak yang tadi agar bisa diakses diluar dari kluster. Nama `pod` yang dipakai adalah ``keycloak-0``. Saya menamakan `service` yang diekspos tersebut ``keycloak-external``.

.. figure:: ../assets/keycloak-images/keycloak-image9.png
   :align: center

   expose pod yang mengandung service Keycloak

Kemudian cek apakah `service` ``keycloak-external`` sudah berjalan.

.. figure:: ../assets/keycloak-images/keycloak-image20.png
   :align: center

   daftar service pada namespace default

Disini kita mendapatkan `external` IP `service` tersebut adalah 34.101.223.88:80.

Setelah itu buka file ``values.yaml`` dan ganti <External-IP> menjadi `external` IP `service` ``keycloak-external``.

.. figure:: ../assets/keycloak-images/keycloak-image18.png
   :align: center

   update external IP pada ``values.yaml``

Terakhir lakukan perintah Helm upgrade untuk menerapkan perubahan.

.. code-block:: bash

   helm upgrade keycloak codecentric/keycloak -f values.yaml --namespace default

Catatan tambahan, karena kita menggunakan Helm maka kita dapat dengan mudah menggunakan banyak `service` untuk penginstalan Keycloak. Awalnya kita menggunakan `service` ``keycloak-http`` untuk menjalankan Keycloak di dalam kluster. Namun setelah melakukan
upgrade kita dapat menggunakan `service` ``keycloak-external`` untuk mengakses Keycloak dari luar. Ini juga menjadi salah satu keuntungan dari Helm.



Konfigurasi admin Keycloak
~~~~~~~~~~~~~~~~~~~~~~~~~~

Instalasi *plugin* OpenID Connector pada WordPress
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Konfigurasi *plugin* OpenID Connector pada Wordpress
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Skenario Testing
----------------

TODO