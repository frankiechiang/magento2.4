### Requirement :
1. Install on new server
2. Install on Amazon Linux 2

### Configuration :
1. Install all require software
2. Configure Magento & fix

### 1. Install all require software
a. Install Mysql 8 server
```bash
yum install https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
yum install mysql-community-server
systemctl enable --now mysqld
mysql_secure_installation
```

b. Install PHP 7.4
```bash
amazon-linux-extras enable php7.4
yum clean metadata
yum install php-cli php-pdo php-fpm php-json php-mysqlnd
yum install php-opcache php-xml php-gd php-devel php-intl php-mbstring php-bcmath php-iconv php-soap
```
c. Change default php date time & increase the memory size
```bash
vi /etc/php.ini
date.timezone = Asia/Hong_Kong
memory_limit=2G
:wq
```
d. Disable opcache comment
```bash
vi /etc/php.d/10-opcache.ini
opcache.save_comments=1
:wq
```
e. Install elasticsearch
```bash
amazon-linux-extras enable java-openjdk11
yum clean metadata
yum install java-11-openjdk
cat <<EOF | sudo tee /etc/yum.repos.d/elasticsearch.repo
[elasticsearch-7.x]
baseurl=https://artifacts.elastic.co/packages/oss-7.x/yum
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
yum install elasticsearch-oss
systemctl enable elasticsearch.service
systemctl start elasticsearch.service
systemctl status elasticsearch
systemctl restart elasticsearch
systemctl status elasticsearch
```

f. Testing elasticsearch
```bash
curl -XGET 'http://localhost:9200/_cat/health?v&pretty'
```

g. Install magento
```bash
cd /var/www/html/<magento install directory>
composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition /var/www/html/<install directory>
bin/magento setup:install --base-url=<url> --db-host=localhost --db-name=<db name> --db-user=<db user> --db-password=<db pass> --admin-firstname=<admin> --admin-lastname=<admin> --admin-email=<email> --admin-user=<admin user> --admin-password=<admin pass> --language=en_US --currency=HKD --timezone=Asia/Hong_Kong --use-rewrites=1 --backend-frontname=<admin>
```

h. Applied directories security
```bash
cd /var/www/html/<magento install directory>
find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +
find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +
chown -R apache:apache .
chmod u+x bin/magento
```

### 2. Configure Magento & fix
“Your web server is set up incorrectly and allows unauthorized access to sensitive files. Please contact your hosting provider”

Add the path to your Magento pub/ directory to the DocumentRoot directive:

        DocumentRoot /var/www/html/magento2ce/pub

Update the env.php file
The following node needs to be appended to the env.php file.

```bash
vi env.php
'directories' => [
    'document_root_is_pub' => true
]
:wq
```


```bash
bin/magento cache:flush
```


setup Magento sample data
```bash
php bin/magento sampledata:deploy
php bin/magento setup:upgrade


php bin/magento setup:di:compile
php bin/magento setup:static-content:deploy -f
```


Backup functionality is disabled
```bash
bin/magento config:set system/backup/functionality_enabled 1

useradd magento -d /var/www/html -G wheel

htpasswd /usr/local/apache/password/passwords <username>
```

```bash
vi /usr/local/apache/password/group
MagentoCronGroup: <username1> ... <usernameN>
:wq
```

```bash
vi <magento_root>/pub/.htaccess
<Files cron.php>
   AuthType Basic
   AuthName "Cron Authentication"
   AuthUserFile /usr/local/apache/password/passwords
   AuthGroupFile <path to optional group file>
   Require group <name>
</Files>
:wq
```
Test cron job
[http://magento.example.com/cron.php?group=default]



