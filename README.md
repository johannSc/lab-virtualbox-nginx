# lab-virtualbox

TP déploiement manuel

https://upcloud.com/resources/tutorials/configure-load-balancing-nginx

3 vms
Debian11 Nginx loadbalancer, client 1 & 2
Conf ip

* Nginx load balancer

Apt update / upgrade
Apt install nginx
Cd /etc/nginx
- Site available
- Site enable (défault)

Apt install curl
Curl localhost: success

Cd /var/www/html/
Mv *.html html.old
Touch index.html

	<html>
  <body>
    <p>hello from site 1</p>
  </body>
</html>

Systemctl restart nginx
Connaitre le DNS: nmcli

Test local: OK

* Clone VM Nginx client 1 et 2

- Changer l’ip, le dns (change Mac adresse)
- Vérifier la connexion via ping
- Modifier les index site2/3

* Nginx load balancer

Touch /etc/nginx/conf.f/load-balancer.conf

# Define which servers to include in the load balancing scheme. 
# It's best to use the servers' private IPs for better performance and security.
# You can find the private IPs at your UpCloud control panel Network section.
http {
   upstream backend {
      server 10.1.0.101; 
      server 10.1.0.102;
      server 10.1.0.103;
   }

   # This server accepts all traffic to port 80 and passes it to the upstream. 
   # Notice that the upstream name and the proxy_pass need to match.

   server {
      listen 8080; 

      location / {
          proxy_pass http://backend;
      }
   }
}


-> peut être supprimer le http du début si erreur lors du redémarrage de nginx

PLUS cycle de vie:
 upstream backend {
   server 10.1.0.101 weight=5;
   server 10.1.0.102 max_fails=3 fail_timeout=30s;
   server 10.1.0.103;
}
