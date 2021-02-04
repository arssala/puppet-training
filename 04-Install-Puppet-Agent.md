<- [Retour](03-Install-Puppet-Master.md#lab-3)

---

### **Lab #4** - Installer l'agent Puppet

---

### Aperçu

Temps pour terminer: 10-15 minutes

Dans ce lab, nous allons:

* Modifiez manuellement le fichier /etc/hosts pour préparer l'installation de l'agent (si ce n'est déjà fait)
* Installez l'agent Puppet sur le nœud **agent**

### Étapes de pré-installation

Assurez-vous que votre VM **agent** est démarrée et y connectez-vous.

```shell
     vagrant up agent
     vagrant ssh agent
```

### Modifier le fichier hosts

Encore une fois, notre fichier `/etc/hosts` devrait déjà être configuré, car nous avons utilisé un box Vagrant pré-configuré qui contient déjà les entrées souhaitées, mais par souci d'exhaustivité,
revérifions ...

Affichez et/ou modifiez `/etc/hosts` et assurez-vous qu'il ressemble exactement à ceci (supprimez toutes les autres lignes):
(Cela devrait être exactement le même fichier d'hôtes que celui que nous avons créé sur le nœud maître **puppet**)

```shell
127.0.0.1      localhost
192.168.198.10 puppet.example.com puppet
192.168.198.11 agent.example.com  agent
192.168.198.12 gitlab.example.com gitlab
```

Dans un lab ultérieur, nous écrirons le code Puppet pour maintenir les entrées /etc/hosts,
mais pour l'instant, nous les ajoutons manuellement.

### Installer l'agent

Pour installer l'agent Puppet sur le nœud **agent**, nous pouvons profiter d'une
caractéristique du PE Master: Le PE Master met à disposition l'installateur de l'agent
à partir de son propre serveur Web. Vous pouvez utiliser **wget** ou **curl** pour télécharger le
script d'installation, puis rediriger-le vers bash.

* Pour utiliser **wget**

```shell
     wget --no-check-certificate --secure-protocol=TLSv1 -O - https://puppet:8140/packages/current/install.bash | bash -s main:certname=agent.example.com
```

* Pour utiliser **curl**

```shell
     curl -k --tlsv1 https://puppet:8140/packages/current/install.bash | bash -s main:certname=agent.example.com
```

Si vous souhaitez parcourir ce qui est accessible via ce serveur Web, essayez
ouverture de <https://localhost:22140/packages> dans le navigateur Web de votre poste de travail.

(N'oubliez pas que nous avons transféré le port **8140** <--> **22140** sur notre station de travail hôte)

Allez-y et installez l'agent si vous ne l'avez pas déjà fait, puis
essayez d'exécuter l'agent de Puppet ...

* Sortie d'installation de Puppet: [04-Puppet-Agent-Install-Output.md](04-Puppet-Agent-Install-Output.md)


### Exécutez l'agent Puppet

Exécutez l'agent Puppet manuellement. Cela entraînera une demande de certificat SSL
à générer et à envoyer au Puppet Master (si ce n'est pas déjà envoyée
en arrière-plan.)

```shell
     [root@agent ~]# puppet agent -t
     Info: Creating a new SSL key for agent.example.com
     Info: Caching certificate for ca
     Info: Caching certificate_request for agent.example.com
     Info: Caching certificate for ca
     Exiting; no certificate found and waitforcert is disabled
```

Remarque: l'agent de Puppet fonctionne également en arrière-plan, il peut donc avoir déjà généré la clé SSL,
vous risquez donc de ne pas le voir dans votre sortie. Dans ce cas, vous verriez juste ceci:

```shell
[root@agent ~]# puppet agent -t
Exiting; no certificate found and waitforcert is disabled
```

### Signez le certificat

Ensuite, nous devons signer le certificat de l'agent sur le Master, alors passez à votre **Puppet**
window/terminal et exécutez les commandes suivantes sur le Puppet Master en tant que root:

```shell
     puppet cert list
     puppet cert sign agent.example.com
```

La commande **puppet cert list** affiche toutes les demandes de signature de certificat en attente. Vous devriez voir celui qui vient d'être généré par l'exécution de votre agent.

```shell
     [root@puppet ~]# puppet cert list
       "agent.example.com" (SHA256) 31:EA:4D:60:DE:44:E8:E1:A1:1A:2E:48:1E:81:CA:40:43:4A:A7:39:E8:B9:61:63:F3:0F:CF:2E:B7:CC:98:22
```

La commande **puppet cert sign agent.example.com** signe le certificat et supprime la demande de signature.

```shell
     [root@puppet ~]# puppet cert sign agent.example.com
     Notice: Signed certificate request for agent.example.com
     Notice: Removing file Puppet::SSL::CertificateRequest agent.example.com at '/etc/puppetlabs/puppet/ssl/ca/requests/agent.example.com.pem'
```

### Exécuter Puppet ...

Maintenant, revenons sur le nœud de l'agent: exécutons à nouveau puppet (assurez-vous que vous êtes root)

```shell
     puppet agent -t
```

Vous devriez voir beaucoup de sortie à l'écran montrant les changements qui sont appliqués.
(Puppet is installing and configuring MCollective on the agent.)
Cependant, comme Puppet s'exécute automatiquement en arrière-plan toutes les 5 minutes avant
son certificat étant signé, il y a une petite chance que la première exécution de Puppet
se produise avant que vous ne puissiez effectuer une exécution manuelle. Dans ce cas, vous devriez voir une petite sortie comme dans la deuxième exécution de Puppet (no changes made.)

### Réexécutez l'agent Puppet

Exécutez puppet une **seconde** fois, et vous devriez obtenir une exécution propre sans changement.

```shell
     puppet agent -t
```

![Puppet Agent Clean Run](images/Puppet-Agent-Clean-Run-agent.png)

Par souci de concision, je n'ai pas inclus la sortie sur cette page, mais elle est disponible pour consultation ici:

* Sortie d'exécution de Puppet: [04-Puppet-Agent-Run-Output.md](04-Puppet-Agent-Run-Output.md)

### Le nom du certificat

Le **certname** est le nom attribué à l'agent Puppet. C'est comme ça que Puppet
master identifie chaque nœud. La plupart du temps, le certname sera le même
que le nom de domaine complet de l'hôte, mais cela peut être tout ce que vous choisissez. La meilleure pratique consiste à
utiliser le FQDN.

Vous avez peut-être remarqué que lorsque nous avons installé l'agent Puppet, nous avons passés
un paramètre supplémentaire utilisant la construction `bash -s <clé:valeur>`. Par défaut, puppet prendra le nom d'hôte et ajoutera le nom de domaine tel que configuré dans le fichier /etc/resolv.conf.
Nous n'avons pas configuré le /etc/resolv.conf du tout, et Vagrant le configure automatiquement lorsque
la VM est démarrée à chaque fois. Il héritera de la configuration DNS de l'hôte système.
La configuration héritée sera différente pour tout le monde, et depuis,
le programme d'installation de l'agent essaiera normalement de déterminer le **certname** à partir du
nom d'hôte et domaine DNS, nous ne voulons pas nous y fier ici .
En spécifiant manuellement le nom du certificat, nous garantissons que nous
obtiendra toujours le nom que nous voulons, plutôt que de laisser la Puppet essayer de choisir
ça pour nous.

Il y a d'autres paramètres que nous souhaitons peut-être également transmettre au script d'installation.
Vous pouvez transmettre n'importe quel élément de configuration qui a du sens. Exécutez simplement `puppet config print` pour voir les possibilités.

Pour en savoir plus sur la transmission de paramètres au programme d'installation de l'agent, consultez: https://docs.puppet.com/pe/latest/install_agents.html#passing-configuration-parameters-to-the-install-script

### Sommaire

À ce stade, nous avons:

- un nœud **Puppet Master** (hostname **puppet.example.com**) qui exécute également un agent pour se configurer
- un nœud **Puppet Agent** (hostname **agent.example.com**) qui exécute un agent, et où nous testerons le code et en apprendrons plus sur PE

Si vous vous connectez à la [PE console](https://127.0.0.1:22443/), vous devriez voir ces deux agents sur la page 'Nodes'

---

Continuez vers **Lab #5** -> [Familiarisez-vous avec les fichiers de configuration des Puppet, le code des Puppet et la CLI](05-Puppet-Config-and-Code.md#lab-5)

---

<- [Retour au sommaire](/README.md)

---
