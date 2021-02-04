<- [Retour](02-Prep-to-Install-Puppet-Master.md#lab-2)

---

### **Lab #3** - Installer Puppet Enterprise

---

### Aperçu

Temps pour terminer: 45 minutes

Dans ce lab, nous installerons Puppet Enterprise 2016.5.1

* PE est gratuit à installer et à évaluer
* Lorsque vous exécutez PE sans licence, vous êtes limité à 10 agents


### Connectez-vous à votre VM Puppet

Nous sommes sur le point d'installer Puppet Enterprise alors assurez vous que vous êtes
connecté à l'hôte Puppet Master.

### Remarque sur l'utilisateur root

Lors de la connexion à une VM provisionnée par Vagrant avec **vagrant ssh**, vous serez
connecté en tant qu'utilisateur **vagrant** (un utilisateur normal / non privilégié). Toutefois, nous
avons besoin de privilèges root lors de l'installation de Puppet Enterprise.

Afin de simplifier les commandes que nous exécutons dans les sections suivantes, je suppose
vous savez comment utiliser sudo et quand l'utiliser. Toutes les commandes de Puppet doivent être exécutées en tant que root, la chose la plus facile à faire serait donc de devenir root. Nous sommes en formation après tout, et si vous gâchez quelque chose, vous pouvez simplement le supprimer et
recommencer.

Donc, si vous êtes déjà root, super!

Si vous n'êtes pas encore l'utilisateur root, devenez root et soyez heureux!

```shell
     sudo su -
```

### Exécutez le programme d'installation

Accédez au répertoire avec le logiciel PE:

```shell
cd /share/software/puppet
tar xzvf puppet-enterprise-2016.5.1-el-7-x86_64.tar.gz
cd puppet-enterprise-2016.5.1-el-7-x86_64
```
Exécutez ensuite le programme d'installation:

```shell
     ./puppet-enterprise-installer
```

Le programme d'installation vous demandera:


```shell
=============================================================
    Puppet Enterprise Installer
=============================================================
Puppet Enterprise offers two different methods of installation.

 [1] Guided install

Recommended for beginners. This method will install and configure a temporary
webserver to walk you through the various configuration options.

NOTE: This method requires you to be able to access port 3000 on this machine
from your desktop web browser.

 [2] Text-mode

Recommended for advanced users. This method will open your $EDITOR (vi)
with a PE config file (pe.conf) for you to edit before you proceed with installation.

The pe.conf file is a HOCON formatted file that declares parameters and values needed to
install and configure PE.
We recommend that you review it carefully before proceeding.

=============================================================


 How to proceed? [1]:
```

Acceptez la valeur par défaut «[1]» (appuyez simplement sur Entrer).

Certains packages seront installés, puis le programme d'installation vous demandera de ouvrir une page dans votre navigateur Web ...

```shell
## We're preparing the Web Installer...

2016-11-14 17:39:20,482 Running command: mkdir -p /opt/puppetlabs/puppet/share/installer/installer
2016-11-14 17:39:20,487 Running command: cp -pR /share/software/puppet/puppet-enterprise-2016.5.1-el-7-x86_64/* /opt/puppetlabs/puppet/share/installer/installer/

## Go to https://puppet.example.com:3000 in your browser to continue installation.

## Be sure to use 'https://' and that port 3000 is reachable through the firewall.
```

N'oubliez pas que nous avons **transféré le port 3000 à 22000** sur notre poste de travail, donc ...

* Use web browser to connect to: **<https://127.0.0.1:22000/>**
* Click **Let's Get Started**
* Click **Monolithic Install**
* For FQDN, enter **puppet.example.com**
* For DNS alias, enter **puppet**
* Select: Install PostgreSQL on the PuppetDB host for me.
* Enter an initial password for the admin user (we will change it later)
* Click **Submit**
* Click **Continue**

Remarque: ne vous inquiétez pas des avertissements relatifs à la mémoire et à l'espace disque. La taille de mémoire et d'espace disque que nous avons provisionné conviendra parfaitement pour cette formation exercice. Évidemment, lors d'un déploiement pour un environnement de production réel, vous
voulez faire attention à ceux-ci!

Vous verrez ceci:

```shell
     We're checking to make sure the installation will work correctly
     Verify local environment.
     Verify that 127.0.0.1 can resolve puppet.example.com.
     Verify root access on puppet.example.com.
     Verify that DNS is properly configured for puppet.example.com.
     Verify that your hardware meets requirements on puppet.example.com.
     [puppet.example.com] We found 2,847 MB RAM. We recommend at least 4096 MB.
     Verify that 127.0.0.1 has a PE installer that matches puppet.example.com's OS.
     Verify that '/opt', '/var', and '/tmp' contain enough free space on puppet.example.com.
     [puppet.example.com] Insufficient space in '/opt' (16 GB); we recommend at least 100 GB for a production environment.
```

Cliquez sur **Deploy Now**


```shell
     Intalling your deployment
     Install Puppet Enterprise on puppet.
     Verify that Puppet Enterprise is functioning on puppet.
     [puppet] The puppet agent ran successfully.
     Verify that MCollective is functioning on puppet.
     Backup installer log files to puppet.
```

Pendant le processus d'installation, vous pouvez cliquer sur 'Log view' pour voir ce qui se passe dans les coulisses, puis cliquez sur 'Summary View' pour revenir à l'aperçu.

![Installation PE terminée](images/PE-Install-Finished.png)

Remarque: une fois l'installation terminée, cliquer sur le bouton 'Start Using Puppet Enterprise' ne fonctionnera **pas**, car nous transférons le port à partir d'un VMs à notre hôte local. Utilisez plutôt le lien ci-dessous.

---

### Connectez-vous à la PE console

Nous avons transféré le port 443 de notre machine virtuelle Puppet au port 22443 de notre station de travail d'hébergement, vous devriez donc pouvoir vous connecter à la PE console via l'URL:

* PE Console URL:  **<https://127.0.0.1:22443/>**

![PE Console Login](images/PE-Console-Login.png)

Connectez-vous en tant que **admin** et entrez le mot de passe administrateur que vous avez choisi lors de l'installation.

Si vous avez oublié (ou pas sûr de ce que vous avez tapé), vous pouvez trouver le mot de passe dans le fichier de réponses:

```shell
     grep console_admin_password /opt/puppetlabs/puppet/share/installer/conf.d/puppet.example.com.conf
```

Probablement une bonne idée de **le changer** ...! (Vous pouvez modifier le mot de passe administrateur via l'interface Web de PE Console)

### Fichiers journaux

Tous les fichiers journaux de Puppet Enterprise et de ses composants se trouvent dans: `/var/log/puppetlabs/`

Si vous souhaitez consulter les journaux d'installation, ils sont disponibles dans `/var/log/puppetlabs/installer/`

---

C'est tout ce qu'il y a à installer comme **Monolithic** puppet server.

---

### Retour à la PE console ...

Jouons un peu dans la PE console. Pour changer le compte 'admin' mot de passe, cliquez sur «admin» dans le coin supérieur droit et sélectionnez «Mon compte».
Vous devriez trouver un lien 'Reset password' en haut/à droite de cette page.

Regardez autour de la PE console . Essayez de cliquer sur les onglets **Events**, **Nodes**,
et **Classification** pour voir ce que vous pouvez voir.

Vous devriez voir 1 agent enregistré appelé
**puppet.example.com**. Ceci est votre Puppet master!


### Testez votre installation Puppet ...

Testez votre installation PE en exécutant l'agent manuellement à partir de l'invite du shell comme ceci:

```shell
     puppet agent -t
```

Remarque: même sur le Puppet Master, l'agent s'exécute régulièrement en arrière-plan
en tant que service. Le master se configure via Puppet. Soyez conscient, si vous
faites un changement qui affecte tous les nœuds Puppet, vous affecterez la config du
master également (par exemple, une modification globale de /etc/hosts).

Ne désactivez **pas** l'agent de Puppet sur le Master.

``` shell
[root@puppet ~]# puppet agent -t
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Loading facts
Info: Caching catalog for puppet.example.com
Info: Applying configuration version '1479158750'
Notice: Applied catalog in 15.20 seconds
```

Nous avons confirmé que l'exécution de l'agent réussit, nous savons donc que le Puppet Master est opérationnel et capable pour construire un catalogue pour lui-même.

Si vous utilisez un terminal prenant en charge la couleur ANSI, vous remarquerez que le texte
est en **VERT**. S'il y a des erreurs de Puppet pendant une course de Puppet, ces erreurs
apparaîtrait dans le texte **ROUGE**.

![Puppet Agent Clean Run](images/Puppet-Agent-Clean-Run-puppet.png)

D'accord, passons à autre chose ...

---

Continuez vers **Lab #4** -> [Installez Puppet Agent sur le nœud de l'agent et testez l'exécution de la Puppet](04-Install-Puppet-Agent.md#lab-4)

---

<- [Retour au sommaire](/README.md)

---
