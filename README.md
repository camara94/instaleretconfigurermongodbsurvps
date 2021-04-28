# instaleretconfigurermongodbsurvps
Installation et configuration d'un serveur de base de donnée MongoDB sur un serveur linux 
## Conditions préalables
* Ubuntu Serveur 18.04 - 64 bit
* Avoir les privilèges **Root**
  
## Ce que nous allons faire dans ce tutoriel:
* Installer MongoDB
* Configurer MongoDB
* Conclusion
  
  ## Installez MongoDB sur Ubuntu 18.04 LTS
  ### Étape 1 - Importation de la clé publique
  Les clés GPG du distributeur de logiciels sont requises par le gestionnaire de packages Ubuntu apt (Advanced Package Tool) pour garantir la cohérence et l'authenticité du package. Exécutez cette commande pour importer les clés MongoDB sur votre serveur.
  <code>
    <pre>
        sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 68818C72E52529D4
    </pre>
  </code>

  ### Step 2 - Create source list file MongoDB
   Créez un fichier de liste MongoDB dans /etc/apt/sources.list.d/ avec cette commande:
   <code>
    <pre>
        sudo echo "deb http://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list
    </pre>
  </code>

### Étape 3 - Mettez à jour le référentiel
mettre à jour le référentiel avec la commande apt:
<code>
    <pre>
        sudo apt-get install -y mongodb-org
    </pre>
</code>

 ### Étape 4 - Installez MongoDB
 Vous pouvez maintenant installer MongoDB en tapant cette commande:
 <code>
    <pre>
        sudo apt-get install -y mongodb-org
    </pre>
</code>

Le programme d'installation d'**apt MongoDB** a créé automatiquement un fichier **mongod.service** pour **Systemd**, il n'est donc plus nécessaire de le créer manuellement.

Démarrez **MongoDB** et ajoutez-le en tant que service à démarrer au démarrage:
<code>
    <pre>
        sudo systemctl start mongod
        sudo systemctl enable mongod
    </pre>
</code>

Vérifiez maintenant que MongoDB a été démarré sur le port 27017 avec la commande netstat.

<code>
    <pre>
        sudo netstat -plntu
    </pre>
</code>

![image 1](images/1.png)

## Configurer le nom d'utilisateur et le mot de passe MongoDB
### Étape 1 - Ouvrez la coque mongo
Avant de configurer un nom d'utilisateur et un mot de passe pour MongoDB, vous devez ouvrir le shell MongoDB sur votre serveur. Vous pouvez vous connecter en tapant:

<code>
    <pre>
        mongo
    </pre>
</code>

Si vous obtenez une erreur Échec de l'initialisation globale: BadValue Non valide ou aucun paramètre régional utilisateur n'est défini. Veuillez vous assurer que les variables d'environnement LANG et / ou LC_ * sont correctement définies, essayez la commande:

<code>
    <pre>
        export LC_ALL=C
        mongo 
    </pre>
</code>

### Étape 2 - Basculez vers l'administrateur de la base de données
Une fois que vous êtes dans le shell MongoDB, passez à la base de données nommée admin:

<code>
    <pre>
        use admin
    </pre>
</code>

### Étape 3 - Créez l'utilisateur root
Créez l'utilisateur root avec cette commande:

<code>
    <pre>
        db.createUser({user:"admin", pwd:"admin123", roles:[{role:"root", db:"admin"}]})
    </pre>
</code>

Desc: crée l'utilisateur **admin** avec le mot de passe **admin123** et avoir l'autorisation rôle en tant que **root** de la base de données est **admin**.

![use](images/2.png)

Tapez maintenant <code>exit</code> pour quitter le shell MongoDB.

<code>
    <pre>
        exit
    </pre>
</code>

Et vous êtes de retour sur le shell de Linux.

### Étape 4 - Activez l'authentification MongoDB
Modifiez le fichier de service mongodb **'/lib/systemd/system/mongod.service'** avec votre éditeur.

<code>
    <pre>
        sudo nano /lib/systemd/system/mongod.service
    </pre>
</code>

A la ligne 9 **ExecStart**, ajoutez la nouvelle option **--auth**.

<code>
    <pre>
        ExecStart=/usr/bin/mongod --auth --config /etc/mongod.conf
    </pre>
</code>

Enregistrez le fichier de service et quittez nano.

Rechargez le **service systemd**:

<code>
    <pre>
        sudo systemctl daemon-reload
    </pre>
</code>

### Étape 5 - Redémarrez MongoDB et essayez de vous connecter

Redémarrez maintenant MongoDB et connectez-vous avec l'utilisateur créé.

<code>
    <pre>
        sudo service mongod restart
    </pre>
</code>

et connectez-vous au shell MongoDB avec cette commande:

<code>
    <pre>
        mongo -u admin -p admin123 --authenticationDatabase admin
    </pre>
</code>

et vous verrez la sortie comme ceci:
![3](images/3.png)

## Activer l'accès externe et configurer le pare-feu UFW
**UFW** est le pare-feu par défaut dans Ubuntu. Dans ce chapitre, je montrerai comment configurer UFW pour autoriser l'accès externe à MongoDB.

Vérifiez l'état de l'UFW

<code>
    <pre>
        sudo ufw status
    </pre>
</code>

Lorsque le résultat est:

<code>
    <pre>
        Status: inactive
    </pre>
</code>

Activez **UFW** avec cette commande et ouvrez d'abord le port SSH s'il est connecté par **SSH**:

<code>
    <pre>
        sudo ufw allow ssh
        sudo ufw enable
    </pre>
</code>

avant de passer aux étapes suivantes.

Pour des raisons de sécurité, vous ne devez autoriser l'accès au port MongoDB 27017 qu'à partir des adresses **IP** qui doivent accéder à la base de données. Par défaut, localhost est toujours en mesure d'y accéder, donc pas besoin d'ouvrir le port MongoDB pour **IP 127.0.0.1**.

### Syntaxe du pare-feu UFW

Pour autoriser l'accès de l'IP externe 192.168.1.10 à MongoDB, utilisez cette commande:

<code>
    <pre>
        sudo ufw allow from 152.228.217.119 to any port 27017
    </pre>
</code>

Remplacez l'adresse IP dans la commande ci-dessus par l'IP externe que vous souhaitez autoriser à accéder à MongoDB.

Si vous souhaitez ouvrir le port MongoDB pour n'importe quelle adresse IP, par exemple si vous l'exécutez sur un réseau local et que tous les systèmes de ce réseau pourront accéder à MongoDB, utilisez cette commande:

<code>
    <pre>
        sudo ufw allow 27017
    </pre>
</code>

MongoDB écoute localhost par défaut, pour rendre la base de données accessible de l'extérieur, nous devons la reconfigurer pour écouter également sur l'adresse IP du serveur.

Ouvrez le fichier mongod.conf dans l'éditeur nano:

<code>
    <pre>
        sudo nano /etc/mongod.conf
    </pre>
</code>

et ajoutez l'adresse IP du serveur dans la ligne bind_ip comme ceci:

<code>
    <pre>
        # network interfaces
        net:
            port: 27017
            bindIp: 127.0.0.1,152.228.217.119
    </pre>
</code>

Remplacez **152.228.217.119** par l'adresse IP de votre serveur, puis redémarrez **MongoDB** pour appliquer les modifications.

<code>
    <pre>
       sudo service mongod restart
    </pre>
</code>

Vous pouvez maintenant accéder au serveur de base de données MongoDB via le réseau.

Par exemple, on peut spécifier à la fois PLAIN et SCRAM-SHA-256 comme mécanismes d'authentification, utilisez la commande suivante:

<code>
    <pre>
       mongod --setParameter authenticationMechanisms=PLAIN,SCRAM-SHA-256 --auth
    </pre>
</code>

## Conclusion
**MongoDB** est une base de données **NoSQL** bien connue qui offre des **performances élevées**, une **haute disponibilité** et une mise à l'échelle automatique. Il diffère des **SGBDR** tels que **MySQL**, **PostgreSQL** et **SQLite** car il n'utilise pas **SQL** pour définir et récupérer des **données**. **MongoDB** stocke les données dans des `documents` appelés **BSON (représentation binaire de JSON avec des informations supplémentaires)**. **MongoDB** n'est disponible que pour la version Ubuntu de support à long terme 64 bits.