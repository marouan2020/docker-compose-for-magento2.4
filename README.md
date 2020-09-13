#Installer Magento 2.4 en utilisant Docker et Docker-Compose.
[![vagrant](https://img.shields.io/badge/vagrant-debian:stretch-blue.svg?longCache=true&style=flat&label=vagrant&logo=vagrant)](https://app.vagrantup.com/debian/boxes/stretch64)
[![dev-box](https://img.shields.io/badge/git/composer-blue.svg?longCache=true&style=flat&label=setup&logo=magento)](https://github.com/zepgram/magento2-fast-vm/blob/master/config.yaml.example)
[![mount](https://img.shields.io/badge/nfs/rsync-blue.svg?longCache=true&style=flat&label=mount)](https://github.com/zepgram/magento2-fast-vm/releases)
[![release](https://img.shields.io/badge/release-v1.3.6-blue.svg?longCache=true&style=flat&label=release)](https://github.com/zepgram/magento2-fast-vm/releases)
[![license](https://img.shields.io/badge/license-MIT-blue.svg?longCache=true&style=flat&label=license)](https://github.com/zepgram/magento2-fast-vm/blob/master/LICENSE)

![windows](https://img.shields.io/badge/windows-ok-green.svg?longCache=true&style=flat&label=windows&logo=windows)
![apple](https://img.shields.io/badge/mac-ok-green.svg?longCache=true&style=flat&label=mac&logo=apple)
![linux](https://img.shields.io/badge/linux-ok-green.svg?longCache=true&style=flat&label=linux&logo=linux)

![image](https://user-images.githubusercontent.com/16258478/68086496-0d43e100-fe4d-11e9-95ea-2bce3bee9884.png)&nbsp;&nbsp;&nbsp;&nbsp;![image](https://user-images.githubusercontent.com/16258478/68086436-70814380-fe4c-11e9-8ef4-6e39388cc679.png)&nbsp;&nbsp;&nbsp;&nbsp;![image](https://user-images.githubusercontent.com/16258478/68086442-7545f780-fe4c-11e9-8c5e-518ddba8735d.png)&nbsp;&nbsp;&nbsp;&nbsp;![image](https://user-images.githubusercontent.com/16258478/68086695-ba6b2900-fe4e-11e9-8f4f-68feb9bb0db2.png)&nbsp;&nbsp;&nbsp;&nbsp;![image](https://user-images.githubusercontent.com/16258478/68086427-62cbbe00-fe4c-11e9-83d5-24aec5b7c686.png)

[![associate-developer](https://images.youracclaim.com/size/340x340/images/48e73336-c91d-477f-a66f-3ad950acb597/Adobe_Certified_Professional_Experience_Cloud_products_Digital_Badge.png)](https://www.youracclaim.com/earner/earned/badge/406cc91a-0fda-4a6f-846b-19d7f8b59e0a)

#Sources
Avant de commencer, cela fait un moment que j’utilise Docker pour développer sous Magento 2. Après avoir testé plusieurs solutions, je me suis basé sur la solution proposée par Mark Shust.
J’ai également visionné un paquet de vidéos de la chaîne Xavki.

#Arborescence
Commencer par créer le répertoire de travail et au sein de ce dossier, créer les répertoires suivants :

mkdir -p src/app_data1 src/nginx_log1 src/phpfpm_log1 src/sock_data1 src/mariadb_data1 src/m

#Build et Installation

Lancer la commande ci-dessous pour construire et lancer les conteneurs :

docker-compose build && docker-compose up -d
Une fois les conteneurs lancés, télécharger et créer le projet Magento 2.4 :

docker-compose exec phpfpm composer create-project --repository=https://repo.magento.com/ magento/project-community-edition:2.4 .
Dès que les fichiers ont été téléchargés, vous pouvez lancer l’installation :

docker-compose exec -T phpfpm bin/magento setup:install \
  --db-host=mariadb1 \
  --db-name=magento \
  --db-user=magento \
  --db-password=magento \
  --base-url=http://localhost/ \
  --backend-frontname=system \
  --admin-firstname=Administrateur \
  --admin-lastname=Magento \
  --admin-email=adresse@email.com \
  --admin-user=administrateur \
  --admin-password=mettreIciVotreMotDePasse \
  --language=fr_FR \
  --currency=EUR \
  --timezone=Europe/Paris \ 
  --use-rewrites=1 \
  --search-engine=elasticsearch7 \
  --elasticsearch-host=elasticsearch1 \
  --elasticsearch-port=9200 \
  --elasticsearch-index-prefix=magento
A cet instant, Magento est installé, avant de vous lancer, je vous invite à lancer les commandes ci-dessous :

#Activation de RabbitMQ :

docker-compose exec -T phpfpm bin/magento setup:config:set --no-interaction --amqp-host=rabbitmq --amqp-port=5672 --amqp-user=magento --amqp-password=magento --amqp-virtualhost=/ --consumers-wait-for-messages=1

#Activation de Redis :

docker-compose exec -T phpfpm bin/magento setup:config:set --no-interaction --cache-backend=redis --cache-backend-redis-server=redis --cache-backend-redis-db=0

#Activation de Redis pour le Full Page Cache :

docker-compose exec -T phpfpm bin/magento setup:config:set --no-interaction  --page-cache=redis --page-cache-redis-server=redis --page-cache-redis-db=1

#Activation de Redis pour les sessions :

docker-compose exec -T phpfpm bin/magento setup:config:set --no-interaction --session-save=redis --session-save-redis-host=redis --session-save-redis-log-level=4 --session-save-redis-db=2

#Redémarrer les conteneurs :

docker-compose stop && docker-compose up -d

#Installation des modules dans la base de données :

docker-compose exec -T phpfpm bin/magento setup:upgrade && docker-compose exec -T phpfpm bin/magento setup:di:compile
