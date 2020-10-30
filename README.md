# Network-Automation
In this assignment, you are to implement a simple web service. The service is really simple, and the content is shown below.

<?php echo date("Y-m-d H:i:s") . " Welcome to " . gethostname() . "|" .$_SERVER['SERVER_ADDR'] .":" . $_SERVER['SERVER_PORT'] ."<br>\n"; ?> 
This line is to be stored in the main index.php file of a webserver, furthermore, the server should serve content from the "/var/www/html" folder. The webserver that you are to use is Nginx, and it should be deployed on a Ubuntu 18.04LTS or Ubuntu 20.04LTS (preferred). For the service to work properly, you need to install the following packages; nginx, php, and php-fpm. 

 

Below you can find an example of a generic Nginx configuration, that could act as a template. 

server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;
        index index.php index.html index.htm index.nginx-debian.html;

        server_name _;
        location / {
                try_files \$uri \$uri/ =404;
        }
        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/run/php/php7.4-fpm.sock;
        }
}
So the basic service is an Nginx web server, that only serves one simple PHP page. That page renders the hostname, IP, and port of the server running it. 

 

However, we expect our service to be a massive hit with our intended customers, so we cannot just run one web server. We need to run multiple web servers. Initially, we want to have three Nginx servers running on three different platforms. To make access simple, we want HAproxy to load balance between these three servers. 

Hence, the logical network/service map would be as follows:

(Internet)--->HAproxy
             (internal network using private address range; 10.0.1.0/27)
              +--->A
              +--->B
              +--->C
(Internet)--->Bastion
The site consists of 5 hosts; HAproxy, A, B, C, Bastion, and an internal site-local network. HAproxy is the host that runs HAproxy and acts as an entry point for accessing the service. A, B, and C are the three web servers. These are only to use private addresses from the site-local network. The last host, Bastion, is acting as an SSH entry point to the internal network. I.e. if you connect to Bastion, you can ssh to all of the other hosts within the site-local network. 

Your task is as follows; 
Write an ansible-playbook, site.yaml, that deploys the HAproxy, and Nginx on appropriate existing hosts, i.e. the playbook does not need to launch any Virtual Machines. Once the hosts have been configured, the 

The playbook is assumed to run -outside- the site, but -via- the Bastion host. Hence, you need to have an SSH config file that allows your host to use the Bastion host as a jump host, using an SSH key. Furthermore, the Bastion host also needs to have SSH access to all of the site-local hosts, using SSH keys, to avoid typing passwords all the time. The playbook should also do a rudimentary function test of the deployed application. That test is to send three requests to the public IP associated with HAproxy, and verify that all three hosts A, B, and C answers. 

As to mitigate problems, below you find the host file that your playbook needs to use. 

[haproxy] 
devhaproxy 

[webservers] 
devA 
devB 
devC 

[all:vars] 
ansible_user=ubuntu 
##REMOVED sshkey entry, handled by external SSH config file ##
Name your bastion host bastionET2598 (for SSH config)
