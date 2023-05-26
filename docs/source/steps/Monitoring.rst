Monitoring
==========

.. table::
   :widths: auto
   :align: center

   +----------------------+------------------------------------------------------------------+
   |**Person in charge**  | Ahmad Mustofa Halim `@mustosoft <https://github.com/mustosoft>`_ |
   +----------------------+------------------------------------------------------------------+
   |**Service**           | Monitoring                                                       |
   +----------------------+------------------------------------------------------------------+
   |**Aplikasi**          | Prometheus                                                       |
   +----------------------+------------------------------------------------------------------+

Aplikasi monitoring yang digunakan dalam tugas ini adalah Prometheus. Kami memilih Prometheus karena merupakan solusi *open-source* yang tangguh dan memiliki fleksibilitas yang tinggi dalam memantau dan mengumpulkan data dari berbagai sumber yang berbeda. Dengan Prometheus, kami dapat melakukan pemantauan *real-time* terhadap berbagai metrik kinerja dari masing-masing nodes seperti CPU *usage*, *memory*, dan aktivitas I/O *disk*.

Pada proses instalasi Prometheus, kami menggunakan Helm sebagai *package manager* untuk menginstall dan mengelola aplikasi di dalam cluster Kubernetes kami. Helm memungkinkan kami untuk dengan mudah mendefinisikan dan menyebarkan konfigurasi Prometheus serta komponen-komponen terkaitnya dalam lingkungan Kubernetes. Selain itu, Helm juga memungkinkan kami untuk melakukan *upgrade* dan *rollback* secara mudah serta memastikan bahwa kami memiliki versi Prometheus yang konsisten.

Proses Instalasi
****************

Berikut adalah langkah-langkah instalasi Prometheus menggunakan Helm

Membuat Namespace Monitoring
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Untuk memulai, pastikan bahwa kita berada di cluster ``ta-final``

.. code-block:: bash

   gcloud container clusters get-credentials ta-final --zone asia-southeast2-a --project miti-ta2

.. figure:: ../assets/Screenshot\ from\ 2023-05-26\ 21-17-02.png
   :align: center
   
   Get credential

Kemudian, buat namespace ``monitoring``

.. code-block:: bash

   kubectl create namespace monitoring

.. figure:: ../assets/Screenshot\ from\ 2023-05-26\ 21-22-11.png
   :align: center

   Membuat namespace

Namespace ``monitoring`` sudah dibuat. Namespace inilah yang akan digunakan untuk menginstall Prometheus.

.. note::

   *Namespace* ``monitoring`` untuk Prometheus dibedakan dengan *service-service* lain yang ada di cluster. Hal ini dilakukan untuk memudahkan proses monitoring dan memastikan bahwa Prometheus tidak akan terpengaruh oleh *service-service* lain yang ada di cluster.

   Selain itu, dengan menggunakan *namespace* yang berbeda, menandakan bahwa Prometheus berbeda secara logis dengan *service-service* lain yang ada di cluster.

Menambahkan Repo ``prometheus-community``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Sebelum menginstall Prometheus, pastikan repo ``prometheus-community`` sudah ditambahkan. Jika belum, tambahkan dengan menjalankan perintah berikut. Lakukan update setelah menambahkan repo.

.. code-block:: bash

   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm repo update

.. figure:: ../assets/Screenshot\ from\ 2023-05-26\ 21-35-24.png
   :align: center

   Menambahkan repo dan update

Repo ``prometheus-community`` sudah ditambahkan dan Helm sudah diupdate.

Menginstall ``kube-prometheus-stack``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Setelah repo ``prometheus-community`` ditambahkan, kita dapat menginstall ``kube-prometheus-stack`` dengan menjalankan perintah berikut

.. code-block:: bash

   helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring

.. figure:: ../assets/Screenshot\ from\ 2023-05-26\ 21-41-12.png
   :align: center

   Install Prometheus

Proses instalasi selesai. Kita dapat mengecek status instalasi dengan menjalankan perintah berikut

.. code-block:: bash

   kubectl get pods -n monitoring -o wide

.. figure:: ../assets/Screenshot\ from\ 2023-05-26\ 21-40-59.png
   :align: center

   Cek status instalasi

Setelah beberapa saat, semua pods akan berjalan dengan baik.

.. figure:: ../assets/Screenshot\ from\ 2023-05-26\ 22-03-40.png
   :align: center

   Semua pods berjalan dengan baik

.. figure:: ../assets/Screenshot\ from\ 2023-05-26\ 22-29-27.png
   :align: center

   List workloads dari dashboard GKE

Beberapa service dari pod-pod tersebut yang terbentuk:

.. figure:: ../assets/Screenshot\ from\ 2023-05-26\ 22-33-07.png
   :align: center

   List service dari ``kubectl``

.. figure:: ../assets/Screenshot\ from\ 2023-05-26\ 22-35-26.png
   :align: center

   List service dari dashboard GKE

Expose Pod
~~~~~~~~~~

Setelah semua pods berjalan dengan baik, kita dapat melakukan expose pod ``prometheus-grafana``. ``prometheus-grafana`` merupakan pod yang berisi aplikasi Grafana yang digunakan untuk memvisualisasikan data yang dikumpulkan oleh Prometheus. Untuk melakukan expose pod grafana, jalankan perintah berikut:

.. code-block:: bash

   kubectl expose pod prometheus-grafana-5bcc8c67c-j9bsx --type=LoadBalancer --name=grafana-service-external --port=80 --target-port=3000 -n monitoring

.. note::

   ``prometheus-grafana-5bcc8c67c-j9bsx`` merupakan nama pod yang berisi aplikasi Grafana. Nama pod dapat berbeda pada setiap instalasi. Untuk mengeceknya dapat dilakukan dengan menjalankan perintah ``kubectl get pods -n monitoring``

.. note::

   Secara default, port yang digunakan oleh Grafana adalah port ``3000``. Port ``80`` digunakan untuk meng-*expose* service Grafana ke luar cluster. Dengan demikian, kita dapat mengakses Grafana melalui IP eksternal yang diberikan oleh GKE.

.. figure:: ../assets/Screenshot\ from\ 2023-05-26\ 23-27-27.png
   :align: center

   Expose service sedang berjalan

Setelah proses expose selesai, kita dapat mengecek IP eksternal yang diberikan oleh GKE dengan menjalankan perintah berikut:

.. code-block:: bash

   kubectl get service -n monitoring

.. figure:: ../assets/Screenshot\ from\ 2023-05-26\ 23-28-50.png
   :align: center

   IP eksternal yang diberikan oleh GKE

Kita dapat mengakses Grafana melalui IP eksternal yang diberikan oleh GKE. Untuk mengeceknya, buka browser dan akses IP eksternal yang diberikan oleh GKE. Setelah itu, kita akan diarahkan ke halaman login Grafana.

.. note::

   IP external yang diberikan oleh GKE adalah ``34.101.36.18``, sehingga untuk mengakses Grafana, kita dapat mengakses ``http://34.101.36.18/login``.

   Username default Grafana adalah ``admin`` dan passwordnya adalah ``prom-operator``.

.. figure:: ../assets/Screenshot\ from\ 2023-05-26\ 23-32-38.png
   :align: center

   Halaman login Grafana


Skenario Testing
****************

Untuk melakukan testing, kita akan masuk ke dashboard Grafana dan melihat apakah data yang dikumpulkan oleh Prometheus sudah muncul di Grafana.

Login Dashboard
~~~~~~~~~~~~~~~

Login ke dashboard Grafana dengan menggunakan username dan password default Grafana.

.. figure:: ../assets/Screenshot\ from\ 2023-05-27\ 00-40-24.png
   :align: center

   Halaman awal setelah login

Monitoring Nodes
~~~~~~~~~~~~~~~~

Setelah login, kita akan masuk ke halaman utama Grafana. Grafana sudah menyediakan beberapa dashboard yang dapat digunakan untuk memonitoring nodes.

1. Masuk ke Dashboard, lalu cari dengan kata kunci ``node``
2. Pilih dashboard **Node Exporter / Nodes**

.. figure:: ../assets/Screenshot\ from\ 2023-05-27\ 00-46-28.png
   :align: center

   Dashboard **Node Exporter / Nodes**

3. Pada dashboard ini, kita dapat melihat beberapa informasi mengenai nodes yang ada di cluster. Informasi yang ditampilkan antara lain:

   - CPU Usage
   - Load Average
   - Memory Usage
   - Disk Usage
   - Network Usage

.. figure:: ../assets/Screenshot\ from\ 2023-05-27\ 00-46-41.png
   :align: center

   Informasi mengenai nodes

.. note::

   Untuk melihat informasi mengenai node yang berbeda, kita dapat memilih node yang ingin dilihat informasinya pada bagian atas kiri dashboard.

   Di cluster kami terdapat 3 buah nodes, yaitu

   .. table::
      :widths: auto
      :align: center

      +-----------------------------------------+-------------------+
      | NAME                                    | INTERNAL-IP       |
      +=========================================+===================+
      | gke-ta-final-default-pool-87063a08-2z3v | ``10.184.0.4``    |
      +-----------------------------------------+-------------------+
      | gke-ta-final-default-pool-87063a08-d7dj | ``10.184.0.5``    |
      +-----------------------------------------+-------------------+
      | gke-ta-final-default-pool-87063a08-gkth | ``10.184.0.3``    |
      +-----------------------------------------+-------------------+

   Identifier untuk masing-masing node/instance adalah IP internal dari node tersebut atau exporter yang digunakan oleh node tersebut.

Monitoring Pods
~~~~~~~~~~~~~~~

Selain monitoring nodes, kita juga dapat melakukan monitoring terhadap pods yang berjalan di cluster. Untuk melakukannya, kita dapat menggunakan dashboard yang sudah disediakan oleh Grafana.

1. Masuk ke Dashboard, lalu cari dengan kata kunci ``pods``
2. Pilih dashboard **Kubernetes / Compute Resources / Namespace (Pods)**

.. figure:: ../assets/Screenshot\ from\ 2023-05-27\ 01-00-40.png
   :align: center

   Dashboard **Kubernetes / Compute Resources / Namespace (Pods)**

3. Pada dashboard ini, kita dapat melihat beberapa informasi mengenai pods yang ada di cluster yang dikelompokkan berdasarkan namespace. Informasi yang ditampilkan antara lain:

   - CPU Usage
   - CPU Quota
   - Memory Usage
   - Memory Quota
   - Network Usage
   - Network Bandwidth

   Di antara informasi tersebut, yang paling penting adalah CPU Usage dan Memory Usage. Informasi tersebut dapat digunakan untuk mengetahui apakah pods tersebut membutuhkan resource yang lebih besar atau tidak. Skenario ini akan dilakukan dengan cara melakukan load testing terhadap masing-masing service yang ada di cluster. Tools yang digunakan untuk melakukan load testing adalah Apache JMeter.

   .. tip::

      Selain kita dapat melihat statistik pods berdasarkan namespace, kita juga dapat melihat statistik pods berdasarkan node. Untuk melakukannya, kita dapat memilih dashboard **Kubernetes / Compute Resources / Node (Pods)**.

      .. figure:: ../assets/Screenshot\ from\ 2023-05-27\ 01-10-22.png
         :align: center
         :width: 80%

         Dashboard **Kubernetes / Compute Resources / Node (Pods)**



Referensi
*********

- `Grafana <https://grafana.com/>`_
- `prometheus-operator repository <https://github.com/prometheus-operator/prometheus-operator>`_
- `TechTarget - How to set up Prometheus for Kubernetes monitoring <https://www.techtarget.com/searchitoperations/tutorial/How-to-set-up-Prometheus-for-Kubernetes-monitoring>`_
- `Answer on why installing Prometheus in GKE Autopilot fails <https://github.com/prometheus-community/helm-charts/issues/713>`_
