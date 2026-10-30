# Lab réseau : cabinet notarial

## Objectif
Concevoir et configurer le réseau d'un cabinet de notaire avec
segmentation, téléphonie IP et accès invités sécurisé.

## Topologie
![Topologie](images/topologie.png)

## Compétences mises en œuvre
- VLANs et trunk 802.1Q
- Routage inter-VLAN sur switch L3
- DHCP centralisé (exclusions, option 150 pour la VoIP)
- Téléphonie IP avec Cisco CME (5 postes)
- ACL pour isoler le Wi-Fi invité
- Accès distant sécurisé (SSH v2)
- Routage statique

## Plan d'adressage
| VLAN | Usage | Réseau |
|---|---|---|
| 10 | Notaire | 192.168.10.0/24 |
| 20 | Clercs | 192.168.20.0/24 |
| 30 | Réception | 192.168.30.0/24 |
| 50 | Guest-Wifi | 192.168.50.0/24 |
| 60 | Voix | 192.168.60.0/24 |
|---|---|---|
plus de détails dans le fichier de documentation.

## Problèmes rencontrés et solutions
Description du souci DHCP bloqué par l'ACL invités et de sa correction.

## Contenu du dépôt
- `packet-tracer/` : **fichier .pkt à ouvrir avec Packet Tracer**
- `configs/` : **running-config de chaque appareil**
- `docs/` : **documentation complète** (Word)

## Améliorations prévues
Sortie Internet (NAT), durcissement STP, isolation du VLAN Notaire.
