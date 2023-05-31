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

Prequisite
----------------
Instalasi Keycloak dapat dilakukan jika kluster sudah memenuhi beberapa hal berikut:

- Sudah memiliki host database
- Cluster Kubernetes aktif
- Wordpress beserta Persistent Volume Claim (PVC) yang sesuai




Proses Instalasi
----------------
Pemasangan fitur SSO pada kluster ini setidaknya memiliki 5 langkah utama, yaitu mulai dari mempersiapkan *database*, instalasi Helm, instalasi Keycloak, konfigurasi Keycloak, dan terakhir konfigurasi WordPress.

Deployment Keycloak kami luncurkan sebagai sebuah service tersendiri dan memiliki external IP yang dapat diakses melalui http://34.101.223.88 . Semua *resource* yang dipasang untuk deployment Keycloak berada pada *namespace* "default".

Berikut ini adalah detail dari langkah-langkah yang dilakukan untuk instalasi SSO menggunakan Keycloak :

Mempersiapkan *Database*
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Pada kluster ini sudah terinstall host database dari MariaDB. Saya hanya perlu membuat database dan user keycloak di dalam host tersebut. Masuk ke dalam *console* MariaDB melalui terminal dan sambungkan MariaDB dengan menjalankan perintah :

.. code-block:: sql

   CREATE DATABASE keycloak;
   GRANT ALL PRIVILEGES ON keycloak.* TO 'keycloak'@'%' IDENTIFIED BY 'password';
   FLUSH PRIVILEGES;
   EXIT;

.. figure:: ../assets/keycloak-images/keycloak-image20.png
   :align: center

Instalasi *Package Manager* Helm
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

*Deployment* Keycloak via Helm
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Konfigurasi admin Keycloak
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Instalasi *plugin* OpenID Connector pada WordPress
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Konfigurasi *plugin* OpenID Connector pada Wordpress
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Skenario Testing
----------------

TODO