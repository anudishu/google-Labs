



 ![Image Alt](https://github.com/anudishu/google-Labs/blob/9c295a40d87c277f19b72affdb7de59017e25454/Screenshot%202025-01-27%20162237.jpg)

Migrate a MySQL Database to Google Cloud SQL: Challenge Lab


1.export ZONE=us-central1-b


2.gcloud sql instances create wordpress --tier=db-n1-standard-1 --activation-policy=ALWAYS --gce-zone $ZONE


3.gcloud sql users set-password --host % root --instance wordpress --password Password1*


4.export ADDRESS=35.224.142.38/32


5.gcloud sql instances patch wordpress --authorized-networks $ADDRESS --quiet


6.gcloud compute ssh blog --zone=us-central1-b



7.mysql --host=35.232.25.42 \
    --user=root --password
  

8.CREATE DATABASE wordpress;
CREATE USER 'blogadmin'@'%' IDENTIFIED BY 'Password1*';
GRANT ALL PRIVILEGES ON wordpress.* TO 'blogadmin'@'%';
FLUSH PRIVILEGES;



9.sudo mysqldump -u root -p Password1* wordpress > wordpress_backup.sql



10.mysql --host=35.232.25.42 --user=root --password=Password1* --verbose wordpress < wordpress_backup.sql




11.sudo service apache2 restart


12.cd /var/www/html/wordpress



13.sudo nano wp-config.php  -- > replace the local host with cloud sql public IP

/** MySQL hostname */
define('DB_HOST', '35.232.25.42');
