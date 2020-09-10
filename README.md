nginx-icinga-nagios-plugin
==========================

This plugin is fixed version of [check_nginx](http://exchange.nagios.org/directory/Plugins/Web-Servers/nginx/check_nginx/details) by **eric_yang**

Instalation
============
This instctruction for centos icinga-client machine(nrpe), for another distributive, probably you will have different paths.

1. go to the plugin folder 

   `cd /usr/lib/nagios/plugins/` or `cd /usr/lib64/nagios/plugins/` for 64bit version
   
2. download latest version

 ```wget https://github.com/nicloay/nginx-icinga-nagios-plugin/raw/master/check_nginx```

3. go to nrpe config folder

 ```cd /etc/nrpe.d```
 
4. add nrpe configuration

 ```
 cat >> nginx.cfg <<EOF
 command[check_nginx]=/usr/bin/python /usr/lib/nagios/plugins/check_nginx -U 127.0.0.1:8088 -P /nginx_status -w 300 -c 500
 EOF
 ```
 
5. enable statictics for nginx
 ```
 cd /etc/nginx/conf.d/
 cat >> status.conf <<EOF
 server {
        listen 127.0.0.1:8088;
        server_name localhost;
        location /nginx_status {
                stub_status on;
                access_log   off;
                allow 127.0.0.1;
                deny all;
        }
  }
  ```
  
6. restart services
 ```
 service nrpe restart
 service nginx restart
 ```
7. add configuration on icinga host in my case it's icinga2 and here is configs which I keep in in different files
 ```
  object Host "nicloay.com" {
    import "generic-host"
    groups = [ "linux", "nginx", "php-fpm", "mysql-client" ]
    address = "private_ip"
  }
 
  object HostGroup "nginx" {
	  display_name = "Nginx Servers"
  }
  
  apply Service "remote-nginx" {
    import "generic-service"
    check_command = "nrpe"
    vars.nrpe_command = "check_nginx"
    assign where "nginx" in host.groups
    vars.sla = "24x7"
  }
  
 ```
 
 License
=======
This fix distribute under [WTFPL](https://en.wikipedia.org/wiki/WTFPL) license. for original code license please consult [developer](https://exchange.nagios.org/directory/Plugins/Web-Servers/nginx/check_nginx/details) of original plugin.
