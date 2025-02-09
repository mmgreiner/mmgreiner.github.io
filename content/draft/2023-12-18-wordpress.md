---
categories:
- publishing
date: "2023-12-18"
draft: true
tags:
- php
- wordpress
title: Using PHP and Wordpress locally
showToc: false
---

## Install phpmyadmin with brew

~~~~
==> Pouring phpmyadmin--5.2.1.sonoma.bottle.tar.gz
==> Caveats
To enable phpMyAdmin in Apache, add the following to httpd.conf and
restart Apache:
    Alias /phpmyadmin /usr/local/share/phpmyadmin
    <Directory /usr/local/share/phpmyadmin/>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        <IfModule mod_authz_core.c>
            Require all granted
        </IfModule>
        <IfModule !mod_authz_core.c>
            Order allow,deny
            Allow from all
        </IfModule>
    </Directory>
Then open http://localhost/phpmyadmin
The configuration file is /usr/local/etc/phpmyadmin.config.inc.php
==> Summary
🍺  /usr/local/Cellar/phpmyadmin/5.2.1: 4,191 files, 48.1MB
==> Running `brew cleanup phpmyadmin`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
~~~~

Where is httpd.conf?

~~~~
% httpd -V                        
[Mon Dec 18 07:42:30.160858 2023] [so:notice] [pid 4802] AH06662: Allowing module loading process to continue for module at /usr/local/opt/php/lib/httpd/modules/libphp.so because module signature matches authority "Markus Greiner" specified in LoadModule directive
Server version: Apache/2.4.56 (Unix)
Server built:   Sep 30 2023 04:15:00
Server's Module Magic Number: 20120211:126
Server loaded:  APR 1.5.2, APR-UTIL 1.5.4, PCRE 8.44 2020-02-12
Compiled using: APR 1.5.2, APR-UTIL 1.5.4, PCRE 8.44 2020-02-12
Architecture:   64-bit
Server MPM:     prefork
  threaded:     no
    forked:     yes (variable process count)
Server compiled with....
 -D APR_HAS_SENDFILE
 -D APR_HAS_MMAP
 -D APR_HAVE_IPV6 (IPv4-mapped addresses enabled)
 -D APR_USE_FLOCK_SERIALIZE
 -D APR_USE_PTHREAD_SERIALIZE
 -D SINGLE_LISTEN_UNSERIALIZED_ACCEPT
 -D APR_HAS_OTHER_CHILD
 -D AP_HAVE_RELIABLE_PIPED_LOGS
 -D DYNAMIC_MODULE_LIMIT=256
 -D HTTPD_ROOT="/usr"
 -D SUEXEC_BIN="/usr/bin/suexec"
 -D DEFAULT_PIDLOG="/private/var/run/httpd.pid"
 -D DEFAULT_SCOREBOARD="logs/apache_runtime_status"
 -D DEFAULT_ERRORLOG="logs/error_log"
 -D AP_TYPES_CONFIG_FILE="/private/etc/apache2/mime.types"
 -D SERVER_CONFIG_FILE="/private/etc/apache2/httpd.conf"
~~~~

**`-D SERVER_CONFIG_FILE=`**

Add to the file.

restart apache

    sudo apachectl restart   

now you should see it under: <http://localhost/phpmyadmin>



## Installing wordpress

https://developer.wordpress.org/advanced-administration/before-install/howto-install/


Password: EM7zcN0*WmIwZ7ml)s
