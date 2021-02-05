<- [Retour](02-Prep-to-Install-Puppet-Master.md#lab-2)

---

### **Lab #3** - Installer Puppet

---

### Aperçu

Temps pour terminer: 45 minutes

Dans ce lab, nous installerons Puppet 2016.5.1


### Connectez-vous à votre VM Puppet

Nous sommes sur le point d'installer Puppet alors assurez vous que vous êtes
connecté à l'hôte Puppet Master.

### Remarque sur l'utilisateur root

Lors de la connexion à une VM provisionnée par Vagrant avec **vagrant ssh**, vous serez
connecté en tant qu'utilisateur **vagrant** (un utilisateur normal / non privilégié). Toutefois, nous avons besoin de privilèges root lors de l'installation de Puppet.

Afin de simplifier les commandes que nous exécutons dans les sections suivantes, je suppose
vous savez comment utiliser sudo et quand l'utiliser. Toutes les commandes de Puppet doivent être exécutées en tant que root, la chose la plus facile à faire serait donc de devenir root. Nous sommes en formation après tout, et si vous gâchez quelque chose, vous pouvez simplement supprimer et recommencer.

Donc, si vous êtes déjà root, super!

Si vous n'êtes pas encore l'utilisateur root, devenez root et soyez heureux!

```shell
     sudo su -
```

### Exécutez le programme d'installation

Accédez au répertoire avec le logiciel PE:

```shell
rpm -ivh https://yum.puppetlabs.com/el/7/products/x86_64/puppetlabs-release-7-11.noarch.rpm
```
Exécutez ensuite le programme d'installation:

```shell
yum install -y puppetserver
```
Modifiez la configuration de 'puppetserver' à l'aide de vi.
```
vi /etc/sysconfig/puppetserver
```
Modifiez maintenant la ligne comme ci-dessous.
```
JAVA_ARGS="-Xms1g -Xmx1g ...."
```
Sauvegarder et quitter.
Ensuite, allez dans le répertoire de configuration de puppet et éditez le fichier 'puppet.conf'.

```
cd /etc/puppetlabs/puppet
vim puppet.conf
```
Ajoutez la configuration suivante.
```
[master]
dns_alt_names=master.hakase.io,puppet

[main]
certname = master.hakase.io
server = master.hakase.io
environment = production
runinterval = 1h
```
Sauvegarder et quitter.

Maintenant, démarrez le puppetserver et activez-le à chaque fois au démarrage.
```
systemctl start puppetserver
systemctl enable puppetserver
```

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
