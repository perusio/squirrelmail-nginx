# Nginx configuration for SquirrelMail

## Introduction 

This is a nginx configuration for
[SquirrelMail](http://www.squirrelmail.org). A webmail application
written in PHP.

## Features

 1. Filtering of invalid HTTP `Host` headers.
 
 2. Specific locations for all the scripts and plugins.
 
 3. Matching of all `.htaccess` files protections with Nginx.
 
 4. Use of the
    [open files cache](http://nginx.org/en/docs/http/ngx_http_core_module.html#open_file_cache)
    for **faster** static file serving.
    
 5. All documents are protected.
 
 6. HTTPS enabled host.
 
 7. Support for safely running the `attach` and `data` directories
    **outside** of the `/var/www` root filesystem.
    
 8. Protection against running unauthorized PHP scripts.
 
 9. Disable of crawling with inline `robots.txt` file.

## IPv6 and IPv4

The configuration of the example vhosts uses **separate** sockets for
IPv6 and IPv4. This way is simpler for those not (yet) having IPv6
support to disable it by commenting out the
[`listen`](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen)
directive with the `ipv6only=on` parameter.

Note that the IPv6 address uses an IP _stolen_ from the
[IPv6 Wikipedia page](https://en.wikipedia.org/wiki/IPv6). You **must
replace** the indicated address by **your** address.

## Installation

    1. Move the old `/etc/nginx` directory to `/etc/nginx.old`.
   
   2. Clone the git repository from github:
   
      `git clone https://github.com/perusio/squirrelmail-nginx.git /etc/nginx`
   
   3. Edit the `sites-available/webmail.example.com.conf` and/or the
      `sites-available/secure.webmail.com.conf` when using the HTTPS
      host configuration file(s) to suit your needs. Especially
      replace stats.example.com with **your** domain.
   
      Since the credentials are sent over the wire from your browser
      to the server it's **highly recommended** that you use an HTTPS
      host rather than a mere HTTP host.
   
   4. Setup the PHP handling method. It can be:
   
      + Upstream HTTP server like Apache with mod_php. To use this
        method comment out the 
          `include upstream_phpcgi.conf;`
        line in `nginx.conf` and uncomment the lines:
        
            include reverse_proxy.conf;
            include upstream_phpapache.conf;

        Now you must set the proper address and port for your
        backend(s) in the `upstream_phpapache.conf`. By default it
        assumes the loopback `127.0.0.1` interface on port
        `8080`. Adjust accordingly to reflect your setup.

        Comment out **all** `fastcgi_pass` directives in
        `stats.example.com.conf` Uncomment out all the `proxy_pass`
        directives. They have a comment around them, stating these
        instructions.
      
      + FastCGI process using php-cgi. In this case an
        [init script](https://github.com/perusio/php-fastcgi-debian-script
        "Init script for php-cgi") is
        required. This is how the server is configured out of the
        box. It uses UNIX sockets. You can use TCP sockets if you prefer.
      
      + [PHP FPM](http://www.php-fpm.org "PHP FPM"), this requires you
        to configure your fpm setup, in Debian/Ubuntu this is done in
        the `/etc/php5/fpm` directory.
        
        Look [here](https://github.com/perusio/php-fpm-example-config) for
        an **example configuration** of `php-fpm`.
        
      Check that the socket is properly created and is listening. This
      can be done with `netstat`, like this for UNIX sockets:
      
         netstat --unix -l
         
      or like this for TCP sockets:    
                  
         netstat -t -l
   
      It should display the PHP CGI socket.
   
      Note that the default socket type is UNIX and the config assumes
      it to be listening on `unix:/tmp/php-cgi/php-cgi.socket`, if
      using the `php-cgi`, or in `unix:/var/run/php-fpm.sock` using
      `php-fpm` and that you should **change** to reflect your setup
      by editing `upstream_phpcgi.conf`.

   5. Create the `/etc/nginx/sites-enabled` directory and enable the
      virtual host using one of the methods described below.
    
      Note that if you're using the
      [nginx_ensite](http://github.com/perusio/nginx_ensite) script
      described below it **creates** the `/etc/nginx/sites-enabled`
      directory if it doesn't exist the first time you run it for
      enabling a site.
    
   6. Reload Nginx:
   
      `/etc/init.d/nginx reload`
   
   7. Check that your site is working using your browser.
   
   8. Remove the `/etc/nginx.old` directory.
   
   9. Done.

## Getting the latest Nginx packaged for Debian or Ubuntu

   I maintain a [debian repository](http://debian.perusio.net/unstable
   "my debian repo") with the
   [latest](http://nginx.org/en/download.html "Nginx source download")
   version of Nginx. This is packaged for Debian **unstable** or
   **testing**. The instructions for using the repository are
   presented on this [page](http://debian.perusio.net/debian.html
   "Repository instructions").
 
   It may work or not on Ubuntu. Since Ubuntu seems to appreciate more
   finding semi-witty names for their releases instead of making clear
   what's the status of the software included. Is it **stable**? Is it
   **testing**? Is it **unstable**? The package may work with your
   currently installed environment or not. I don't have the faintest
   idea which release to advise. So you're on your own. Generally the
   APT machinery will sort out for you any dependencies issues that
   might exist.

## Other Nginx configs on github

   + [Drupal](https://github.com/perusio/drupal-with-nginx "Drupal
     Nginx configuration")
     
   + [WordPress](https://github.com/perusio/wordpress-nginx "WordPress Nginx
     configuration")
     
   + [Chive](https://github.com/perusio/chive-nginx "Chive Nginx
     configuration")

   + [Piwik](https://github.com/perusio/piwik-nginx "Piwik Nginx configuration")

   + [Redmine](https://github.com/perusio/redmine-nginx "Redmine Nginx
     configuration")

## Securing your PHP configuration

   There's a small shell script that parses your `php.ini` and
   sets a sane environment, be it for **development** or
   **production** settings. 
   
   Grab it [here](https://github.com/perusio/php-ini-cleanup "PHP
   cleanup script").
