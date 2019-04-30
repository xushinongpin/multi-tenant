

下载laravel  -- composer create-project --prefer-dist laravel/laravel blog



## **https 设置**

```
cd c:\wamp64\bin\apache\apache2.4.27\bin
openssl genrsa -aes256 -out private.key 2048
openssl rsa -in private.key -out private.key
openssl req -new -x509 -nodes -sha1 -key private.key -out certificate.crt -days 36500
c:\wamp64\bin\apache\apache2.4.27\conf\openssl.cnf
```

迁移位置

```
private.key 和certificate.crt 移动到 c:\wamp64\bin\apache\apache2.4.27\conf\key\目录下
```

修改 httpd.conf

```
c:\wamp64\bin\apache\apache2.4.27\conf\httpd.conf
    LoadModule ssl_module modules/mod_ssl.so
    Include conf/extra/httpd-ssl.conf
    LoadModule socache_shmcb_module modules/mod_socache_shmcb.so
```

修改 httpd-ssl.conf

```
c:\wamp64\bin\apache\apache2.4.27\conf\extra\httpd-ssl.conf
    DocumentRoot "${INSTALL_DIR}/www/multi-tenant-demo/public/"
    ServerName psc.app:443
    ServerAdmin admin@example.com
    SSLSessionCache "shmcb:c:/wamp64/bin/apache/apache2.4.27/logs/ssl_scache(512000)"
    ErrorLog "c:/wamp64/bin/apache/apache2.4.27/logs/error.log"
    TransferLog "c:/wamp64/bin/apache/apache2.4.27/logs/access.log"
    SSLCertificateFile "c:/wamp64/bin/apache/apache2.4.27/conf/key/certificate.crt"
    SSLCertificateKeyFile "c:/wamp64/bin/apache/apache2.4.27/conf/key/private.key"
```

检测

```
c:\wamp64\bin\apache\apache2.4.27\bin\httpd -t
```

重启wampserver

