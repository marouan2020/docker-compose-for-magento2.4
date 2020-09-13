# docker-compose-for-magento2.4

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
