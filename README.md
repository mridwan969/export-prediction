# export-prediction

**How to deploy the Flask API and the ML model to Google Cloud Platform**

Login to Google Cloud Platform

1. Go to Navigation Menu > Compute Engine > VM Instances

    Create a VM instance to build a docker image:
    - choose e-2 standard-2 for the machine configuration
    - choose Debian GNU/Linux(Buster) for the operating system
    - set disk to Standard Persistent Disk and set the size to be 30GB

2. Activate Cloud Shell

    SSH to the VM instance that just created with the command:
    $ gcloud compute ssh _INSTANCE_NAME_ --zone _INSTANCE_ZONE_

3. Install Git and Git-LFS in order to be able to clone the dataset in the git repo in the VM with the commands below:
    $ sudo apt-get update
    $ sudo apt-get install git
    $ sudo apt-get update
    $ sudo apt-get install git-lfs

    Install Docker in the VM, just follow the steps in this article https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-debian-10

    After the installation done, clone the repository for the tensorflow api with the command:
    $ sudo git clone https://github.com/rahilwisdom/Projects.git

4. After cloning, go to the repo directory:
    $ cd Projects

    Build the docker image inside the directory:
    $ sudo docker build -t ghcr.io/GITHUB_USERNAME/IMAGE_NAME:TAG .

5. Access this documentation https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry to learn how to work around github      PAT and how to use github container registry

6. After everything's done just push the image to github container registry by using the command:
   sudo docker push ghcr.io/GITHUB_USERNAME/IMAGE_NAME:TAG

    When the push done, we can just delete the VM if there is no more use to it or you can just keep it to build another container.

    Exit the SSH by tiping exit

7. Go back to Google Cloud shell and login with your github account the same way as the fifth step.
8. Pull the image from github container registry to the cloud shell with the command:
    $ docker pull ghcr.io/GITHUB_USERNAME/IMAGE_NAME:TAG
    
    then change the name of the image:
    $ docker tag ghcr.io/GITHUB_USERNAME/IMAGE_NAME:TAG asia.gcr.io/GCP_PROJECT_ID/IMAGE_NAME:TAG
    
    lastly, push the image to GCP container registry, in order to be safe we can just logout from github container registry by using the command:
    $ docker logout
    
    then push immediately
    $ docker push asia.gcr.io/GCP_PROJECT_ID/IMAGE_NAME:TAG
    
9. When the push is done, go to Navigation Menu > Container Registry > Images then select the newly pushed image.

   Click the three dots at the end of the image then choose which services do you want to deploy your image, in our case we decided to use Cloud Run since we can use continous      deployment for our API.
   
10. Deploy to Cloud Run, then select your desired region, leave the rest to default and then deploy.

11. In addition you can add your image to your github repositories by navigating to github in the top left, go to Your Profile > Packages
    Click the newly pushed image then connect it to your repository.
    Go to repository and confirm that the image has been connected to the repository.
    

**How to deploy the Laravel API to Google Cloud Platform**

1. Login to Google Cloud Platform and Activate Cloud Shell

2. Clone the laravel api repository with the command:
    $ git clone https://github.com/gungdekrisna/Expert-API.git
    
    then go to the directory with:
    $ cd Expert-API

3. In the directory create a Dockerfile and a folder named docker:
    $ nano Dockerfile
    and paste the following to the file:
    
FROM composer:1.9.0 as build
WORKDIR /app
COPY . /app
RUN composer global require hirak/prestissimo && composer install

FROM php:7.3-apache-stretch
RUN docker-php-ext-install pdo pdo_mysql

EXPOSE 8080
COPY --from=build /app /var/www/
COPY docker/000-default.conf /etc/apache2/sites-available/000-default.conf
COPY .env.example /var/www/.env
RUN chmod 777 -R /var/www/storage/ && \
    echo "Listen 8080" >> /etc/apache2/ports.conf && \
    chown -R www-data:www-data /var/www/ && \
    a2enmod rewrite
    
    
    create docker folder then immediately go to the directory:
    $ mkdir docker
    then navigate to the folder to create two files:
    $ nano 000-default.conf
    then paste the following:
<VirtualHost *:8080>

  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/public/

  <Directory /var/www/>
    AllowOverride All
    Require all granted
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

    create another file:
    $ nano docker-compose.yml
    then paste:
version: '3'
services:
  app:
    build:
      context: ./
    volumes:
      - .:/var/www
    ports:
      - "8080:8080"

4. For the next part, go to Navigation Menu > Cloud Storage and create a bucket with all settings as default then upload the db_expert.sql file from this repositories to the bucket
5. Navigate to Navigation Menu > SQL > Create Instance > MySQL
    Customize the instance with :
    - Standard Machine : 1 vCPU, 3.75 GB
    - Storage : SSD
    - Storage Capacity : 10GB
    - Connection : Enable both Private and Public IP
    Then create the database
   
    After Creating the database, go to the newly created instance and choose Import, then choose the database bucket then leave everything to default
    Click Import
    
6. Go back to Cloud shell to and go to the root directory of the cloned repository then paste this command:
    $ nano .env.example
    and change the file to resemble this:
    APP_NAME=Laravel
APP_ENV=local
APP_KEY=base64:DJkdj8L5Di3rUkUOwmBFCrr5dsIYU/s7s+W52ClI4AA=
APP_DEBUG=true
APP_URL=http://localhost

LOG_CHANNEL=stack
LOG_LEVEL=debug

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=db_expert
DB_USERNAME=root
DB_PASSWORD=[DB_PASSWORD]

     then save the file
     
7. Go to Navigation Menu > VPC Networks > Serverless VPC Access > Create Connector
    - Choose a name and the region 
    - Set network to default
    - Subnet, Custom IP range and add 10.8.0.0
    - instance-type : f1-micro
    - 
9. In the root directory paste this command to build the image:
    $ docker build -t asia.gcr.io/PROJECT_ID/IMAGE_NAME:IMAGE_TAG .
    
    after the process done, push the image with the command:
    $ docker push asia.gcr.io/PROJECT_ID/IMAGE_NAME:IMAGE_TAG
    
    When the push is done, go to Navigation Menu > Container Registry > Images then select the newly pushed image.

    Click the three dots at the end of the image then choose which services do you want to deploy your image, in our case we decided to use Cloud Run since we can use continous     deployment for our API and choose deploy to Cloud Run.
    
 10. In the Cloud Run:
    - Choose a name and the region, then next
    - Confirm the Image, then go to Advanced settings
    - In the Variables & Secrets, ADD VARIABLE:
        DB_CONNECTION=mysql
        DB_HOST=127.0.0.1
        DB_PORT=3306
        DB_DATABASE=db_expert
        DB_USERNAME=root
        DB_PASSWORD=[DB_INSTANCE]
     - In the Connectioon, ADD CONNECTION and choose the database, then select the VPC Connector and next
     - Check Allow Unaunthenticated invocations
     - CREATE

  That's all the steps to deploy the Laravel API to Google Cloud Platform
    
  

