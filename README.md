# Lab-virtualbox

- [Architecture](#architecture)
- [Déploiement du load-balancer (VM1)](#déploiement-du-load-balancer-(vm1))
- [Déploiement VM2 et VM3](#déploiement-vm2-et-vm3)
- [Vérification](#vérification)

## Architecture:

  * 3 vms
  * Debian11 Nginx loadbalancer, client 1 & 2
  * schéma de présentation

## Déploiement du load-balancer (VM1)

```
apt update / upgrade
apt install nginx
cd /etc/nginx
```
Doit comporter:
```
- Site available
- Site enable (défault)
```

Installation de l'outil curl pour tester une page web en local;
```
apt install curl
curl localhost: success
```
On modifie la page d'accueil:

```
cd /var/www/html/
Mv *.html html.old
nano index.html
```

```
<html>
  <body>
    <p>hello from site 1</p>
  </body>
</html>
```

Puis on redémarre le service:

```
Systemctl restart nginx
```

Tips: modification / information des paramètres locaux de la VM

```
nmcli
```

Test local: si OK, on poursuit

## Déploiement VM2 et VM3

- Clone VM Nginx client 1 et 2 via virtualbox

  * Changer l’ip, le dns (change Mac adresse)
  * Vérifier la connexion via ping
  * Modifier les index site2/3

- Activation coté nginx load balancer (VM1)

```
touch /etc/nginx/conf.f/load-balancer.conf
```

Puis le peupler:

```
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
```

TIPS: **peut être supprimer le http du début si erreur lors du redémarrage de nginx**

TIPS2: possibilité de paramétrer un cycle de vie:

```
upstream backend {
   server 10.1.0.101 weight=5;
   server 10.1.0.102 max_fails=3 fail_timeout=30s;
   server 10.1.0.103;
}
```

## Vérification

Depuis un navigateur sur votre poste de travail, naviguer sur 10.1.0.101:8080 et raffraichissez la page. Vous aurez aléatoirement un des 3 sites qui répondra.

Si ok, le load-balancing est fonctionnel.

En cas de la configuration avec le cycle de vie adapté, vous pouvez tester d'arrêter une VM ou le service nginx, les autres serveurs garantiront le service.

source: https://upcloud.com/resources/tutorials/configure-load-balancing-nginx
