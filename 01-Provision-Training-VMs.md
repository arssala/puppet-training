

<- [Back](/README.md#labs)

---

## **Lab #1** - Installer les logiciels et utiliser Vagrant pour déployer 3 VM pour la formation



### Aperçu

Temps nécessaire: 30 minutes

Dans le lab suivant, vous installerez les logiciels suivants:

1. **Git** (devrait déjà être installé) - le système de contrôle de version
2. **VirtualBox** - l'hyperviseur utilisé par défaut par Vagrant
3. **Vagrant** - l'outil de déploiement de machine virtuelle

... puis créerez 3 VMs VirtualBox nommées comme suit:

1. **puppet** (Puppet Master, PE Console, etc.)
3. **agent** (Un agent de Puppet)
2. **gitlab** (GitLab, qui exécutera également l'agent Puppet)

### Etapes
* Téléchargez les logiciels nécessaires pour ce tutoriel ...
```
[puppet-tutorial-pe]$ cd share
[puppet-tutorial-pe/share]$ cd software
[puppet-tutorial-pe/share/software]$ ./download-all.sh
```

* Installez VirtualBox
      - trouvez le programme d'installation approprié dans le répertoire `puppet-tutorial-pe/share/software/virtualbox/`
      - Des programmes d'installation pour Mac OS X et Windows sont fournis, mais d'autres peuvent également être téléchargés
      - autres installateurs disponibles ici: <https://www.virtualbox.org/wiki/Downloads>

* Démarrez VirtualBox et configurez `Default Machine Folder` pour placer les VMs dans un répertoire autre que celui par défaut (si vous le souhaitez), puis quittez à nouveau VirtualBox.

* Installez Vagrant
      - trouvez le programme d'installation approprié dans `puppet-tutorial-pe/share/software/vagrant`
      - d'autres disponibles ici: <https://www.vagrantup.com/downloads.html>

* cd dans `puppet-tutorial-pe/`

* Nous installons un plugin Vagrant pour gérer les `VirtualBox guest additions`

* Puis `vagrant up puppet` pour provisionner la VM

```
cd puppet-tutorial-pe
vagrant plugin install vagrant-vbguest
vagrant up puppet
```

* une fois que la VM **puppet** est provisionnée, familiarisez-vous avec vagrant et confirmez que les spécifications de la VM sont correctes:

```
    vagrant ssh puppet              # to verify that you are able to login (should not be prompted for password)
    cat /etc/centos-release         # what CentOS release are we running?
    cat /proc/cpuinfo | grep proc   # how many vCPU's does the VM have? (should see 4 processors)
    lscpu                           # another way to see CPU info
    free -h                         # how much RAM is available on this VM?
    df -h /share                    # is our shared space mounted up? (should see 'share' filesystem mounted on /share)
    ls -al /share/software          # should see our puppet/ directory in there
    exit                            # and drop out of the VM, back to host OS shell prompt
```

### Arrêt de votre VM gérée par Vagrant

Lorsque vous avez terminé de travailler dans un lab en particulier et que vous souhaitez arrêter votre VM, exécutez la commande `vagrant halt` suivie du nom de la VM, comme suit:

```
    vagrant halt puppet        # and wait for VM to shutdown
```

### Démarrage de votre VM gérée par Vagrant

Si vous avez exécuté  `vagrant halt puppet` pour tester l'arrêt, vous pouvez exécuter `vagrant up puppet` avant de continuer pour la redémarrer ...

Si tout se passe bien avec ce qui précède, nous pouvons être sûrs que notre VM fonctionne comme prévu et passons à autre chose.

### Connectez-vous à nouveau à votre VM ...

Reconnectons-nous à la machine virtuelle *Puppet* avec `vagrant ssh puppet`

Vous devriez voir qui est `/share` dans la sortie `df`. Il s'agit de votre espace de système de fichiers partagé qui est également accessible en dehors de la machine virtuelle

```
$ vagrant ssh puppet
Last login: Fri Oct 21 15:14:38 2020 from 10.0.2.2
[vagrant@puppet ~]$ df -h /share
Filesystem      Size  Used Avail Use% Mounted on
none            223G  206G   18G  93% /share
```

Ce répertoire `/share` est accessible **à la fois** dans votre VM et depuis votre système hôte.
Par exemple, si vous copiez un fichier dans `/share` dans votre VM, vous pourrez le trouver à partir de votre OS hôte dans `puppet-tutorial-pe/share/` (et vice versa)

Si pour une raison quelconque, vous ne voyez pas le système de fichiers `/share/` monté, essayez d'arrêter votre VM et de redémarrer. Parfois, les outils VMware doivent être mis à jour et un arrêt/démarrage déclenchera que:

```
[vagrant @ puppet ~]$ exit
[puppet-tutorial-pe]$ vagrant halt Puppet
[puppet-tutorial-pe]$ vagrant up puppet
[puppet-tutorial-pe]$ vagrant ssh puppet
```

... et vérifiez à nouveau.

### Explorez plus de commandes Vagrant

Familiarisez-vous avec Vagrant et connectez-vous à votre VM. Les commandes les plus utiles dont vous aurez besoin sont:
```
vagrant up <name>          # start the VM, and provision it if it's the first time
vagrant halt <name>        # shutdown the VM cleanly
vagrant global-status      # show the status of your VMs (powered on or off)
vagrant destroy <name>     # if you want to completely destroy your VM and start over
vagrant ssh <name>         # connect to your VM with ssh
vagrant ssh-config <name>  # show ssh config in case you want to use PuTTY to connect to your VM
```

### Provisionnez deux machines virtuelles supplémentaires

Vous devrez également provisionner 2 VM supplémentaires, une pour **GitLab** et une de plus qui exécutera l'**agent** puppet . (En fait, les 3 exécuteront la Puppet agent, mais nous créons une petite machine virtuelle distincte pour les tests afin de ne pas avoir s'inquiéter de casser quelque chose pendant que nous testons.)

```
vagrant up gitlab          # provision the VM for your GitLab server
vagrant up agent           # provision your VM that will be one of your puppet agents
```

Une fois que vous avez provisionné vos VM **gitlab** et **agent**, vous devriez les voir dans la sortie de ** vagrant global-status **


```
$ vagrant global-status
id       name         provider   state    directory
------------------------------------------------------------------------------
4dd5ed7  puppet       virtualbox running  /Users/Mark/Vagrant/puppet-tutorial-pe
070258c  gitlab       virtualbox running  /Users/Mark/Vagrant/puppet-tutorial-pe
682eafe  agent        virtualbox running  /Users/Mark/Vagrant/puppet-tutorial-pe

The above shows information about all known Vagrant environments
on this machine. This data is cached and may not be completely
up-to-date. To interact with any of the machines, you can go to
that directory and run Vagrant, or you can use the ID directly
with Vagrant commands from any directory. For example:
"vagrant destroy 1a2b3c4d"
```

### Validez votre environnement de formation

Nous avons utilisé Vagrant pour provisionner 3 machines virtuelles d'entraînement. Si vous êtes curieux de savoir comment
Les VM sont configurées, jetez un œil au [puppet-tutorial-pe/Vagrantfile](/ Vagrantfile)

Nous avons utilisé une boîte de base personnalisée qui contient déjà certains éléments dont nous avons besoin (packages, configuration, etc.)
Cette boîte de base personnalisée a déjà des entrées dans `/ etc/hosts` que nous voulons/avons besoin (puisque nous allons
comptez sur/etc/hosts pour résoudre les noms dans notre environnement de formation.)

Chaque VM doit avoir les contnets suivants dans leur fichier `/ etc/hosts`:

```
127.0.0.1       localhost
192.168.198.10  puppet.example.com  puppet
192.168.198.11  agent.example.com   agent
192.168.198.12  gitlab.example.com  gitlab
```

Avec les 3 de vos machines virtuelles opérationnelles, vérifiez que vous pouvez envoyer un ping à chaque
le FQDN ainsi que le nom court.

Par exemple:


```
[vagrant@puppet ~]$ ping puppet.example.com
PING puppet.example.com (192.168.198.10) 56(84) bytes of data.
64 bytes from puppet.example.com (192.168.198.10): icmp_seq=1 ttl=64 time=0.043 ms
64 bytes from puppet.example.com (192.168.198.10): icmp_seq=2 ttl=64 time=0.098 ms
64 bytes from puppet.example.com (192.168.198.10): icmp_seq=3 ttl=64 time=0.075 ms
^C
--- puppet.example.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1999ms
rtt min/avg/max/mdev = 0.043/0.072/0.098/0.022 ms

[vagrant@puppet ~]$ ping puppet
PING puppet.example.com (192.168.198.10) 56(84) bytes of data.
64 bytes from puppet.example.com (192.168.198.10): icmp_seq=1 ttl=64 time=0.052 ms
64 bytes from puppet.example.com (192.168.198.10): icmp_seq=2 ttl=64 time=0.088 ms
64 bytes from puppet.example.com (192.168.198.10): icmp_seq=3 ttl=64 time=0.098 ms
^C
--- puppet.example.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1999ms
rtt min/avg/max/mdev = 0.052/0.079/0.098/0.021 ms
```

Vous devriez également pouvoir effectuer des ssh entre chaque VM en tant qu'utilisateur ** vagrant **
ou l'utilisateur **root**. Le mot de passe par défaut est simplement **"vagrant"**. Donne-le
essayez si vous le souhaitez. Vous devriez également pouvoir effectuer une connexion SSH vers l'une de vos VM à partir de
l'hôte lui-même. Vous pouvez apprendre le port ssh à utiliser en utilisant le `vagrant ssh-config`
commander. Essaie:

```
[puppet-tutorial-pe]$ vagrant ssh-config puppet
Host puppet
  HostName 127.0.0.1
  User vagrant
  Port 22022
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /Users/Mark/.vagrant.d/boxes/bentlema-VAGRANTSLASH-centos-7.2-64/1.0.0/virtualbox/vagrant_private_key
  IdentitiesOnly yes
  LogLevel FATAL
```

Notez que le port NAT est 22022.

```
[puppet-tutorial-pe]$ ssh root@127.0.0.1 -p 22022
Warning: Permanently added '[127.0.0.1]:22022' (ECDSA) to the list of known hosts.
root@127.0.0.1's password:
Last login: Fri Nov 11 21:57:06 2016
[root@puppet ~]# exit
logout
Connection to 127.0.0.1 closed.
```

Cette capacité à ssh entre nos VM deviendra utile dans un lab ultérieur (lorsque nous configurons R10K pour extraire le code de GitLab.)

### Ensuite ...

Ceci conclut le lab n ° 1. Vous devriez maintenant avoir 3 machines virtuelles opérationnelles nommées comme
indiqué ci-dessus dans la sortie **global-status** ci-dessus.


Si vous ne prévoyez pas de passer immédiatement au lab suivant, vous voudrez peut-être
pour arrêter toutes vos machines virtuelles d'entraînement. Assurez-vous d'avoir quitté la session ssh
de chaque VM, et revenez à l'invite du shell sur votre poste de travail, puis émettez
le "vagrant halt" pour chaque VM comme suit:

```
vagrant halt puppet
vagrant halt agent
vagrant halt gitlab
```

Remarque: faire un **arrêt** arrêtera en douceur votre VM, pas simplement
il éteint. Vous n'avez **pas** besoin d'arrêter d'abord le système d'exploitation qui s'exécute dans la machine virtuelle. Si vous
faire cela, le **vagrant halt** ne pourra pas s'arrêter gracieusement,
timeout essayant, puis éteignez la machine virtuelle.

---

Continuez vers **Lab #2** -> [Préparez-vous à installer Puppet Enterprix](02-Prep-to-Install-Puppet-Master.md # lab-2)

---

<- [Retour au sommaire](/README.md)
