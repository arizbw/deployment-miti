Database
========

.. table::
   :widths: auto
   :align: center

   +----------------------+------------------------------------------------------------------+
   |**Person in charge**  | Hanif Omar Kertapati https://github.com/hanif-omar-k             |
   +----------------------+------------------------------------------------------------------+
   |**Service**           |Database                                                          |
   +----------------------+------------------------------------------------------------------+
   |**Aplikasi**          | MariaDB                                                          |
   +----------------------+------------------------------------------------------------------+

Aplikasi database yang kami gunakan untuk project ini adalah MariaDB, Sebuah fork open source dari MySql. MariaDB digunakan karena gabungan dari kompabilitasnya dengan aplikasi menggunakan mysql sebagai drop in replacement dan simplisitasnya dalam install

Lebih tepatnya, versi MariaDB yang digunakan menggunakan install Google Cloud Engine marketplace click to deploy.

Proses Instalasi
----------------
Untuk memulai, pastikan bahwa kita berada di cluster ``ta-final``

.. code-block:: bash

   gcloud container clusters get-credentials ta-final --zone asia-southeast2-a --project miti-ta2


Navigasi ke halaman marketplace mariadb untuk kubernetes

.. figure:: ../assets/database2023-06-09\ 093502.png
   :align: center


Dan configure secara default dalam cluster utama

.. figure:: ../assets/database22023-06-09094128.png
   :align: center

Setelah menunggu beberapa saat, MariaDB akan terinstall dalam cluster. ini dapat dicek dalam tab kubernetes--applications--mariadb untuk mengecek user root, password, dan service

.. figure:: ../assets/database32023-06-09\ 094809.png
   :align: center

Skenario Testing
----------------

Seksi Testing ini akan digunakan untuk cara mengakses database

Pertama, connect kedalam kluster utama

.. code-block:: bash

   gcloud container clusters get-credentials ta-final --zone asia-southeast2-a --project miti-ta2



Akses lokal
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Untuk mengakses server mariadb secara lokal

dengan  $MARIADB_ROOT_PASSWORD sebagai root password dalam page application mariadb

.. code-block:: bash
   
   kubectl exec -it mariadb-1-mariadb-0 --namespace default -- mysql -h mariadb-1-mariadb -p$(echo $MARIADB_ROOT_PASSWORD)

.. figure:: ../assets/database4.png
   :align: center


Akses external
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Untuk testing akses external, akan digunakan DBvisualizer sebagai DBMS untuk testing


Untuk mengakses database secara external, service akan di expose melewati load balancer yang di-patch dalam service tersebut

$APP_INSTANCE_NAME sebagai mariadb-1
$NAMESPACE sebagai default

.. code-block:: bash

   kubectl patch svc "$APP_INSTANCE_NAME-mariadb" "$APP_INSTANCE_NAME-mariadb-secondary" \
   --namespace "$NAMESPACE" \
   --patch '{"spec": {"type": "LoadBalancer"}}'
Lalu, dapat ditunjukkan ip main untuk penggunaan external

.. code-block:: bash

   PRIMARY_IP=$(kubectl get svc $APP_INSTANCE_NAME-mariadb \
   --namespace $NAMESPACE \
   --output jsonpath='{.status.loadBalancer.ingress[0].ip}')
   echo "$PRIMARY_IP"

Data ini, kemudian dapat digunakan dalam DBMS bersama dengan data dari halaman application

.. figure:: ../assets/database5.png
   :align: center


Kemudian, database siap untuk dipakai.
