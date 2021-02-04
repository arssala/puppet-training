<- [Retour](01-Provision-Training-VMs.md#lab-1)

---

### **Lab #2** - Préparez l'installation de Puppet Enterprise sur la VM **puppet**

---

### Aperçu

Temps pour terminer: 10 minutes

Dans cet atelier, nous effectuerons certaines étapes de pré-installation pour préparer notre VM à prendre Puppet Enterprise

### Démarrez vos VM de formation

Si ce n'est déjà fait ...

```
vagrant up puppet
```


Si vous ne connaissez pas l'état de vos VM contrôlées par vagrant, vous pouvez toujours vérifier avec ...

```
$ vagrant global-status
id       name         provider   state    directory
--------------------------------------------------------------------------------
4dd5ed7  puppet       virtualbox running  /Users/Mark/Vagrant/puppet-tutorial-pe
070258c  gitlab       virtualbox poweroff /Users/Mark/Vagrant/puppet-tutorial-pe
682eafe  agent        virtualbox poweroff /Users/Mark/Vagrant/puppet-tutorial-pe
```

Notez que la VM **puppet** est en cours d'exécution, mais les VM **gitlab** et **agent** sont éteintes.

### Connectez-vous à votre machine virtuelle de formation


```
$ vagrant ssh puppet
```

Une fois que vous êtes connecté à la VM **puppet**, notez que le prompt du shell bash ressemble à ceci:


```
[vagrant@puppet ~]$
```

Devenez root ...

```
[vagrant@puppet ~]$ sudo su -
```

... et notez que l'invite du shell change en:


```
Last login: Fri Oct 21 15:37:10 UTC 2020 on pts/0
[root@puppet ~]#
```

Maintenant, quittez votre shell root, et revenez au shell de l'utilisateur vagrant ...

```
[root@puppet ~]# exit
logout
[vagrant@puppet ~]$
```

Tout au long des travaux pratiques, vous devrez exécuter de nombreuses commandes (la plupart) en tant que root.

Utiliser sudo est une bonne habitude à développer, mais démarrer un shell root peut être plus
pratique tant que vous vous souvenez que vous pouvez gravement endommager un système en cours d'exécution si vous tapez la mauvaise commande au mauvais endroit. (par exemple, que faire si vous copiez accidentellement et collez du texte dans un shell  root, et il contient des commandes qui suppriment tout le répertoire /lib?)

### Étapes de pré-installation

Il y a quelques choses que nous devons faire pour que notre VM soit prête à prendre PE:

    - Ajoutez quelques entrées au fichier /etc/hosts (si ce n'est déjà fait)
    - Ouvrez certains ports via le pare-feu hôte

### Modifier /etc/hosts

Le fichier `/etc/hosts` devrait déjà être configuré correctement, mais si pour une raison quelconque
vous trouvez que ce n'est pas le cas, allez-y et modifiez-le comme suit.

Modifiez `/etc/hosts` et ajoutez des entrées pour localhost, ainsi que nos 3 machines virtuelles du lab, les deux noms long (FQDN) et court.

    Remarque: aux fins de cette formation, nous n'utiliserons pas de DNS. Nous nous fierons
          sur le fichier /etc/hosts pour la résolution de noms. Dans un déploiement de production
          vous voudriez certainement utiliser des noms de domaine complets (FQDN), et très probablement DNS aussi.

```
sudo vi /etc/hosts
```

Nous voulons que notre fichier `/etc/hosts` ressemble à ceci:

```
127.0.0.1      localhost
192.168.198.10 puppet.example.com puppet
192.168.198.11 agent.example.com  agent
192.168.198.12 gitlab.example.com gitlab
```

Dans un lab ultérieur, nous configurerons Puppet pour gérer les entrées /etc/hosts pour nous.

** Configurer le pare-feu **

Configurer le pare-feu hôte pour permettre à Puppet de fonctionner conformément au [PE Install Guide - Firewall Config](https://docs.puppet.com/pe/latest/sys_req_sysconfig.html#for-monolithic-installs)

```shell
sudo su -
firewall-cmd --permanent --add-service=https   # PE Console (default port 443)
firewall-cmd --permanent --add-port=3000/tcp   # PE web-based installer
firewall-cmd --permanent --add-port=8080/tcp   # PuppetDB
firewall-cmd --permanent --add-port=8081/tcp   # PuppetDB
firewall-cmd --permanent --add-port=8140/tcp   # Puppet Master, Certificate Authority
firewall-cmd --permanent --add-port=8142/tcp   # Orchestration services
firewall-cmd --permanent --add-port=8143/tcp   # The Orchestrator client uses this port to communicate with the orchestration services
firewall-cmd --permanent --add-port=61613/tcp  # MCollective / ActiveMQ
firewall-cmd --permanent --add-port=4432/tcp   # Local connections for node classifier, activity service, and RBAC status checks
firewall-cmd --permanent --add-port=4433/tcp   # Classifier / Console Services API endpoint
firewall-cmd --permanent --add-port=8170/tcp   # Code Manager
firewall-cmd --reload
firewall-cmd --list-all
exit # drop out of root shell
```

La sortie de votre `firewall-cmd --list-all` devrait ressembler à ceci:

```
[vagrant@puppet ~]$ firewall-cmd --list-all
public (default, active)
  interfaces: enp0s3 enp0s8
  sources:
  services: dhcpv6-client https ssh
  ports: 3000/tcp 8140/tcp 8170/tcp 8080/tcp 4433/tcp 8081/tcp 4432/tcp 8143/tcp 8142/tcp 61613/tcp
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
```

D'accord, nous sommes maintenant prêts à exécuter le programme d'installation de Puppet Enterprise ...

---

Continuez vers **Lab #3** -> [Installer Puppet Master](03-Install-Puppet-Master.md#lab-3)

---

<- [Retour au sommaire](/README.md)
