# 5 Minutes Stacks, épisode 24 : CozyCloud #

## Episode 24 : CozyCloud

![CozyCloudlogo](http://blog.jingleweb.fr/wp-content/uploads/2015/03/bighappycloud.png)

Cozycloud est votre serveur de cloud personnel libre. A la différence des autres serveurs cloud personnels auto-hébergeables, Cozy met l'accent sur les applications et la collaboration de ses applications autour de vos données personnelles. Cozy est une solution de PaaS (Platform as a Service) personnel qui vous permet de déployer des applications web personnelles en un clic. Il ne s'agit pas là de simples greffons mais d'applications web riches. Vous pouvez choisir parmi les applications Cozy existantes (Notes, Todos, Agenda, Contacts, Photos…), adapter une application Node.js existante ou commencer votre propre application web « from-scratch »(documentation et tutoriaux disponibles).

Une particularité de Cozy est la centralisation du stockage des différentes applications dans une base de données commune avec des données typées et un contrôle des accès par type de donnée. De cette manière les différentes applications travaillent avec la même source de données (contacts, mails, notes…). Cozycloud est pour le moment centré sur Node.js mais le support d'applications Python et Ruby est prévu. De plus cozy est développé en **France** par des développeur **Français**.  

## Preparations

### Les versions
 - Ubuntu Trusty 14.04.2
 - Cozy 2.0

### Les pré-requis pour déployer cette stack
 Ceci devrait être une routine à présent:

 * Un accès internet
 * Un shell linux
 * Un [compte Cloudwatt](https://www.cloudwatt.com/cockpit/#/create-contact) avec une [ paire de clés existante](https://console.cloudwatt.com/project/access_and_security/?tab=access_security_tabs__keypairs_tab)
 * Les outils [OpenStack CLI](http://docs.openstack.org/cli-reference/content/install_clients.html)
 * Un clone local du dépôt git [Cloudwatt applications](https://github.com/cloudwatt/applications)

### Taille de l'instance
 Par défaut, le script propose un déploiement sur une instance de type "Small" (s1.cw.small-1). Il
 existe une variété d'autres types d'instances pour la satisfaction de vos multiples besoins. Les instances sont facturées à la minute, vous permettant de payer uniquement pour les services que vous avez consommés et plafonnées à leur prix mensuel (vous trouverez plus de détails sur la [Page tarifs](https://www.cloudwatt.com/fr/produits/tarifs.html) du site de Cloudwatt).

 Vous pouvez ajuster les parametres de la stack à votre goût.

### Au fait...

 Si vous n’aimez pas les lignes de commande, vous pouvez passer directement à la version ["Je lance avec la console"](#console)...

## Tour du propriétaire

 Une fois le dépôt cloné, vous trouverez le répertoire `bundle-trusty-cozycloud/`

 * `bundle-trusty-cozycloud.heat.yml`: Template d'orchestration HEAT, qui servira à déployer l'infrastructure nécessaire.
 * `stack-start.sh`: Scipt de lancement de la stack, qui simplifie la saisie des parametres et sécurise la création du mot de passe admin.
 * `stack-get-url.sh`: Script de récupération de l'IP d'entrée de votre stack, qui peut aussi se trouver dans les parametres de sortie de la stack.

## Démarrage

### Initialiser l'environnement

 Munissez-vous de vos identifiants Cloudwatt, et cliquez [ICI](https://console.cloudwatt.com/project/access_and_security/api_access/openrc/).
 Si vous n'êtes pas connecté, vous passerez par l'écran d'authentification, puis le téléchargement d'un script démarrera. C'est grâce à celui-ci que vous pourrez initialiser les accès shell aux API Cloudwatt.

 Sourcez le fichier téléchargé dans votre shell et entrez votre mot de passe lorsque vous êtes invité à utiliser les clients OpenStack.

 ~~~ bash
 $ source COMPUTE-[...]-openrc.sh
 Please enter your OpenStack Password:

 ~~~

 Une fois ceci fait, les outils de ligne de commande d'OpenStack peuvent interagir avec votre compte Cloudwatt.


### Ajuster les paramètres

 Dans le fichier `bundle-trusty-cozycloud.heat.yml` vous trouverez en haut une section `parameters`. Le seul paramètre obligatoire à ajuster
 est celui nommé `keypair_name` dont la valeur `default` doit contenir le nom d'une paire de clés valide dans votre compte utilisateur.
 C'est dans ce même fichier que vous pouvez ajuster la taille de l'instance par le paramètre `flavor`.

 ~~~ yaml
 heat_template_version: 2013-05-23


 description: All-in-one CozyCloud stack


 parameters:
   keypair_name:
     default: my-keypair-name                   <-- Rajoutez cette ligne avec le nom de votre paire de clés
     description: Keypair to inject in instance
     label: SSH Keypair
     type: string

   flavor_name:
     default: s1.cw.small-1
     description: Flavor to use for the deployed instance
     type: string
     label: Instance Type (Flavor)
 [...]
 ~~~
### Démarrer la stack

 Dans un shell, lancer le script `stack-start.sh` en passant en paramètre le nom que vous souhaitez lui attribuer :

 ~~~ bash
 $ ./stack-start.sh CozyCloud
 +--------------------------------------+------------+--------------------+----------------------+
 | id                                   | stack_name | stack_status       | creation_time        |
 +--------------------------------------+------------+--------------------+----------------------+
 | ed4ac18a-4415-467e-928c-1bef193e4f38 | Cozycloud  | CREATE_IN_PROGRESS | 2015-04-21T08:29:45Z |
 +--------------------------------------+------------+--------------------+----------------------+
 ~~~

 Enfin, attendez **5 minutes** que le déploiement soit complet.

 ~~~
 $ heat resource-list EXP_STACK
 +------------------+-----------------------------------------------------+---------------------------------+-----------------+----------------------+
 | resource_name    | physical_resource_id                                | resource_type                   | resource_status | updated_time         |
 +------------------+-----------------------------------------------------+---------------------------------+-----------------+----------------------+
 | floating_ip      | 44dd841f-8570-4f02-a8cc-f21a125cc8aa                | OS::Neutron::FloatingIP         | CREATE_COMPLETE | 2015-11-25T11:03:51Z |
 | security_group   | efead2a2-c91b-470e-a234-58746da6ac22                | OS::Neutron::SecurityGroup      | CREATE_COMPLETE | 2015-11-25T11:03:52Z |
 | network          | 7e142d1b-f660-498d-961a-b03d0aee5cff                | OS::Neutron::Net                | CREATE_COMPLETE | 2015-11-25T11:03:56Z |
 | subnet           | 442b31bf-0d3e-406b-8d5f-7b1b6181a381                | OS::Neutron::Subnet             | CREATE_COMPLETE | 2015-11-25T11:03:57Z |
 | server           | f5b22d22-1cfe-41bb-9e30-4d089285e5e5                | OS::Nova::Server                | CREATE_COMPLETE | 2015-11-25T11:04:00Z |
 | floating_ip_link | 44dd841f-8570-4f02-a8cc-f21a125cc8aa-`floating IP`  | OS::Nova::FloatingIPAssociation | CREATE_COMPLETE | 2015-11-25T11:04:30Z |
   +------------------+-----------------------------------------------------+-------------------------------+-----------------+----------------------
 ~~~

   Le script `start-stack.sh` s'occupe de lancer les appels nécessaires sur les API Cloudwatt pour :

   * démarrer une instance basée sur Ubuntu trusty, pré-provisionnée avec la stack backup,
   * l'exposer sur Internet via une IP flottante.

![cozycloudlogo](http://www.woinux.fr/wp-content/uploads/2013/10/cozy-banner.jpg)

### C’est bien tout ça, mais vous n’auriez pas un moyen de lancer l’application par la console ?

Et bien si ! En utilisant la console, vous pouvez déployer un serveur cozycloud:

1.	Allez sur le Github Cloudwatt dans le répertoire [applications/bundle-trusty-cozycloud](https://github.com/cloudwatt/applications/tree/master/bundle-trusty-cozycloud)
2.	Cliquez sur le fichier nommé `bundle-trusty-cozycloud.heat.yml`
3.	Cliquez sur RAW, une page web apparait avec le détail du script
4.	Enregistrez-sous le contenu sur votre PC dans un fichier avec le nom proposé par votre navigateur (enlever le .txt à la fin)
5.  Rendez-vous à la section « [Stacks](https://console.cloudwatt.com/project/stacks/) » de la console.
6.	Cliquez sur « Lancer la stack », puis cliquez sur « fichier du modèle » et sélectionnez le fichier que vous venez de sauvegarder sur votre PC, puis cliquez sur « SUIVANT »
7.	Donnez un nom à votre stack dans le champ « Nom de la stack »
8.	Entrez votre keypair dans le champ « keypair_name »
9.  Donner votre passphrase qui servira pour le chiffrement des sauvegardes
10.	Choisissez la taille de votre instance parmi le menu déroulant « flavor_name » et cliquez sur « LANCER »

La stack va se créer automatiquement (vous pouvez en voir la progression cliquant sur son nom). Quand tous les modules deviendront « verts », la création sera terminée. Vous pourrez alors aller dans le menu « Instances » pour découvrir l’IP flottante qui a été générée automatiquement. Ne vous reste plus qu'à vous connecter en ssh avec votre keypair.

C’est (déjà) FINI !

### Vous n’auriez pas un moyen de lancer l’application en 1-clic ?

Bon... en fait oui ! Allez sur la page [Applications](https://www.cloudwatt.com/fr/applications/index.html) du site de Cloudwatt, choisissez l'appli, appuyez sur DEPLOYER et laisser vous guider... 2 minutes plus tard un bouton vert apparait... ACCEDER : vous avez votre cozy !

### Enjoy

Une fois tout ceci fait vous pouvez vous connecter sur votre serveur en SSH en utilisant votre keypair préalablement téléchargé sur votre poste,

Vous etes maintenant en possession de votre propre serveur de cloud, vous pouvez y acceder via l'url `https://ip-floatingip.rev.cloudwatt.com` en replacant les `.` de votre floating IP par des `-` (exemple: ip-10-11-12-13.rev.cloudwatt.com). Votre url complète sera présente dans le fichier `/etc/ansible/cozy-vars.yml`.

Un certificat SSL est automatiquement généré via Let's encrypt et celui-ci est renouvellé via un job CRON tous les 90 jours,

Vous pouvez à présent télécharger l'application android cozy et faire une synchronisation de vos données avec votre cozy, celui ci etant hébergé en france dans un environnement maitrisé vous pouvez faire une totale confiance dans cette application.

![prezcozy](http://www.usine-digitale.fr/mediatheque/2/3/2/000337232_homePageUne/cozy-cloud.jpg)

Sur le bureau de votre cozy vous y trouverez un bouton `Store` qui sera votre marketplace, vous pouvez y installer un serveur de mail ou encore un ghost. La liste se rempli de jour en jour, de plus les contributions via des dépot git sont possible, la communauté cozy s'agrandi à vu d'oeil.

## So watt ?

Ce tutoriel a pour but d'accélerer votre démarrage. A ce stade vous êtes maître(sse) à bord.

Vous avez un point d'entrée sur votre machine virtuelle en SSH via l'IP flottante exposée et votre clé privée (utilisateur `cloud` par défaut).

* Vous avez accès à l'interface web en https via l'adresse indiquée dans le fichier `/etc/ansible/cozy-vars.yml`.

* Voici quelques sites d'informations avant d'aller plus loin :

  - https://cozy.io/fr/    
  - http://blog.jingleweb.fr/2015/03/pourquoi-je-passe-a-my-cozy-cloud/  
  - http://korben.info/my-cozy-cloud-degaine-sa-version-2-0.html/


----
Have fun. Hack in peace.
