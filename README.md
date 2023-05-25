# MITI Final Project Documentation

## Introduction

This is the documentation for the final project of the MITI course. The project is about the deployment of services in kubernetes cluster. The services are:

- **Wordpress** - A content management system (CMS) based on PHP and MySQL.
- **MariaDB** - A community-developed, commercially supported fork of the MySQL relational database management system (RDBMS), intended to remain free and open-source software under the GNU General Public License. Development is led by some of the original developers of MySQL, who forked it due to concerns over its acquisition by Oracle Corporation in 2009.
- **Prometheus** - A free software application used for event monitoring and alerting. It records real-time metrics in a time series database (allowing for high dimensionality) built using a HTTP pull model, with flexible queries and real-time alerting.
- **Keycloak** - An open source software product to allow single sign-on with Identity Management and Access Management aimed at modern applications and services. As of March 2018 this JBoss community project is under the stewardship of Red Hat who use it as the upstream project for their RH-SSO product.
- **Joomla** - A free and open-source content management system (CMS) for publishing web content, developed by Open Source Matters, Inc. It is built on a model–view–controller web application framework that can be used independently of the CMS.

## Deployment of The Documentation

### Prerequisites

- Make

  _Find the installation step for your machine_

- Sphinx

  ```bash
  pip install sphinx
  ```

- Sphinx Autobuild

  ```bash
  pip install sphinx-autobuild
  ```

### Steps

1. Clone the repository.

   ```bash
   git clone
   ```

2. Go to the directory `docs`.

   ```bash
   cd docs
   ```

3. Build the documentation.

   ```bash
   make html
   ```

4. Clean built files.

   ```bash
   make clean
   ```

5. Auto build the documentation.

   This will build the documentation and serve it in `localhost:8000` and refresh the page when there is a change in the documentation.

   ```bash
   sphinx-autobuild ./source docs/_build/html
   ```
