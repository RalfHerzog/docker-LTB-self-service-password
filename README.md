docker-LTB-self-service-password
================================

A dockerfile for the LDAP ToolBox (LTB) Self Service Password utility, which is a PHP application that allows users to change their password in an LDAP directory. See http://ltb-project.org/wiki/documentation/self-service-password

## Quick Start
You can either run the image and link it to an external configuration file, or you can rebuild your own standalone image.

#### Running from the image, i.e. the `--with-volume` way
Pull the latest version of the image from the docker index. This is the recommended method of installation as it is easier to update image in the future. These builds are performed by the **Docker Trusted Build** service.

```bash
docker pull ralfherzog/ltb-self-service-password:1.3
```

Then, provide your own `config.inc.php` file, downloaded from [https://github.com/ltb-project/self-service-password/blob/v1.3/conf/config.inc.php](https://github.com/ltb-project/self-service-password/blob/v1.3/conf/config.inc.php) and modified according to your settings.

You can now run container:
* in foreground:
```bash
docker run -d \
-v $(pwd)/assets/config.inc.php:/usr/share/self-service-password/conf/config.inc.php:ro \
ralfherzog/ltb-self-service-password:1.3
```

The examples above expose service on port `80` on the containers ip address, so you can point your browser to http://container-ip/ in order to change LDAP passwords.

#### Building the image yourself

```bash
git clone https://github.com/ralfherzog/docker-LTB-self-service-password.git
cd docker-LTB-self-service-password
```
Edit `assets/config.inc.php` according to your local settings, then (re)build image with: 
```bash
docker build -t="$USER/ltb-self-service-password" .
```
You can now run container:
* in foreground:
```bash
docker run -it --rm $USER/ltb-self-service-password
```
* as a daemon:
```bash
docker run -d $USER/ltb-self-service-password
```

## Troubleshooting

#### What's going on ?
You can debug LDAP connection problems by adding this line in  `config.inc.php`:
```php
ldap_set_option(NULL, LDAP_OPT_DEBUG_LEVEL, 7);
```
Then inspect apache logs of a runnning container:
```bash
docker exec -it $(docker ps | grep 'ltb-self-service-password' | awk '{print $1}') tail /var/log/apache2/error.log
```

#### LDAPS with self-signed certificate
When connecting with LDAPS protocol to a server wtih a self-signed certificate, you will see this error in apache logs:
```
TLS: peer cert untrusted or revoked (0x42)
TLS: can't connect: (unknown error code).
```

Mount your self signed ca certificate into the container:

```bash
docker run --rm -it \
-v $(pwd)/assets/config.inc.php:/usr/share/self-service-password/conf/config.inc.php:ro \
-v $(pwd)/assets/int.rherzog.de-CA.crt:/usr/local/share/ca-certificates/int.rherzog.de-CA.crt:ro \
ralfherzog/docker-ltb-self-service-password:latest
```

##### The insecure way (not recommended)

Add this into `config.inc.php` to disable all certificate validation:
```php
putenv('LDAPTLS_REQCERT=never');
```

