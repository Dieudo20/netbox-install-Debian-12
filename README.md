# netbox-install-Debian-12
Install netbox debian 12

# Mise à jour du système :
    sudo apt update
    sudo apt upgrade

# Installation des dépendances :
    sudo apt install -y python3 python3-pip python3-venv libpq-dev libxml2-dev libxslt1-dev libffi-dev libssl-dev zlib1g-dev

# Création d'un utilisateur dédié :
    sudo adduser --system --group netbox

# Installation de PostgreSQL :
    sudo apt install -y postgresql postgresql-contrib

# Configuration de PostgreSQL :
    sudo -u postgres psql

    CREATE DATABASE netbox;
    CREATE USER netboxuser WITH PASSWORD 'mot_de_passe';
    ALTER DATABASE netbox OWNER TO netboxuser;
    \q

# Installation de NetBox :

    git clone -b master --depth 1 https://github.com/netbox-community/netbox.git /opt/netbox
    cd /opt/netbox
    sudo -u netbox python3 -m venv venv
    sudo -u netbox /opt/netbox/venv/bin/pip install -r requirements.txt
    sudo -u netbox ln -s /opt/netbox/netbox/netbox/configuration.py /opt/netbox/netbox/configuration.py.orig

# Configuration de NetBox :
    sudo nano /opt/netbox/netbox/configuration.py

# Initialisation de la base de données :
# Appliquez les migrations et chargez les données initiales :

    sudo -u netbox /opt/netbox/venv/bin/python3 /opt/netbox/netbox/manage.py migrate
    sudo -u netbox /opt/netbox/venv/bin/python3 /opt/netbox/netbox/manage.py createsuperuser
    sudo -u netbox /opt/netbox/venv/bin/python3 /opt/netbox/netbox/manage.py loaddata initial_data

# Configuration du serveur web 
    sudo apt install apache2

# Configuration du virtual host :
    sudo nano /etc/apache2/sites-available/netbox.conf

# Ajoutez le contenu suivant dans ce fichier :
<VirtualHost *:80>
    ServerName netbox.example.com  # Remplacez par votre nom de domaine ou adresse IP
    DocumentRoot /opt/netbox/netbox/static/
    
    Alias /static/ /opt/netbox/netbox/static/
    Alias /media/ /opt/netbox/netbox/media/
    
    <Directory /opt/netbox/netbox/static>
        Require all granted
    </Directory>
    
    <Directory /opt/netbox/netbox/media>
        Require all granted
    </Directory>
    
    <Directory /opt/netbox/netbox/netbox>
        <Files wsgi.py>
            Require all granted
        </Files>
    </Directory>
    
    WSGIDaemonProcess netbox python-home=/opt/netbox/venv python-path=/opt/netbox
    WSGIProcessGroup netbox
    WSGIScriptAlias / /opt/netbox/netbox/netbox/wsgi.py
    
    ErrorLog ${APACHE_LOG_DIR}/netbox_error.log
    CustomLog ${APACHE_LOG_DIR}/netbox_access.log combined
</VirtualHost>

# Activer le virtual host :
      sudo a2ensite netbox.conf
      sudo systemctl restart apache2

# Configuration du pare-feu :
    sudo ufw allow 80/tcp

# Test de connexion :










