#Install Apache, MySQL and PHP
apt update -y
apt install apache2 -y
apt install mysql-server -y
apt install php libapache2-mod-php php-mysql -y

#Default bind address is to localhost. Comment out to allow remote connection to database
sed -e '/^bind/s/^/#/g' -i /etc/mysql/mysql.conf.d/mysqld.cnf
systemctl restart mysql.service

#Create virtualhost for website
mkdir /var/www/html/lamp
cat <<EOF > /etc/apache2/sites-available/lamp.conf
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/lamp
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
EOF

#Enable virtual host
a2ensite lamp.conf

#Disable default website
a2dissite 000-default
systemctl reload apache2

#Test page for Apache
cat <<EOF > /var/www/html/lamp/index.html
<html>
  <head>
    <title>your_domain website</title>
  </head>
  <body>
    <h1>Hello World!</h1>

    <p>This is the landing page of <strong>your_domain</strong>.</p>
  </body>
</html>
EOF

#PHP test script
cat <<EOF > /var/www/html/lamp/info.php
<?php
phpinfo();
EOF

#Configure the database
mysql -u root -e "ALTER USER 'root'@'localhost' IDENTIFIED BY 'Password123?';"
mysql -u root -e "CREATE DATABASE lamp_database;"
mysql -u root -e "CREATE USER 'lamp_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';"
mysql -u root -e "GRANT ALL ON lamp_database.* TO 'lamp_user'@'%';"
mysql -u lamp_user -ppassword -e "CREATE TABLE lamp_database.shopping_list (item_id INT AUTO_INCREMENT, content VARCHAR(255), PRIMARY KEY(item_id));"

#Add items to shopping list
mysql -u lamp_user -ppassword -e "INSERT INTO lamp_database.shopping_list (content) VALUES ('Bread'),('Milk'),('Cheese');"

#Create PHP script
cat <<EOF > /var/www/html/lamp/shopping_list.php
<?php
\$user = "lamp_user";
\$password = "password";
\$database = "lamp_database";
\$table = "shopping_list";

try {
  \$db = new PDO("mysql:host=localhost;dbname=\$database", \$user, \$password);
  echo "<h2>SHOPPING LIST</h2><ol>";
  foreach(\$db->query("SELECT content FROM \$table") as \$row) {
    echo "<li>" . \$row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException \$e) {
    print "Error!: " . \$e->getMessage() . "<br/>";
    die();
}
EOF
