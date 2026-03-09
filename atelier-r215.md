# Guide de déploiement d'une application AstroJS + PocketBase

## Certification de l'application
### Sur Infomaniak
1. Ajouter un enregistrement DNS qui fait pointer l'url de votre projet \<nomprojet\>.\<votredomaine\>.\<tld\> vers le VPS (185.157.244.202)


### Sur le VPS
2. Configurer Apache avec une nouvelle section VirtualHost
    - Ouvrir le fichier de configuration Apache
      ``` bash
      sudo nano /etc/apache2/sites-available/000-default.conf
      ```
    - Ajouter une section VirtualHost :   
      ``` apache
       <VirtualHost *:80>
             ServerName <nomprojet>.<votredomaine>.<tld>
      </VirtualHost>
       ```
    - Enregistrer et quitter nano
    
    - Relancer le serveur Apache
      ``` bash
        sudo /etc/init.d/apache2 restart
      ```
3. Lancer la certification du site
``` bash
sudo certbot certonly --apache
```   
et choisir le numéro du site à certifier


## Déployer PocketBase sur le VPS
### Sur le VPS
1.  Installer un **pocketbase vierge** 
   
``` bash
cd /var/www
sudo mkdir <nomprojet>
sudo chown -R etudiant:etudiant <nomprojet>
sudo chmod -R 755 <nomprojet>
cd <nomprojet>
wget https://github.com/pocketbase/pocketbase/releases/download/v0.36.6/pocketbase_0.36.6_linux_amd64.zip
unzip pocketbase_0.36.6_linux_amd64.zip -d .
```

### Sur la machine de développement 
2.  Copier les données du pocketbase local vers le pocketbase du VPS
``` bash
cd <dossier_pocketbase_local>
scp -P 23<XXX> -r pb_data etudiant@185.157.244.202:/var/www/<nomprojet> 
```

### Sur le VPS
3. Lancer le pocketbase sur le vps
``` bash
screen
cd /var/www/<nomprojet>
./pocketbase serve --http="0.0.0.0:<port_pocketbase>"
```
4. Pour quitter le mode screen appuyez sur ctrl A puis D

5. Configurer Apache en tant que proxy pour le PocketBase
``` bash
sudo nano /etc/apache2/sites-available/000-default.conf
```

Modifier la section VirtualHost de notre application de la manière suivante (la certification doit être déjà faite)

``` apache
<VirtualHost *:443>
    ServerName <nomprojet>.<votredomaine>.<tld>

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/<nomprojet>.<votredomaine>.<tld>/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/<nomprojet>.<votredomaine>.<tld>/privkey.pem


    ProxyPreserveHost On
    ProxyRequests Off

    ProxyPass /_/ http://localhost:<port_pocketbase>/_/
    ProxyPassReverse /_/ http://localhost:<port_pocketbase>/_/

    ProxyPass /api/ http://localhost:<port_pocketbase>/api/
    ProxyPassReverse /api/ http://localhost:<port_pocketbase>/api/
</VirtualHost>
```

Redémarrer Apache :

``` bash
sudo /etc/init.d/apache2 restart
```

## Test du déploiement
### Sur la navigateur 
Accéder à l'URL : 
https://\<nomprojet\>.\<votredomaine\>.\<tld\>/_/

