# Lab réseau : cabinet notarial

Lab réalisé sur **Cisco Packet Tracer** : conception et configuration du réseau d'un cabinet de notaire, avec segmentation par VLAN, routage inter-VLAN, DHCP centralisé, téléphonie IP, Wi-Fi invité isolé et accès d'administration sécurisé.

!\[Topologie du lab](images/topologie.png)

## Objectif

Mettre en place un réseau d'entreprise réaliste pour une petite structure :

* séparer les services (notaire, clercs, réception, imprimante, invités, voix, management) ;
* fournir automatiquement les adresses IP et la configuration téléphonique ;
* isoler le Wi-Fi invité du réseau interne ;
* permettre l'administration à distance des équipements en SSH.

## Compétences mises en œuvre

|Domaine|Détail|
|-|-|
|Commutation|VLANs, trunk 802.1Q, ports d'accès, PortFast, voice VLAN, PVST|
|Routage|Routage inter-VLAN sur switch L3 (SVI), routes statiques, route par défaut|
|Services|DHCP multi-pools avec exclusions, option 150 pour la VoIP|
|Téléphonie|Cisco CME (Call Manager Express), 5 postes IP 7960|
|Sécurité|ACL étendue pour le VLAN invité, SSH v2, authentification locale, secrets chiffrés|
|Documentation|Plan d'adressage, tableaux de configuration, analyse des problèmes|

## Topologie

```
EDGE-R1 (Fa0/1) ── 10.0.0.0/30 ── (Gi0/1) L3-SW1 (Gi0/2) ══ trunk ══ (Gi0/2) ACCESS-SW1
```

|Appareil|Modèle|Rôle|
|-|-|-|
|EDGE-R1|Cisco 2811|Routeur de bordure + serveur de téléphonie CME|
|L3-SW1|Switch multicouche|Cœur de réseau, routage inter-VLAN, serveur DHCP|
|ACCESS-SW1|Switch d'accès|Raccordement des postes, téléphones, imprimante et borne Wi-Fi|

## Plan d'adressage

### Liaison point à point

|Appareil|Interface|Adresse|
|-|-|-|
|EDGE-R1|Fa0/1|10.0.0.1/30|
|L3-SW1|Gi0/1 (port routé)|10.0.0.2/30|

### VLANs

|VLAN|Usage|Réseau|Passerelle|DHCP|Exclusions|
|-|-|-|-|-|-|
|10|Notaire|192.168.10.0/24|192.168.10.1|Oui|.1 à .20|
|20|Clercs de notaire|192.168.20.0/24|192.168.20.1|Oui|.1 à .20|
|30|Réception|192.168.30.0/24|192.168.30.1|Oui|.1 à .20|
|40|Imprimante|192.168.40.0/24|192.168.40.1|Non (IP statique)|n/a|
|50|Wi-Fi invité|192.168.50.0/24|192.168.50.1|Oui|.1 à .20|
|60|Voix|192.168.60.0/24|192.168.60.1|Oui + option 150|.1 à .20|
|99|Management|192.168.99.0/24|192.168.99.1|Non (IP statique)|n/a|

Serveur DNS distribué aux clients : `8.8.8.8` (hors VLAN 60).

### Adresse de management

|Appareil|Interface|Adresse|
|-|-|-|
|L3-SW1|Vlan99|192.168.99.1/24|
|ACCESS-SW1|Vlan99|192.168.99.2/24 (passerelle 192.168.99.1)|

## Configuration des ports d'ACCESS-SW1

|Port|Description|VLAN data|VLAN voix|
|-|-|-|-|
|Fa0/1 à Fa0/3|Clercs (téléphone + PC)|20|60|
|Fa0/4|Notaire (téléphone + PC)|10|60|
|Fa0/5|Réception (téléphone + PC)|30|60|
|Fa0/6|Imprimante|40|n/a|
|Fa0/7|Borne Wi-Fi invité|50|n/a|
|Gi0/2|Trunk vers L3-SW1 (VLAN 10, 20, 30, 40, 50, 60, 99)|n/a|n/a|

## Routage

|Appareil|Route|Détail|
|-|-|-|
|EDGE-R1|`ip route 192.168.0.0 255.255.0.0 10.0.0.2`|Retour vers tous les VLAN internes|
|L3-SW1|`ip route 0.0.0.0 0.0.0.0 10.0.0.1`|Route par défaut vers EDGE-R1|

## Téléphonie IP (Cisco CME)

* Source de téléphonie : `10.0.0.1`, port SCCP 2000
* 5 téléphones Cisco 7960, lignes **1001 à 1005**, attribution automatique
* Les téléphones obtiennent leur adresse dans le VLAN 60 et trouvent le CME grâce à l'**option DHCP 150** (`10.0.0.1`)

## Sécurité

* **SSH v2** activé sur les switches (domaine `opn.local`), Telnet désactivé (`transport input ssh`)
* Authentification locale (`login local`) avec un utilisateur dédié
* Accès disrtant : Username : - **Opn** – Mot de passe : - **Opn123** –
* Mot de passe wifi : **Opn12345**
* **ACL `GUEST\\\_RESTRICTIONS`** appliquée en entrée sur le VLAN 50 pour interdire l'accès aux réseaux internes tout en laissant la sortie vers l'extérieur

> Les hash de mots de passe ont été retirés des fichiers de configuration publiés dans ce dépôt.



## Vérifications utiles

```
show vlan brief
show interfaces trunk
show ip interface brief
show ip route
show ip dhcp binding
show access-lists
show ip ssh
show telephony-service ephone-dn      (sur EDGE-R1)
show ephone registered                (sur EDGE-R1)
```

## Améliorations prévues

* \[ ] NAT/PAT et sortie Internet sur EDGE-R1
* \[ ] VLAN natif dédié sur le trunk
* \[ ] L3-SW1 forcé en racine STP
* \[ ] ACL pour isoler le VLAN Notaire des VLAN Clercs et Réception

## Contenu du dépôt

```
.
├── README.md
├── packet-tracer/
│   └── lab-cabinet-notarial.pkt     # à ouvrir avec Cisco Packet Tracer
├── configs/
│   ├── EDGE-R1\\\_running-config.txt
│   ├── L3-SW1\\\_running-config.txt
│   └── ACCESS-SW1\\\_running-config.txt
├── docs/
│   └── Documentation\\\_Lab\\\_Packet\\\_Tracer.docx
└── images/
    └── topologie.png
```

## Utilisation

1. Installer [Cisco Packet Tracer](https://www.netacad.com/).
2. Ouvrir le fichier `.pkt` du dossier `packet-tracer/`.
3. Consulter les configurations dans `configs/` ou directement via le CLI de chaque équipement.

## Auteur

**\[Tahar HAdjeres]** : en reconversion vers l'IT, orienté technicien réseau.
[LinkedIn](https://www.linkedin.com/in/ton-profil) · [GitHub](https://github.com/ton-nom)

