# Bases de l'administration système Linux 

## Configuration du profil utilisateur  

Lors de notre première utilisation de la machine virtuelle nous devrons configurer un utilisateur afin de pouvoir se connecter par la suite. Cette utilisateur possedera une clé ssh lui permettant si il la renseigne dans le pc d'un de ses camaarades, de pouvoir accéder si ce dernier lui autorise. Cependant la connexion ssh ne pourra se faire que sur le même réseau pour des problèmes de sécurités et longues configuration au niveau du serveur. 

Première connexion au serveur : 
```
>> ssh student@172.16.235.45
>> student@172.16.235.45's password: hr#xU34}
```
Création du profil utilisateur :
```
Enter the new value, or press ENTER for the default
        Full Name []: Emmanuel
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] Y
Adding user `anicet_e' to group `student' ...
Adding user anicet_e to group student
Done.
```
Génération de la clé ssh :

```
1) Generate ssh key
2) Enter your public key
#? 1


 #######################################################
 #                                                     #
 # Creation d'une clef SSH :                           #
 #                                                     #
 #######################################################


Enter passphrase (empty for no passphrase):
Enter same passphrase again:


 #######################################################
 #                                                     #
 # Voici la clef PRIVEE :                              #
 #                                                     #
 #######################################################

-----BEGIN DSA PRIVATE KEY-----
    (secret)
-----END DSA PRIVATE KEY-----
```

Connexion au login existant :

```
>> ssh anicet_e@172.16.235.45
>> anicet_e@172.16.235.45's password:
```

## Etape 1 : Création des groupes et utilisateurs

Pour cette première étape nous créerons des groupes et des utilisateurs qu'on leurs assignera. 

Exemple de la création du groupe admin : 
```
>> sudo groupadd -g 3040 admin
```
Voir la liste des groupes existant :
```
>> cat /etc/group | awk -F: '{print $ 1}'
.
.
.
ucar_m
admin
anicet_e
```
### Création utilisateur version simplifié (useradd = low-level)

Cette étape traitera la création d'un utilisateur dit de "bas niveau".

Assignation d'un utilisateur à un groupe existant : 
```
>> sudo useradd -G admin alice
>> sudo passwd alice
Enter new UNIX password:alice
Retype new UNIX password:alice
passwd: password updated successfully
```
Voir les groupes d'un utilisateur : 
```
>> groups alice
alice : alice admin
```
L'utilisateur est créé et est directement ajouté au groupe admin.

### Création utilisateur version avancée (adduser = high-level)

Cette étape traitera la création d'un utilisateur dit de "haut niveau"
```
>> sudo adduser martine
Adding user `martine' ...
Adding new group `martine' (1010) ...
Adding new user `martine' (1010) with group `martine' ...
Creating home directory `/home/martine' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for martine
Enter the new value, or press ENTER for the default
        Full Name []: Martine
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] Y
```
Une fois créé l'utilisateur devra être assigné à un groupe. 
```
>> sudo usermod -a -G compta martine
```
Pour finir on vérifie si l'utilisateur est bien dans le groupe : 
```
>> groups martine
martine : martine compta
```

## Etape 2 : Installation de vim pour le groupe admin 

L'objectif de cette étape sera d'assigner des commandes et droits spécifiques à un groupe spécifique. 

Pour accéder au fichier "sudoers" représentant les droits il faut taper la commande suivante `sudo visudo`. Une fois dedans il faudra ajouter comme si dessous les droits root pour notre groupe :
```
# Members of the group 'admin'
%admin ALL=(ALL) NOPASSWD:ALL
```

L'utilisateur pourra enfin executer la commande suivante : 
```
>> sudo apt install vim
```

## Etape 3 : se connecter à son profil depuis une machine distante 

Pour se connecter à notre utilisateur bob depuis une machine distante nous devrons créer une clé ssh de 2048 bits à notre machine distante.  

```
client$> ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/bob/.ssh/id_rsa):
Created directory '/home/bob/.ssh'.
Enter passphrase (empty for no passphrase):
```

Sur le serveur de bob il faudra appliquer quelques configurations au fichier en faisant un `sudo nano /etc/ssh/sshd_config` : 
 
Mettre `PubkeyAuthentication yes` (fait par défaut) et ajouter `RSAAuthentication yes`. Il faudra vérifier si la ligne `#AuthorizedKeysFile     %h/.ssh/authorized_keys` est #commanté car le fichier séléctionné lors de l'authentification est `authorized_keys` (dans le dossier ssh) par defaut. Ensuite, verifier si la ligne `PasswordAuthentication no` est bien comme présentée.  

Une fois les modifications apportés, il faudra mettre la clé de la machine distante et surtout la votre, pour ne pas être bloqué lors du redemarrage du service. Pour se faire il faudra coller la longue clé publique connu sous le nom de `id_rsa.pub` dans le fichier `authorized_keys` qui se trouve dans le dossier `.ssh/` depuis votre ern`/home/utilisateur_concé`. N'hésitez pas à créer le fichier si il n'existe pas. 

Une fois cette étape fait, n'hésitez pas à vérifier de nouveau. Vous pourrez enfin redémarrer votre systemctl en faisant `sudo systemctl restart sshd`. Votre machine distante ainsi que votre machine pourra se connecter à la session de bob. 

## Etape 4 : installation et configuration de wordpress

Pour installer wordpress il faudra installer plusieurs dépendances comme le langage qui lui est propre ainsi qu'une base de donnée. Si jamais votre machine n'est pas à jour n'hésitez pas à `sudo apt update`, `sudo apt upgrade`.

Installation de Apache :
`sudo apt install apache2 `

Il faudra le démarrer ainsi que l'activer : 
`sudo systemctl start apache2 && sudo systemctl enable apache2 `

Installation de mysql-server : 
`sudo apt install mysql-server `

Pour démarrer mysql : 
`sudo systemctl start mysql `

Ensuite il faudra le configurer : 
```
>> sudo mysql_secure_installation 
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):my_secret_password

Change the root password? [Y/n] Y

New password:root
Re-enter new password:root

Remove anonymous users? [Y/n] n

Disallow root login remotely? [Y/n] n

Remove test database and access to it? [Y/n] Y

Reload privilege tables now? [Y/n] Y
```

Il faudra par la suite rentrer dans le shell de mysql pour faire les commandes suivantes : 
```
>> sudo mysql -u root -p 
Enter password:root
CREATE DATABASE mydb; 
GRANT ALL PRIVILEGES ON mydb.* TO votre_login@localhost identified BY 'votre_mot_de_passe'; 
FLUSH PRIVILEGES; 
exit 
```

Installation de php :
`sudo apt install php && sudo apt upgrade php`

Installation et configuration de Wordpress : 
```
>> sudo wget https://wordpress.org/latest.zip && sudo unzip latest.zip
>> sudo mv wordpress/* /var/www/html/ 
>> cd /var/www/html/
>> sudo rm index.html
```

Il faudra mettre à jour wordpress et php : 
```
sudo apt upgrade wordpress && sudo apt upgrade php
```

Une fois sur l'interface wordpress depuis votre navigateur en tapant l'ip de votre serveur sur la barre recherche. Il faudra renseigner quelques informations pour relier à la base de données et à l'utilisateur créé précedemment.<img width="960" alt="wordpress" src="https://user-images.githubusercontent.com/92017625/159031518-99e1194f-3c09-4cdb-b7bd-a4b5f2d4ae39.png">

De retour sur le cli, dans le dossier `/var/www/html` il faudra faire la commande `sudo touch wp-config.php && sudo nano wp-config.php` puis lui insérer le contenu comme indiqué par le site.
<img width="960" alt="wordpress2" src="https://user-images.githubusercontent.com/92017625/159031659-1f1fd357-9a57-4ff3-aad7-7d4fcc904482.png">

Pensez à bien sauvegarder le fichier puis cliqué sur "Run the installation" sur la page de wordpress". Le site va vous accompagner lors de la configuration de votre compte admin. Enfin vous pourrez vous connecter. 

<img width="960" alt="wordpress3" src="https://user-images.githubusercontent.com/92017625/159031694-2160ec04-0173-479a-99d1-86b571bc9f34.png">



