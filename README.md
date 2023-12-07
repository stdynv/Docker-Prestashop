

# Deploy PrestaShop application on Docker

:star: Star us on GitHub — it motivates us a lot!

[Prestashop](https://prestashop.fr/) is an open source Web application for creating an online store for e-commerce purposes. The application is published under the terms of the Open Software 3.0 license. PrestaShop is also the name of the company that publishes this solution.

![aimeos-frontend](https://res.cloudinary.com/practicaldev/image/fetch/s--81UoW7-c--/c_imagga_scale,f_auto,fl_progressive,h_420,q_auto,w_1000/https://raw.githubusercontent.com/moghwan/dev.to/master/blog-posts/1-prestashop-docker-compose/assets/header.png)

## Table Of Content

- [Project Overview](#project-overview)
    - [Project Architecture](#project-architecture)
    - [Task 1](#task-1)
    - [Task 2](#task-2)
- [Project setup](#typo3-setup)
    - [Database setup](#database-setup)
    - [Security](#security)
- [Page setup](#page-setup)
    - [Download the Aimeos Page Tree t3d file](#download-the-aimeos-page-tree-t3d-file)
    - [Go to the Import View](#go-to-the-import-view)
    - [Upload the page tree file](#upload-the-page-tree-file)
    - [Go to the import view](#go-to-the-import-view)
    - [Import the page tree](#import-the-page-tree)
    - [SEO-friendly URLs](#seo-friendly-urls)
- [License](#license)
- [Links](#links)

## Project Overview
the project is divided into two tasks, the first consists of deploying the Prestashop application in the same network range, the second part becomes challenging separate the frontend from the server side by network range is to ensure communication between the two.

- Task 1 : Deploy application on the same network adress
- Task 2 : Communicate the front side and server side with different network adresses 

### Project Architecture

![architeture diagram](https://github.com/stdynv/Docker-Prestashop/assets/78117993/5d48aaed-5194-45c8-a33a-9a5e7f925d59)

### Task 1 
Deploy this application inside a network. Make sure the two containers can communicate with each other using their names.

### Task 2
create two networks with the cidr specified in the schemas. Please make use of `--subnet` for adding a subnet to the network when creating it. The name of the networks are `ynov-frontend-network` for the frontend website container and `ynov-backend-network` for the database container.
*in order for the two containers to talk to each other* :
- make use of a third container that can be called `gateway` or `router` and connect this container to the two above networks.
- Inside the router gateway, configure the route table by using ip below command

**Note**: The goal for the second task is not to deploy the application but to make sure the containers can talk to each other using their ips.

## Project Setup

### Introduction
The prestashop image available [Prestashop image avaible here](https://hub.docker.com/r/bitnami/prestashop) can be used to deploy an e-commerce application. This application make use of two components. a fronten website and a database for storing persistante data.


```docker
docker pull bitnami/prestashop:latest
```

### Task1 Setup

**Step 1: Create a docker network for the application**

```docker
docker network create ynov-network
```
**Step 2:  Create a volume for MariaDB and create a MariaDB container**
- Create MariaDB volume
```docker
docker volume create --name mariadb_data
```
- Create a MariaDB container
```docker
docker run -d --name mariadb --network ynov-network \
-e MYSQL_ROOT_PASSWORD=0000 -e MYSQL_DATABASE=prestashop_db \
-e MYSQL_USER=prestashop_user -e MYSQL_PASSWORD=0000 \
-v mariadb_data:/var/lib/mysql mariadb
```

### Security

Since **TYPO3 9.5.14+** implements **SameSite cookie handling** and restricts when browsers send cookies to your site. This is a problem when customers are redirected from external payment provider domain. Then, there's no session available on the confirmation page. To circumvent that problem, you need to set the configuration option `cookieSameSite` to `none` in your `./typo3conf/LocalConfiguration.php`:

```php
    'FE' => [
        'cookieSameSite' => 'none'
    ]
```

## Site Setup

TYPO3 10+ requires a site configuration which you have to add in "Site Management" > "Sites" available in the left navigation. When creating a root page (a page with a globe icon), a basic site configuration is automatically created (see below at [Go to the Import View](#go-to-the-import-view)).

## Page Setup

### Download the Aimeos Page Tree t3d file

The page setup for an Aimeos web shop is easy, if you import the example page tree for TYPO3 10/11. You can download the version you need from here:

* [23.4+ page tree](https://aimeos.org/fileadmin/download/Aimeos-pages_2023.04.t3d) and later
* [22.10 page tree](https://aimeos.org/fileadmin/download/Aimeos-pages_2022.10.t3d)
* [21.10 page tree](https://aimeos.org/fileadmin/download/Aimeos-pages_21.10.t3d)

**Note:** The Aimeos layout expects [Bootstrap](https://getbootstrap.com) providing the grid layout!

In order to upload and install the file, follow the following steps:

### Go to the Import View

**Note:** It is recommended to import the Aimeos page tree to a page that is defined as "root page". To create a root page, simply create a new page and, in the "Edit page properties", activate the "Use as Root Page" option under "Behaviour". The icon of the root page will change to a globe. This will also create a basic site configuration. Don't forget to also create a typoscript root template and include the bootstrap templates with it!

![Create a root page](https://user-images.githubusercontent.com/213803/211549273-1d3883dd-710c-4e27-8dbb-3de6e45680d7.jpg)

* In "Web::Page", right-click on the root page (the one with the globe)
* Click on "More options..."
* Click on "Import"

![Go to the import view](https://user-images.githubusercontent.com/213803/211550212-df6daa73-74cd-459e-8d25-a56c413c175d.jpg)

### Upload the page tree file

* In the page import dialog
* Select the "Upload" tab (2nd one)
* Click on the "Select" dialog
* Choose the T3D file you've downloaded
* Press the "Upload files" button

![Upload the page tree file](https://user-images.githubusercontent.com/8647429/212347778-17238e05-7494-4413-adb3-a54b2b524e05.png)

### Import the page tree

* In Import / Export view
* Select the uploaded file from the drop-down menu
* Click on the "Preview" button
* The pages that will be imported are shown below
* Click on the "Import" button that has appeared
* Confirm to import the pages

![Import the uploaded page tree file](https://user-images.githubusercontent.com/8647429/212348040-c3e10b60-5579-4d1b-becc-72548826c6db.png)

Now you have a new page "Shop" in your page tree including all required sub-pages.

### SEO-friendly URLs

TYPO3 9.5 and later can create SEO friendly URLs if you add the rules to the site config:
[https://aimeos.org/docs/latest/typo3/setup/#seo-urls](https://aimeos.org/docs/latest/typo3/setup/#seo-urls)

## License

The Aimeos TYPO3 extension is licensed under the terms of the GPL Open Source
license and is available for free.

## Links

* [Web site](https://aimeos.org/integrations/typo3-shop-extension/)
* [Documentation](https://aimeos.org/docs/TYPO3)
* [Forum](https://aimeos.org/help/typo3-extension-f16/)
* [Issue tracker](https://github.com/aimeos/aimeos-typo3/issues)
* [Source code](https://github.com/aimeos/aimeos-typo3)
