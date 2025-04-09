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
#3.3 Installation d’un serveur Web:
