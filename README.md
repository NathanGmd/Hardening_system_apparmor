# Hardening_system_apparmor

### 3.2 Sécurisation de l’administration du serveur :
Dans le cadre du renforcement de la sécurité du serveur, l’authentification SSH a été configurée avec un jeu de clés Ed25519, garantissant de meilleures performances et une sécurité accrue par rapport aux clés RSA traditionnelles.

L’accès SSH est désormais restreint aux connexions par clé, avec l’authentification par mot de passe désactivée. Par mesure de sécurité supplémentaire, seul le port 2222/TCP a été ouvert dans le pare-feu, limitant ainsi l’exposition du service SSH aux connexions non autorisées.
De plus, uniquement mon utilisateur Admin est autorisé à la connexion

Cette configuration permet de sécuriser efficacement l’accès au serveur tout en maintenant un niveau de flexibilité optimal pour l’administration distante.

Audit ssh par lynis du serveur :

```
[+] Prise en charge SSH
------------------------------------
  - Checking running SSH daemon                               [ TROUVÉ ]
    - Searching SSH configuration                             [ TROUVÉ ]
    - OpenSSH option: AllowTcpForwarding                      [ OK ]
    - OpenSSH option: ClientAliveCountMax                     [ OK ]
    - OpenSSH option: ClientAliveInterval                     [ OK ]
    - OpenSSH option: FingerprintHash                         [ OK ]
    - OpenSSH option: GatewayPorts                            [ OK ]
    - OpenSSH option: IgnoreRhosts                            [ OK ]
    - OpenSSH option: LoginGraceTime                          [ OK ]
    - OpenSSH option: LogLevel                                [ OK ]
    - OpenSSH option: MaxAuthTries                            [ OK ]
    - OpenSSH option: MaxSessions                             [ OK ]
    - OpenSSH option: PermitRootLogin                         [ OK ]
    - OpenSSH option: PermitUserEnvironment                   [ OK ]
    - OpenSSH option: PermitTunnel                            [ OK ]
    - OpenSSH option: Port                                    [ OK ]
    - OpenSSH option: PrintLastLog                            [ OK ]
    - OpenSSH option: StrictModes                             [ OK ]
    - OpenSSH option: TCPKeepAlive                            [ OK ]
    - OpenSSH option: UseDNS                                  [ OK ]
    - OpenSSH option: X11Forwarding                           [ OK ]
    - OpenSSH option: AllowAgentForwarding                    [ OK ]
    - OpenSSH option: AllowUsers                              [ TROUVÉ ]
    - OpenSSH option: AllowGroups                             [ NON TROUVÉ ]
```

Configuration du firewall :

```
ngermond@ngermond-VirtualBox:~$ sudo iptables -L -v -n
Chain INPUT (policy DROP 111 packets, 19662 bytes)
 pkts bytes target     prot opt in     out     source               destination
  841 66393 ACCEPT     6    --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:2222
    0     0 ACCEPT     6    --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:80
   27  3054 ACCEPT     0    --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED

Chain FORWARD (policy DROP 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 957 packets, 78711 bytes)
 pkts bytes target     prot opt in     out     source               destination
```
80 ouvert pour le httpd plus tard.

### 3.3 Installation d’un serveur Web:

AppArmor peut fonctionner de différentes façons selon le niveau de contrôle qu’on veut appliquer sur un programme. Il peut soit surveiller sans bloquer, soit bloquer pour de vrai, soit ne rien faire du tout. Le mode le plus utilisé en production, c’est le mode enforce, où le profil est strictement appliqué : si le programme tente une action qui n’est pas prévue, c’est bloqué. On passe souvent par le mode complain, qui permet de voir ce que le programme ferait en dehors du cadre sans l’empêcher de fonctionner. Cela permet d'ajuster les règles sans casser les apps. Sinon on désactive.\
Le programme va se heurter à des refus d’accès un peu partout, que ce soit pour lire, écrire ou accéder à certaines ressources. Ces blocages peuvent entraîner des dysfonctionnements, voire empêcher le programme de démarrer correctement. C’est pour ça que le passage en enforce doit se faire après avoir bien testé et affiné le profil.

### 3.4 Configuration d’un profil Apparmor

Il faut penser a modifier dans logprof.conf, dans la partie Qualifier, la ligne "/usr/bin/ls = icn" et tout simplement la supprimer. Ce sont des sécurités d'apparmor pour ne pas tout casser.
Pour générer un profils de base pour la commande linux :
```
sudo aa-genprof /bin/ls
```
Une fois la commande lancé, il faut ouvrir un nouveau terminal et exécuter plusieurs fois la commande ls dans différents contexte puis retourner sur la génération du profils et appuyer sur scan afin que apparmor puisse scanner les logs.
Puis pour chaque event de ls, il faudra informer apparmor si l'on autorise ou non (cf exemple) :
```
Profil:         /usr/bin/ls
Chemin d'accès: /home/ngermond/
Nouveau mode:   owner r
Gravité:        4

 [1 - owner /home/*/ r,]
  2 - owner /home/ngermond/ r,
(A)llow / [(D)eny] / (I)gnore / (G)lob / Glob with (E)xtension / (N)ew / Audi(t) / (O)wner permissions off / Abo(r)t / (F)inish
Adding owner /home/*/ r, to profile.
```

La commande pour voir l'état des profils est :
```
sudo aa-status
```
Cela liste tout les profils et leurs état actuel.\
Je vais donc passer mon profils /usr/bin/ls en mode enforce et voir ce qu'il se passe 
```
sudo aa-enforce /usr/bin/ls
```
Dans un premier temps, lors de mon scan, je n'ai pas fais de ls simple, juste des ls /home par exemple, donc quand je fais un ls global simple, je me prend une "permission denied". Ce que l'on peut d'ailleurs constater dans les logs avec la commande : 
```
sudo dmesg | grep apparmor
```
```
[ 9667.359156] audit: type=1400 audit(1744203801.442:365): apparmor="DENIED" operation="open" class="file" profile="/usr/bin/ls" name="/" pid=108836 comm="ls" requested_mask="r" denied_mask="r" fsuid=1000 ouid=0
```
Pour améliorer mon profils, je vais repasser mon profils en mode complain, faire le maximum de test avec ma commande ls puis re définir mes permissions pour chaque event de ls en utilisant la commande :
```
sudo aa-logprof
```
Je remarque en mode complain que par exemple mon ls dev ne passe pas : 
```
[12748.146052] audit: type=1400 audit(1744206882.107:427): apparmor="ALLOWED" operation="file_perm" class="file" profile="/usr/bin/ls" name="/dev/" pid=109268 comm="ls" requested_mask="r" denied_mask="r" fsuid=1000 ouid=0
```
Je l'autorise en écrivant directement dans /etc/apparmor.d/usr.bin.ls :
```
abi <abi/3.0>,

include <tunables/global>

/usr/bin/ls flags=(complain) {
  include <abstractions/base>
  include <abstractions/opencl-pocl>

  /etc/apparmor.d/ r,
  /home/ r,
  /srv/ r,
  /usr/bin/ls mr,
  /var/ r,
  /etc/ r,
  /dev/ r,
  owner /home/*/ r,

}
```
De ce fais je peux maintenant accéder a ls dev.
  


