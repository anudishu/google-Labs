Migrate a MySQL Database to Google Cloud SQL: Challenge Lab


export ZONE=us-central1-b


gcloud sql instances create wordpress --tier=db-n1-standard-1 --activation-policy=ALWAYS --gce-zone $ZONE


gcloud sql users set-password --host % root --instance wordpress --password Password1*


export ADDRESS=35.224.142.38/32


gcloud sql instances patch wordpress --authorized-networks $ADDRESS --quiet


gcloud compute ssh blog --zone=us-central1-b



mysql --host=35.232.25.42 \
    --user=root --password
  

CREATE DATABASE wordpress;
CREATE USER 'blogadmin'@'%' IDENTIFIED BY 'Password1*';
GRANT ALL PRIVILEGES ON wordpress.* TO 'blogadmin'@'%';
FLUSH PRIVILEGES;



sudo mysqldump -u root -p Password1* wordpress > wordpress_backup.sql



mysql --host=35.232.25.42 --user=root --password=Password1* --verbose wordpress < wordpress_backup.sql




sudo service apache2 restart


cd /var/www/html/wordpress



sudo nano wp-config.php 
