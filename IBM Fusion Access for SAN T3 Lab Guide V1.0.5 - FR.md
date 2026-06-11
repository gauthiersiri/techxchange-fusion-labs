# IBM Fusion Access pour SAN, Fusion Data Foundation & Fusion Backup and Restore labs

Guide du laboratoire

<img src="./media/image1.jpeg" style="width:7.00903in;height:3.64375in"
alt="Une personne debout devant un écran d&#39;ordinateur Description générée automatiquement" />

# Table des matières
- [IBM Fusion Access pour SAN, Fusion Data Foundation \& Fusion Backup and Restore labs](#ibm-fusion-access-pour-san-fusion-data-foundation--fusion-backup-and-restore-labs)
- [Table des matières](#table-des-matières)
- [Aperçu d'IBM Fusion Access for SAN](#aperçu-dibm-fusion-access-for-san)
  - [Avantages clés d'IBM Fusion Access pour SAN](#avantages-clés-dibm-fusion-access-pour-san)
  - [Prérequis et limitations d'IBM Fusion Access pour SAN](#prérequis-et-limitations-dibm-fusion-access-pour-san)
- [Aperçu du laboratoire](#aperçu-du-laboratoire)
- [Infrastructure de laboratoire](#infrastructure-de-laboratoire)
- [Connection à l'environnement du laboratoire](#connection-à-lenvironnement-du-laboratoire)
  - [OpenShift Web Console](#openshift-web-console)
  - [Accès en ligne de commande (optionnel)](#accès-en-ligne-de-commande-optionnel)
- [Déploiement de Fusion Access for SAN](#déploiement-de-fusion-access-for-san)
- [Déploiement Fusion Data Foundation](#déploiement-fusion-data-foundation)
- [Déploiement de l’application message-board](#déploiement-de-lapplication-message-board)
- [Utilisation de l’application message-board](#utilisation-de-lapplication-message-board)
- [Déploiement de Fusion Backup \& Restore](#déploiement-de-fusion-backup--restore)
- [Configuration de Fusion Backup \& Restore](#configuration-de-fusion-backup--restore)
  - [Création d'un bucket d'objets S3](#création-dun-bucket-dobjets-s3)
  - [Creation de la repository de backup](#creation-de-la-repository-de-backup)
  - [Création d'un politique de sauvegarde](#création-dun-politique-de-sauvegarde)
  - [Assignation de la politique de sauvegarde à l'application Message-board](#assignation-de-la-politique-de-sauvegarde-à-lapplication-message-board)
  - [Création de la recette de sauvegarde pour l'application Message-board](#création-de-la-recette-de-sauvegarde-pour-lapplication-message-board)
  - [Association de la recette de sauvegarde à la politique de sauvegarde](#association-de-la-recette-de-sauvegarde-à-la-politique-de-sauvegarde)
- [Backup de l'application message-board](#backup-de-lapplication-message-board)
- [Suppression de l'application message-board](#suppression-de-lapplication-message-board)
- [Restauration de l'application message-board](#restauration-de-lapplication-message-board)
- [Annexe A – Troubleshooting déploiement Backup \& Restore](#annexe-a--troubleshooting-déploiement-backup--restore)
- [Annexe B – Retrouver les éléments de connexion au bucket créé pour stocker les backups](#annexe-b--retrouver-les-éléments-de-connexion-au-bucket-créé-pour-stocker-les-backups)


# Aperçu d'IBM Fusion Access for SAN

IBM Fusion Access pour SAN est une solution qui fournit un système de
fichiers en cluster évolutif pour le stockage d'entreprise,
principalement conçu pour offrir un accès au stockage de données
consolidé au niveau des blocs. Il présente les dispositifs de stockage,
tels que des baies de disques, au système d'exploitation comme s'il
s'agissait d'un stockage directement attaché.

Cette solution permet de supporter le stockage pour la virtualisation
Red Hat OpenShift et/ou de conserver une infrastructure SAN existante
pour un passage à la containérisation.

## Avantages clés d'IBM Fusion Access pour SAN

**Expérience utilisateur facilité**

IBM Fusion Access pour SAN propose une interface utilisateur intégrée à
la console OpenShift pour installer et configurer des clusters de
stockage, des systèmes de fichiers et des classes de stockage, afin de
simplifier le processus d'installation.

**Utilisation des infrastructures existantes**

Vous pouvez utiliser leurs investissements SAN existants, y compris les
technologies Fibre Channel (FC) et iSCSI, lors de leur transition vers
la virtualisation OpenShift® ou de leur expansion .

**Évolutivité**

Le cluster de stockage est conçu pour évoluer avec les clusters
OpenShift Container Platform et les charges de travail de machines
virtuelles. Il peut supporter jusqu'à environ 3000 VM sur 6 hôtes
bare-metal, avec des possibilités d'évolution supplémentaire en ajoutant
plus de systèmes de fichiers ou en utilisant des paramètres spécifiques
de classe de stockage.

**Stockage consolidé et partagé**

Les SAN permettent à plusieurs serveurs d'accéder à une grande capacité
de stockage de données partagée. Cette architecture facilite la
sauvegarde automatique des données et surveille en continu les processus
de stockage et de sauvegarde.

**Transfert de données à grande vitesse**

En utilisant un réseau dédié à haute vitesse pour le stockage, IBM
Fusion Access for SAN surmonte les goulots d'étranglement liés au
transfert de données qui peuvent survenir sur un LAN traditionnel, en
particulier pour de grands volumes de données.

**Accès au niveau des fichiers**

Bien qu'un SAN fonctionne principalement au niveau du bloc, les systèmes
de fichiers construits sur le stockage SAN peuvent fournir un accès au
niveau des fichiers via des systèmes de fichiers à disque partagé.

**Gestion centralisée**

Le logiciel SAN sous-jacent gère les serveurs, les dispositifs de
stockage et le réseau afin de garantir que les données circulent
directement entre les dispositifs de stockage avec une intervention
minimale des serveurs. Il supporte également la gestion centralisée et
la configuration des composants SAN tels que les numéros d'unité logique
(LUN).

## Prérequis et limitations d'IBM Fusion Access pour SAN

Prérequis pour installer et configurer IBM Fusion Access pour SAN :

- Red Hat OpenShift 4.19 ou plus

- Worker nodes baremetal avec un attachement SAN

- Un registre de conteneurs fonctionnel activé

- Tous les worker nodes doivent se connecter aux mêmes LUN. Un LUN
  partagé est un disque partagé auquel tous les workers accèdent
  simultanément.

- Un « secret pull » Kubernetes pour la bibliothèque de conteneurs IBM
  (cp.icr.io)

Puisque IBM Fusion Access pour SAN est basé sur IBM Storage Scale
Container Native Storage Access (CNSA), les limitations applicables à
l'IBM Storage Scale CNSA sont :

- Le déploiement de pods IBM Storage Scale sur les nœuds maîtres n'est
  pas pris en charge, sauf pour les clusters Red Hat OpenShift «
  compacts ».

- Le déploiement de pods IBM Storage Scale sur les nœuds équipés de CPU
  ARM n'est pas pris en charge.

- Les clusters natifs, IBM Storage Scale à nœud unique, ne sont pas pris
  en charge.

- Le nombre maximal de worker nodes pris en charge est de 128

- Le nombre maximal de systèmes de fichiers locaux pris en charge est de 7

- Le nombre maximal de nœuds de stockage est de 32

- Le nombre maximal de disques par système de fichiers local est de 128

# Aperçu du laboratoire

Ce guide présentera le déploiement d'**IBM Fusion Access for SAN**, le
déploiement de **IBM Storage Fusion**, qui nous permettra de disposer
d’un stockage objet via **Fusion Data Foundation** pour les backups
réalisés avec **Fusion Backup & restore**.

Les participants acquerront une expérience pratique avec :

- Déploiement et configuration d'IBM Fusion Access for SAN

- Déploiement d'IBM Fusion Data Foundation en mode interne

- Déploiement d'une application containerisée (message-board) utilisant
  la classe de stockage Fusion Access for SAN

- Déploiement et utilisation d’IBM Fusion Backup & restore

# Infrastructure de laboratoire


La plateforme de ce laboratoire est un cluster Red Hat OpenShift
Container Platform basé sur VMware, dans lequel vous installerez et
utiliserez Fusion. Voir le schéma ci-dessous.

<img src="./media/image2.png"
style="width:7.26389in;height:4.21944in" />

L'environnement de laboratoire est composé de 12 machines virtuelles
(VM), hébergées sur un cluster VMware vSphere. Toutes ces VM forment un
cluster Red Hat OpenShift Container Platform (OCP), exécutant
Kubernetes. Ce cluster est un cluster mixte avec différents types de
nœuds ouvriers. Il est composé de :

- 3 nœuds master, fonctionnant sous Red Hat CoreOS

- 3 nœuds d'infrastructure

- 3 nœuds de stockage pour Fusion Data Foundation (avec stockage local)

- 4 nœuds Worker (nous utiliserons ces nœuds pour le cluster Fusion
  Access for SAN)

Une seule VM, le nœud IBM Storage Scale, fournit le système de fichiers
distant Storage Scale (pour les fichiers) et une autre VM, le nœud IBM
Storage Ceph, assure le stockage objet. L'installation par défaut
n'inclut pas l'opérateur IBM Fusion.

Le tableau ci-dessous présente un résumé des informations détaillées sur
l'environnement du laboratoire que vous utiliserez.

<table style="width:93%;">
<colgroup>
<col style="width: 15%" />
<col style="width: 31%" />
<col style="width: 19%" />
<col style="width: 26%" />
</colgroup>
<thead>
<tr>
<th></th>
<th>Adresse IP</th>
<th>Utilisateur</th>
<th>Mot de passe</th>
</tr>
</thead>
<tbody>
<tr>
<td>Cluster URL</td>
<td>https://console-openshift-console.apps.fusion.Storage.lan</td>
<td>kubeadmin</td>
<td>Fourni par votre instructeur</td>
</tr>
<tr>
<td>IBM Storage Scale</td>
<td>192.168.252.5</td>
<td>root</td>
<td>Passw0rd !</td>
</tr>
<tr>
<td>IBM Storage Scale GUI</td>
<td><a href="https://192.168.252.5">https://192.168.252.5</a></td>
<td>admin</td>
<td>Passw0rd !</td>
</tr>
<tr>
<td>IBM Storage Scale GUI</td>
<td></td>
<td>csi-storage-gui-user</td>
<td>csi-storage-gui-password</td>
</tr>
<tr>
<td>IBM Ceph</td>
<td>192.168.252.7</td>
<td>admin</td>
<td>Passw0rd !</td>
</tr>
<tr>
<td>IBM Ceph GUI</td>
<td><a
href="https://192.168.252.7:8443">https://192.168.252.7:8443</a></td>
<td>admin</td>
<td>Ceph</td>
</tr>
</tbody>
</table>

# Connection à l'environnement du laboratoire

Pour accéder à l'infrastructure IBM Fusion fournie dans votre
environnement, vous devrez obtenir les identifiants de la console auprès
de votre instructeur. L'URL du bureau fournie est utilisée pour un accès
direct à la console OpenShift.

URL du bureau :
https://console-openshift-console.apps.ocp-xxxxxxxxx-xxxx.cloud.techzone.ibm.com

Nom d’utilisateur Mot de passe

kubeadmin xxxx-xxxx-xxxx-xxxx

## OpenShift Web Console

1.  Cela vous donne un accès direct à la console OpenShift via votre navigateur. Lorsque votre navigateur se connecte à l'interface utilisateur web (UI) de la plateforme de conteneurs OpenShift, sélectionnez **kube :admin** pour afficher l'écran de connexion.

    <img src="./media/image3.png" style="width:7in;height:3.56458in" />
2.  Saisissez **le nom d'utilisateur** (kubeadmin) et **le mot de passe** tels que fournis par votre instructeur.  
    <img src="./media/image4.png" style="width:7in;height:4.05in" />

3.  Une fois connecté, la page d'accueil d'OpenShift vous sera présentée.
    <img src="./media/image5.png" style="width:7in;height:3.60764in" />

## Accès en ligne de commande (optionnel)
 
1.  Pour accéder en ligne de commande, trouvez la **connexion SSH
    Bastion** et le **mot de passe Bastion** fournis par votre
    instructeur.

2.  Pour vous connecter via SSH, ouvrez un terminal sur un Mac ou un
    système Linux (Putty ou Powershell si vous êtes sous Windows) et
    saisissez une commande qui ressemble à ceci :

    ```sh
    ssh itzuser@apps.ocp-50t6fjgae-droi.cloud.techzone.ibm.com -P 40222
    ```

    (Utilisez le mot de passe du Bastion. De plus, le nom d'hôte n'est qu'un
    exemple. Utilisez les informations fournies par votre professeur).

3.  Maintenant que vous êtes connecté à votre nœud Bastion, vous pouvez accéder au cluster et exécuter quelques commandes OpenShift depuis la ligne de commande 'oc' :

    ```sh
    oc get clusterversion
    ```

    Si vous obtenez l'erreur suivante, vous devez alors obtenir les informations de connexion en ligne de commande depuis la console web OpenShift.

    ```sh
    [itzuser@69d783693027bf2af13ed1bc-bastion ~]$ oc get nodes

    Impossible de se connecter au serveur : tls : failed to verify
    certificate : x509 : certificate signé par une autorité inconnue

    [itzuser@69d783693027bf2af13ed1bc-bastion ~]$
    ```

    Vous pouvez obtenir les identifiants depuis la console web OpenShift.

    <img src="./media/image6.png" />

    <img src="./media/image7.png" />

# Déploiement de Fusion Access for SAN

Le stockage SAN étant bien connecté aux Workers, nous pouvons déployer IBM Fusion Access for SAN.

Sélectionnez **Ecosystem / Software Catalog** et chercher IBM Fusion Access for SAN.

<img src="./media/image8.png"  />

Sélectionnez l'opérateur Fusion Access for SAN. Conservez les options par défaut et cliquez sur **Install**.

<img src="./media/image9.png" />

Notez qu'il nous faut au minimum 3 nœuds Worker et chacun avec au moins 20 Go de mémoire. 

Sur la page suivante, passez **Update approval** sur **Manual**. Note : ne changez pas le namespace par défaut !

<img src="./media/image10.png"
style="width:7.26389in;height:3.73264in" />

Cliquez sur **Install** en bas de la page. Vous devrez ensuite approuver l'installation (car l'update approval est sur manuel).

<img src="./media/image11.png"
style="width:7.26389in;height:4.45694in" />

Attendez que l'opérateur termine l'installation, cela peut prendre quelques minutes.

<img src="./media/image12.png"
style="width:7.26389in;height:3.73681in" />

Avant de créer notre cluster IBM Fusion Access for SAN, nous devons créer un "pull secret" vers la bibliothèque de conteneurs IBM
(cp.icr.io).

Vous pouvez utiliser la clé disponible dans ce document (en cliquant droit pour ouvrir dans une autre tab) :
<https://ibm.box.com/v/clef-acces-repo-ibm>

Dans la console OpenShift, sélectionnez le menu **Workloads (1) > Secrets
(2)**, sélectionnez le projet **ibm-fusion-access (3)**, puis **Create** > **Key/value secret (4-5)**

<img src="./media/image13.png"
style="width:7.26389in;height:4.45694in" />

Dans **Secret name**, entrez **fusion-pullsecret**, dans **Key**, entrez **ibm-entitlement-key** puis **copiez/collez** la clef fournit dans le
lien ci-dessus dans le champ **value** réservé à cet effet, cliquez sur **Create**

<img src="./media/image14.png"
style="width:7.26389in;height:4.45694in" />

L'étape suivante consiste à créer une instance Fusion Access for SAN.
Dans la barre de menue latérale, sélectionnez **Ecosystem** > **Installed Operators** et **cliquez sur Fusion Access for SAN**.

<img src="./media/image15.png"
style="width:7.26389in;height:3.69514in" />

Assurez-vous que le projet sélectionné est bien **ibm-fusion-access**,
puis cliquez sur **Create instance**.

<img src="./media/image16.png"
style="width:7.26389in;height:4.45694in" />

Vous pouvez accepter les défauts. Notez que la boîte déroulante IBM
Storage Scale utilise par défaut la dernière version IBM CNSA, qui est
la 6.0.0.2.1. 

Cliquez sur **Create**.

<img src="./media/image17.png"
style="width:7.26389in;height:4.45694in" />

Après avoir cliqué sur **Create**, attendez que le statut affiche **Ready**. 

Nous pouvons maintenant créer un cluster de stockage Fusion Access for SAN.

<img src="./media/image18.png"
style="width:7.26389in;height:3.73264in" />

Dans le menu lateral, sélectionnez **Storage** > **Fusion Access for SAN**. 

Notez que vous devrez peut-être rafraîchir votre page web (F5) si vous ne voyez pas Fusion Access for SAN dans Stockage.

<img src="./media/image19.png"
style="width:7.26389in;height:3.73264in" />

Cliquez maintenant sur **Create storage cluster**. 

À la page suivante, une liste de nœuds OpenShift est affichée.  
Sélectionnez les **quatres** noeuds Worker auxquels les LUNs ont été attribuées, puis cliquez sur **Create
storage cluster**.  
Notez que l’interface indique que les 3 noeuds
sélectionnés partagent bien deux disques entre eux.

<img src="./media/image20.png"
style="width:7.26389in;height:3.75625in" />

La console OpenShift affiche ensuite une nouvelle page dans laquelle le bouton **Create file system claim** est grisé.  
IBM Fusion Access for SAN est occupé à déployer IBM Storage Scale CNSA. Vous devrez attendre que
ce processus soit terminé (environ 10mn).

<img src="./media/image21.png"
style="width:7.26389in;height:3.71111in" />

Pour suivre l’avancement du déploiement, dans le menu latéral, sélectionnez **Workloads** / **Pods** et changez le projet courant pour **ibm-spectrum-scale.**  
Lorsque le déploiement est terminé, la liste des
pods déployés doit être similaire à celle ci-dessous :

<img src="./media/image22.png"
style="width:7.26389in;height:4.20347in" />

Vous pouvez maintenant, dans le menu latéral, revenir à la page **Storage > Fusion Access for SAN**.   
Le bouton **Create file system claim** doit maintenant être bleu, vous pouvez cliquer pour créer le claim.

<img src="./media/image23.png"
style="width:7.26389in;height:3.75208in" />

À la page suivante, saisissez un nom pour le système de fichiers (fusion3) et sélectionnez les deux luns disponibles partagées par les 3 noeuds Worker.   
Puis cliquez sur **Create file system claim**.

<img src="./media/image24.png"
style="width:7.26389in;height:3.76597in" />

La création du file system claim démarre.

<img src="./media/image25.png"
style="width:7.26389in;height:3.76389in" />

Vous pouvez monitorer la progression en utilisant l’outil de recherche de la console OpenShift. 

Dans le menu latéral, sélectionnez **Home >
Search**, puis entrez **local** dans le champ **Resources**, sélectionnez ensuite **LocalDisk** pour afficher le status des deux disques.

<img src="./media/image26.png"
style="width:7.26389in;height:4.26389in" />

Dans la page **Storage > Fusion Access for SAN**, la console  devrait finalement afficher le système de fichiers en état **Ready**.

<img src="./media/image27.png"
style="width:7.26389in;height:4.26389in" />

**Optionnellement**, vous pouvez cliquer sur le lien **Dashboard** en haut à droite pour vous connecter au cluster IBM Storage Scale CNSA.

<img src="./media/image28.png"
style="width:7.26389in;height:3.96944in" />

Connectez-vous avec votre nom d'utilisateur kubeadmin et votre mot de passe.

<img src="./media/image29.png"
style="width:7.26389in;height:3.64306in" />

Après la connexion, vous devriez voir l'interface graphique par défaut d'IBM Storage Scale. Vous pouvez utiliser votre utilisateur kubeadmin et votre mot de passe pour cela.

<img src="./media/image30.png"
style="width:7.26389in;height:3.75208in" />

Vous pouvez accéder à la vue Tableau de bord en cliquant sur les quatre barres horizontales en haut à gauche de l'interface graphique et en naviguant dans **Monitoring / Dashboard**.

<img src="./media/image31.png"
style="width:7.26389in;height:3.65764in" />

Dans la console OpenShift, sur la page **Storage / Storage Classes**, une vérification rapide dmontrer qu'une nouvelle classe de stockage a été créée par l'opérateur Fusion Access for SAN.  
Le nom correspond à celui du filesystem. (fusion3 dans notre exemple)

<img src="./media/image32.png"
style="width:7.26389in;height:3.70486in" />

# Déploiement Fusion Data Foundation 

IBM Fusion Data Foundation mets a disposition différentes classes de stockage – Fichier, Object, Bloc.  
Dans ce lab, nous allons utiliser son stockage Objet pour le backup de l’application qui sera réalisé plus loin.

Nous allons commencer par déployer l'opérateur **IBM Fusion**.

Dans le menu latéral, sélectionnez **Ecosystem / Software Catalog**, puis recherchez **fusion**.

<img src="./media/image33.png"
style="width:7.26389in;height:3.75347in" />

Choisissez **IBM Storage Fusion** et installez l'opérateur.

<img src="./media/image34.png"
style="width:7.26389in;height:4.26389in" />

Sur la page d’installation de l’opérateur, acceptez les paramètres par défaut.  
Cliquez sur **Install** en bas de la page.

<img src="./media/image35.png"
style="width:7.26389in;height:4.26389in" />

Attendez que l'installation de l'opérateur soit terminée.  
Puis cliquez sur l'icône de bloc en haut à droite de la page pour lancer l'interface
graphique IBM Fusion.

<img src="./media/image36.png"
style="width:7.26389in;height:4.26389in" />

Si nécessaire, connectez-vous avec votre nom d'utilisateur kubeadmin et votre mot de passe.  
Puis accepter le contrat de licence.

<img src="./media/image37.png"
style="width:7.26389in;height:2.75972in" />

Sur le tableau de bord Fusion, cliquez sur **Services** dans la barre gauche.

<img src="./media/image38.png"
style="width:7.26389in;height:3.51597in" />

Cliquez sur **Data Foundation** pour lancer son installation.

Vous pouvez simplement accepter les paramètres par défaut et cliquer sur **Install**.

<img src="./media/image39.png"
style="width:7.26389in;height:3.74167in" />

Sur la page Install Service, choisissez une installation **Locale** pour utiliser les disques attachés aux noeuds de stockage, cliquez sur **Install** pour lancer l’installation.

<img src="./media/image40.png"
style="width:7.26389in;height:3.69028in" />

Attendez que le service s'installe.

<img src="./media/image41.png"
style="width:7.26389in;height:3.75903in" />

Une fois installé, cliquez sur **Get Started**.

<img src="./media/image42.png"
style="width:7.26389in;height:3.21181in" />

Renseignez ensuite la page de configuration comme ci-dessous et cliquez sur **Next**.

<img src="./media/image43.png"
style="width:7.26389in;height:4.52222in" />

Entrez **fusionvolset** dans les champs **LocalVolumeSet name** et **StorageClass name**

<img src="./media/image44.png"
style="width:7.26389in;height:3.67014in" />

Sélectionnez les 3 noeuds de stockage.

<img src="./media/image45.png"
style="width:7.26389in;height:4.26389in" />

Cliquez sur **Next** et confirmez la création de LocalVolumeSet.  
À la page suivante, vérifiez que la capacité est de 2.93 TiB puis cliquez sur **Next**.

<img src="./media/image46.png"
style="width:7.26389in;height:4.26389in" />

Sur l'écran de sécurité et réseau, nous acceptons simplement les
paramètres par défaut.  
Cliquez sur **Next**.

<img src="./media/image47.png"
style="width:7.26389in;height:3.7125in" />

Sur le dernier écran, cliquez sur **Create storage system**.

<img src="./media/image48.png"
style="width:7.26389in;height:4.26389in" />

La création va prendre un peu de temps (environ 10 minutes).

<img src="./media/image49.png"
style="width:7.26389in;height:3.73542in" />

Une fois terminée, vous verrez que le statut du cluster de stockage et de la résilience des données est vert.

<img src="./media/image50.png"
style="width:7.26389in;height:4.26389in" />

Et les classes de stockage FDF par défaut sont créées automatiquement.

<img src="./media/image51.png"
style="width:7.26389in;height:3.65556in" />

Nous disposons maintenant à la fois des classes de stockage de FDF ainsi que d’un système de fichier Access for SAN sur lequel nous allons déployer une application.  
Elle sera ensuite sauvegardée via une "recipe" **Fusion Backup and Restore**.

# Déploiement de l’application message-board

Cette application s’appuie sur une base de données Postgresql et un frontal web. Les messages qui sont saisies sont stockés dans la base de données, ce qui nous permettra de montrer que le processus de backup/restauration permet bien leur récupération.

Les composants de l’application sont automatiquement déployés dans le projet **message-board**.

Depuis **Home / Overview** par exemple, **cliquez sur le + en haut à droite** de la fenêtre pour pouvoir importer le manifeste yaml qui permettra de réaliser l’installation.

Le fichier yaml peut être accédé à cette adresse [**all-in-one.yaml**](./message-board/all-in-one.yaml).  
Copiez-collez son contenu dans la fenêtre Import YAML et cliquez sur **Create**

<img src="./media/image52.png"
style="width:7.26389in;height:4.15556in" />

Le déployement devrait être assez rapide, sélectionné **Networking / Routes**, puis le projet message-board et cliquez sur le bouton d’ouverture de l’url.  
<img src="./media/image53.png"
style="width:0.23611in;height:0.26389in" />

<img src="./media/image54.png"
style="width:7.26389in;height:4.15556in" />

# Utilisation de l’application message-board

Depuis l’application, créez quelques messages qui seront stockés dans la base de données, sauvegardés et restaurés grâce à **Fusion Backup & Restore**
<img src="./media/image55.png"
style="width:7.26389in;height:4.15556in" />

La base de données contient maintenant quelques éléments et peut être
sauvegardée.  

Pour cela nous devons installer Fusion Backup & Restore.

# Déploiement de Fusion Backup & Restore

**IBM Fusion Backup & Restore** offre une solution robuste et native pour la protection des applications containérisées, en prenant en charge non seulement les données mais aussi l’état complet des applications. 

Son principal atout réside dans l’utilisation de “recettes” (**recipes**), qui permettent de capturer de manière cohérente les composants complexes d’une application (déploiement, volumes, configurations, ...) et d’automatiser des sauvegardes et restaurations applicatives complètes.  
Cette approche simplifie considérablement la gestion des workloads Kubernetes avancés. 

Par ailleurs, le stockage des sauvegardes s’appuie sur des buckets S3, garantissant une grande flexibilité, une portabilité multi-cloud et une meilleure résilience des données, tout en s’intégrant facilement aux infrastructures existantes.

Le **déploiement** de Fusion Backup & Restore est réalisé **depuis l’interface Fusion**.  
Cliquez sur l'icône de bloc en haut à droite de la page pour lancer l'interface graphique **IBM Fusion**.

<img src="./media/image36.png"
style="width:7.26389in;height:4.26389in" />

Cliquez sur **Backup & Restore**, puis cliquez sur **Install**

<img src="./media/image56.png"
style="width:7.26389in;height:4.26389in" />

Sélectionnez la classe de stockage qui sera utilisée par le service : **ocs-storagecluster-ceph-rbd** puis cliquez sur **Install**

<img src="./media/image57.png"
style="width:7.26389in;height:4.26389in" />

> **Si l’installation reste bloquée à 95%, il est nécessaire d’appliquer la solution corrective décrite dans l’[annexe A](#annexe-a--troubleshooting-déploiement-backup--restore) à la fin du présent document**.

# Configuration de Fusion Backup & Restore

## Création d'un bucket d'objets S3

La première étape consiste à créer un bucket destiné à recevoir les sauvegardes.

Pour les besoins de ce lab, nous allons utiliser le stockage objet fournit par Fusion. Idéalement, ce stockage objet devrait bien sûr être créé à l’extérieur du cluster OpenShift.

Sélectionner **Storage / Object storage**, vérifiez que le **projet** sélectionné est bien **ibm-backup-restore**, puis, sélectionnez **Object Bucket Claims**, puis cliquez sur **Create ObjectBucketClaim**.

<img src="./media/image58.png"
style="width:7.26389in;height:3.79028in" />

Renseignez le nom du bucket à créer dans le champ **ObjectBucketClaim Name**, ici **backup-repository**, conservez les autres options par défaut puis cliquez sur **Create**

<img src="./media/image59.png"
style="width:7.26389in;height:3.79028in" />

Une fois le bucket créé, faites défiler la page, **cliquez sur Reveal Values** pour voir les informations de connexion au bucket.
Copiez ces informations, elles seront nécessaire pour déclarer la **backup location** dans **IBM Fusion**

<img src="./media/image60.png"
style="width:7.26389in;height:3.79028in" />

Sélectionnez ensuite **Networking / Routes**.  
Dans la liste déroulenate des project, activez **Show default projects** et sélectionnez le projet **openshift-storage**.  
Recherchez l’entrée **s3** puis cliquez sur l’icône de copie <img src="./media/image61.png" style="width:0.22222in;height:0.25in" /> de l’url du stockage.  
Cette URL sera également nécessaire pour déclarer la **backup location** dans **IBM Fusion**


<img src="./media/image62.png"
style="width:7.26389in;height:3.79028in" />

## Creation de la repository de backup
Nous pouvons maintenant configurer Fusion Backup & Restore.    
**Dans l’interface Fusion**, sélectionnez **Backup & restore > Locations** puis cliquez sur **Add location**

<img src="./media/image63.png"
style="width:7.26389in;height:2.55069in" />

**Attribuez un nom** à la location, ici **backup-repository** par exemple et sélectionnez S3 Compliant

<img src="./media/image64.png"
style="width:7.26389in;height:3.03056in" />

Puis complétez les informations de connexion au bucket: 
- le **endpoint** correspond à l’URL que vous venez de copier
- les champs **Bucket**, **Access Key, Secret Key** sont complétez avec les éléments relevés lors de la création du bucket
Enfin cliquez sur **Add**

Si vous n’avez pas noté ces éléments lors de la création du bucket, vous
pouvez vous référer à l’[annexe B](#annexe-b--retrouver-les-éléments-de-connexion-au-bucket-créé-pour-stocker-les-backups) à la fin de ce document pour retrouver ces informations.

<img src="./media/image65.png"
style="width:7.26389in;height:3.79028in" />

## Création d'un politique de sauvegarde
L’étape suivante consiste à créer une policy. Sélectionnez **Backup & restore > Policies** puis **Add policy**

- Attribuez un nom
- Sélectionnez **Object storage**
- Sélectionnez le repository créé précédemment
- cliquez sur **Create Policy**

<img src="./media/image66.png"
style="width:4.54114in;height:8.48501in" />

## Assignation de la politique de sauvegarde à l'application Message-board
Cette politique de sauvegarde peut maintenant être assignée à l’application Message-board.

Dans **Applications**, sélectionnez l’application **message-board**, puis cliquez sur **Assign backup policy** puis cliquez sur **Save**

<img src="./media/image67.png"
style="width:7.26389in;height:4.15556in" />

## Création de la recette de sauvegarde pour l'application Message-board
La dernière étape avant de pouvoir effectuer une sauvegarde consiste à créer la recette et à l’associer à la policy. 

Depuis la console OpenShift, cliquez sur le +, puis copiez/collez le contenu du fichier [recipe.yaml](./message-board/recipe.yaml) dans la zone de saisie et cliquez sur **Create**.

<img src="./media/image68.png"
style="width:7.26389in;height:4.15556in" />

## Association de la recette de sauvegarde à la politique de sauvegarde
Dans la console OpenShift, cliquez sur le **projet ibm-spectrum-fusion-ns**, puis dans la zone de recherche entrez **policya** cela devrait suffire pour retrouver la ressource
La recette étant créée dans le projet message-board, il faut maintenant l’associer à la policy. 

Pour la retrouver facilement dans la console OpenShift, cliquez sur **Home / Search**, veillez à sélectionner le **projet ibm-spectrum-fusion-ns**, puis dans la zone de recherche entrez **policya** cela devrait suffire pour retrouver la ressource
**PolicyAssignment**.

<img src="./media/image69.png"
style="width:7.26389in;height:4.15556in" />

> Note: La Custom Resource PolicyAssignement permet d'associer une politique de sauvegarde à une application. Pour chaque politique de sauvegarde assignée à l'application, une objet PolicyAssignment est créé.

Cliquez sur la PolicyAssignment pour l’éditer.  
Recherchez la section **spec** (Ctrl+F pour afficher la boite de recherche), puis, si nécessaire, **ajoutez la section recipe** comme dans la capture ci-dessous et cliquez sur **Save**
```yaml
...
  recipe :
    name : message-board-backup-recipe
    namespace : message-board
    apiVersion : spp-data-protection.isf.ibm.com/v1alpha1
...
```
<img src="./media/image70.png"
style="width:7.26389in;height:4.15556in" />

La configuration de la policy de backup est terminée.  

# Backup de l'application message-board
Nous pouvons maintenant retourner sur le portail Fusion pour lancer une sauvegarde de
l’application. 
Sélectionnez **Application**, cliquez sur les **trois points** en regard de l’application message-board, lancez le backup et confirmez.

<img src="./media/image71.png"
style="width:7.26389in;height:3.94514in" />

Vous pouvez suivre le déroulement du backup en sélectionnant **Backup & restore > Jobs**.  
La section Backup séquence affiche les étapes du backup définie dans la recipe.

<img src="./media/image72.png"
style="width:6.74867in;height:6.96609in" />

Une fois le backup terminé, nous pouvons procéder à une restauration, soit dans un nouveau projet pour par exemple tester une nouvelle version de code, soit dans le projet d’origine suite par exemple à l’effacement accidentel du projet ou de ses ressources.

# Suppression de l'application message-board
Pour le cas présent, nous allons nous placer dans la seconde situation en **supprimant le projet message-board** pour pouvoir le restaurer.

Depuis la console OpenShift, sélectionnez **Home / Projects** et recherchez le projet message-board.  
**cliquez sur les trois points et cliquez sur Delete Project**.  
Entrez le nom du projet pour confirmer sa suppression et cliquez sur **Delete**

<img src="./media/image73.png"
style="width:7.26389in;height:3.79028in" />

# Restauration de l'application message-board
Pour la restauration d'une application existante, nous allons utiliser le même projet.  
Une fois le projet supprimé, **revenez sur la console Fusion** dans le menu **Applications**.  
L’application message-board est toujours présente, **cliquez sur les trois points** puis sélectionnez **Restore.**  
Sélectionnez l’option **Use same project**, cliquez sur **Next**

<img src="./media/image74.png" style="width:7.26389in;height:3.79028in" />

**Sélectionnez le backup existant** puis cliquez sur **Next** et confirmez la restauration.

<img src="./media/image75.png"
style="width:7.26389in;height:3.79028in" />

Ici encore, vous pouvez suivre le déroulement de la restauration depuis le menu **Backup & restore > Jobs**.

Lorsque la restauration est terminée, retournez dans la console OpenShift, sélectionnez le menu **Networking / Routes**, sélectionnez leprojet message-board et cliquez sur le bouton d’accès à l’url.
<img src="./media/image76.png"
style="width:0.19444in;height:0.19444in" />

<img src="./media/image77.png"
style="width:7.26389in;height:3.79028in" />

L'application et les différents messages que vous aviez postés sont restaurés.

Et nous avons fini avec les laboratoires !


# Annexe A – Troubleshooting déploiement Backup & Restore

Du fait des ressources limitées de la plateforme utilisée pour ce laboratoire, le pod **backup-location-deployment** peut avoir des difficultés à démarrer du fait de durées de timeouts trop courtes. 

Il est possible de changer ces valeurs en éditant le yaml du pod.

Dans le menu latéral de la console OpenShift, sélectionnez **Workloads > Pods**,  sélectionnez le projet **ibm-backup-restore** puis recherchez et vérifiez si le pod **backup-location-deployement-xxxxx** ne démarre pas.

<img src="./media/image79.png"
style="width:7.26389in;height:4.27361in" />

Si tel est bien le cas, sélectionnez **Workloads > Deployments** et recherchez et cliquez sur **backup-location-deployment**

<img src="./media/image80.png"
style="width:7.26389in;height:4.27361in" />

Éditez le YAML pour appliquer les nouvelles valeurs comme ci-dessous

<img src="./media/image81.png"
style="width:7.26389in;height:4.27361in" />

Cherchez et modifiez les valeurs ci-dessous de la façon suivantes :
```yaml
readinessProbe:
  httpGet:
    path: /health/ready
    port: 9443
    scheme: HTTPS
    initialDelaySeconds: 120
    timeoutSeconds: 30
    periodSeconds: 10
    successThreshold: 1
    failureThreshold: 30
...
livenessProbe:
  httpGet:
    path: /health/live
    port: 9443
    scheme: HTTPS
    initialDelaySeconds: 180
    timeoutSeconds: 5
    periodSeconds: 10
    successThreshold: 1
    failureThreshold: 10
...
startupProbe:
  httpGet:
    path: /health/started
    port: 9443
    scheme: HTTPS
    initialDelaySeconds: 180
    timeoutSeconds: 10
    periodSeconds: 10
    successThreshold: 1
    failureThreshold: 30
```
Une fois ces nouvelles valeurs appliquées, cliquez sur **Save**

# Annexe B – Retrouver les éléments de connexion au bucket créé pour stocker les backups

Sélectionnez **Storage > Object storage**, vérifiez que le projet sélectionné est **All Projects** ou **ibm-backup-restore**. 

Sélectionnez l’onglet **Object Bucket Claims**, puis cliquez sur backup repository

<img src="./media/image82.png"
style="width:7.26389in;height:4.17153in" />

Faites défiler la fenêtre puis cliquez sur **Reveal Values**

<img src="./media/image83.png"
style="width:7.26389in;height:4.17153in" />
