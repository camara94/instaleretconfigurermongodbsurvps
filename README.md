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
