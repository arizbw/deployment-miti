E-Commerce
==========

.. table::
   :widths: auto
   :align: center

   +----------------------+---------------------------------------------------------------+
   |**Person in charge**  | `Al Ghifari Enerza Sentanu <https://github.com/penguinerza>`_ |
   +----------------------+---------------------------------------------------------------+
   |**Service**           | E-Commerce                                                    |
   +----------------------+---------------------------------------------------------------+
   |**Aplikasi**          | Joomla                                                        |
   +----------------------+---------------------------------------------------------------+


Joomla merupakan aplikasi e-commerce open source yang dipublish oleh Open Source Matters, inc. Menggunakan MySQL sebagai database penyimpanan data.

Proses Instalasi
----------------

Untuk melakukan deployment joomla pada Google Kubernetes Engine, anda perlu mempersiapkan project pada Google Cloud Platform. Untuk mempersiapkan project, anda dapat mengikuti tutorial pada `link berikut <https://cloud.google.com/kubernetes-engine/docs/tutorials/hello-app>`_.
Setelah project terbuat, anda perlu mempersiapkan container registry joomla.


Untuk membuat container registry joomla, anda dapat membuat Dockerfile dengan isi sebagai berikut:

.. code-block:: bash
   
   # Base image
   FROM php:7.4-apache

   # Set working directory
   WORKDIR /var/www/html

   # Install dependencies
   RUN apt-get update && apt-get install -y \
      wget \
      curl \
      unzip \
      && rm -rf /var/lib/apt/lists/*

   # Install Joomla
   ARG JOOMLA_VERSION=4-3-1
   RUN curl -L -o joomla.zip "https://downloads.joomla.org/cms/joomla4/${JOOMLA_VERSION}/Joomla_${JOOMLA_VERSION}-Stable-Full_Package.zip?format=zip" \
      && unzip -q joomla.zip \
      && rm joomla.zip

   # Set file permissions
   RUN chown -R www-data:www-data /var/www/html

   # Apache configuration
   COPY apache-config.conf /etc/apache2/sites-available/000-default.conf
   RUN a2enmod rewrite

   # Expose port
   EXPOSE 80

   # Start Apache
   CMD ["apache2-foreground"]
   
Untuk versi joomla dapat anda sesuaikan dengan versi yang anda inginkan. Untuk membuat container registry, anda dapat menggunakan perintah berikut:

.. code-block:: bash
   
   docker build -t gcr.io/[PROJECT-ID]/joomla:v1 .
   docker push gcr.io/[PROJECT-ID]/joomla:v1

Karena proyek ini bernama miti-ta2, maka container registry yang digunakan adalah gcr.io/miti-ta2/joomla:v1. Sehingga perintah yang digunakan adalah sebagai berikut:

.. code-block:: bash
   
   docker build -t gcr.io/miti-ta2/joomla:v1 .
   docker push gcr.io/miti-ta2/joomla:v1

Anda dapat menggantinya sesuai dengan nama proyek anda.

Setelah membuat container registry, berikutnya anda dapat membuat file joomla.yaml dengan isi sebagai berikut:

.. code-block:: yaml

   apiVersion: v1
   kind: Service
   metadata:
   name: joomla
   spec:
   ports:
   - protocol: TCP
      port: 80
      targetPort: 80
   selector:
      app: joomla
   type: LoadBalancer
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
   name: joomla-deploy
   spec:
   selector:
      matchLabels:
         app: joomla
   template:
      metadata:
         labels:
         app: joomla
      spec:
         containers:
         - name: joomla
         image: gcr.io/miti-ta2/joomla@sha256:f136a05418dde8176b72e079065d07d6505cdafb4e76e32f8f6dec3bfbe70b86
         ports:
         - containerPort: 80
         volumeMounts:
            - mountPath: /var/www/html/configuration.php
               name: joomla-storage
               subPath: configuration.php
         volumes:
         - name: joomla-storage
            configMap:
               name: joomla-config

File joomla.yaml adalah file yang digunakan untuk melakukan deployment joomla. Pada file ini sudah terdapat service dan deployment joomla. Selain itu, terdapat juga volume dan configMap yang digunakan untuk menyimpan konfigurasi joomla. Pada proyek ini, menggunakan configMap sebagai berikut:

.. code-block:: yaml
   
   apiVersion: v1
   kind: ConfigMap
   metadata:
   name: joomla-config
   data:
   configuration.php: |
      <?php
      class JConfig {
         // ...
         public $host = 'mariadb-1-mariadb';
         public $db = 'wordpress_ecommerce';
         public $user = 'root';
         public $password = 'sijs2pyBmoNHk38QiW4EdFQZTyXsnC5N';
         // ...
      }
      ?>

Anda dapat menyesuaikan konfigurasi joomla sesuai dengan kebutuhan anda. Setelah membuat file joomla.yaml dan joomla-config.yaml, anda dapat melakukan deployment joomla dengan menggunakan perintah berikut:

.. code-block:: bash
   
   kubectl apply -f joomla-config.yaml
   kubectl apply -f joomla.yaml

Setelah melakukan deployment, anda dapat melihat status deployment dengan menggunakan perintah berikut:

.. code-block:: bash
   
   kubectl get deployment joomla-deploy

Setelah deployment selesai, anda dapat melihat status pod dengan menggunakan perintah berikut:

.. code-block:: bash
   
   kubectl get pods

Setelah pod berstatus running, anda dapat melihat status service dengan menggunakan perintah berikut:

.. code-block:: bash
   
   kubectl get service joomla

Setelah service berstatus running, anda dapat mengakses joomla dengan menggunakan external IP tersebut.

.. warning::
   Untuk melakukan instalasi joomla, anda membutuhkan database MySQL terlebih dahulu.


