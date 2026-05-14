# Lab Routage Statique IPv4/IPv6 — Cisco Packet Tracer

## Objectif

Mettre en place un réseau multi-sites avec routage statique IPv4 et IPv6 dans Cisco Packet Tracer. Le lab simule l'interconnexion de deux sites d'entreprise distants via un routeur faisant office d'Internet.

## Topologie

```
       ┌──────┐         ┌──────┐
       │ PC0  │         │ PC1  │
       └──┬───┘         └──┬───┘
          │                │
     ┌────┴────────────────┴────┐
     │       Switch0            │
     └──────────┬───────────────┘
                │ Gi0/0
           ┌────┴────┐
           │   R1    │  Site 1 — 192.168.1.0/24
           └────┬────┘
                │ Gi0/1
                │
                │ Gi0/0
           ┌────┴────┐
           │ Internet│
           └────┬────┘
                │ Gi0/1
                │
                │ Gi0/1
           ┌────┴────┐
           │   R2    │  Site 2 — 192.168.2.0/24
           └────┬────┘
                │ Gi0/0
     ┌──────────┴───────────────┐
     │       Switch1            │
     └────┬────────────────┬────┘
          │                │
       ┌──┴───┐         ┌──┴───┐
       │ PC2  │         │ PC3  │
       └──────┘         └──────┘
```

## Plan d'adressage

### IPv4

| Machine         | Interface | Adresse IP      | Masque            | Passerelle    |
|-----------------|-----------|-----------------|-------------------|---------------|
| PC0             | Fa0       | 192.168.1.10    | 255.255.255.0     | 192.168.1.1   |
| PC1             | Fa0       | 192.168.1.11    | 255.255.255.0     | 192.168.1.1   |
| R1 (LAN)        | Gi0/0     | 192.168.1.1     | 255.255.255.0     | —             |
| R1 (WAN)        | Gi0/1     | 198.51.100.1    | 255.255.255.252   | —             |
| Internet (→R1)  | Gi0/0     | 198.51.100.2    | 255.255.255.252   | —             |
| Internet (→R2)  | Gi0/1     | 198.51.100.6    | 255.255.255.252   | —             |
| R2 (WAN)        | Gi0/1     | 198.51.100.5    | 255.255.255.252   | —             |
| R2 (LAN)        | Gi0/0     | 192.168.2.1     | 255.255.255.0     | —             |
| PC2             | Fa0       | 192.168.2.10    | 255.255.255.0     | 192.168.2.1   |
| PC3             | Fa0       | 192.168.2.11    | 255.255.255.0     | 192.168.2.1   |

### IPv6

| Machine         | Interface | Adresse IPv6                  | Passerelle IPv6             |
|-----------------|-----------|-------------------------------|-----------------------------|
| PC0             | Fa0       | 2001:db8:cafe:cafe::10/64     | 2001:db8:cafe:cafe::1       |
| PC1             | Fa0       | 2001:db8:cafe:cafe::11/64     | 2001:db8:cafe:cafe::1       |
| R1 (LAN)        | Gi0/0     | 2001:db8:cafe:cafe::1/64      | —                           |
| R1 (WAN)        | Gi0/1     | 2001:db8:cafe:feed::1/64      | —                           |
| Internet (→R1)  | Gi0/0     | 2001:db8:cafe:feed::2/64      | —                           |
| Internet (→R2)  | Gi0/1     | 2001:db8:babe:feed::2/64      | —                           |
| R2 (WAN)        | Gi0/1     | 2001:db8:babe:feed::1/64      | —                           |
| R2 (LAN)        | Gi0/0     | 2001:db8:babe:babe::1/64      | —                           |
| PC2             | Fa0       | 2001:db8:babe:babe::10/64     | 2001:db8:babe:babe::1       |
| PC3             | Fa0       | 2001:db8:babe:babe::11/64     | 2001:db8:babe:babe::1       |

## Déroulement

### 1 — Mise en place du Site 1

Placement d'un switch 2960 et de deux PCs reliés en câbles droits. Configuration des adresses IPv4/IPv6 et des masques sur PC0 et PC1, puis vérification de la connectivité locale par ping.

### 2 — Configuration du routeur R1

Activation du routage IPv6 avec `ipv6 unicast-routing`, puis configuration de l'interface LAN (Gi0/0) sur le réseau 192.168.1.0/24 et de l'interface WAN (Gi0/1) sur le lien 198.51.100.0/30. Les PCs du site utilisent l'adresse LAN du routeur (192.168.1.1) comme passerelle par défaut.

```
R1(config)# ipv6 unicast-routing
R1(config)# interface gigabitEthernet 0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# ipv6 enable
R1(config-if)# ipv6 address 2001:db8:cafe:cafe::1/64
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface gigabitEthernet 0/1
R1(config-if)# ip address 198.51.100.1 255.255.255.252
R1(config-if)# ipv6 enable
R1(config-if)# ipv6 address 2001:db8:cafe:feed::1/64
R1(config-if)# no shutdown
```

### 3 — Mise en place du Site 2

Même principe que le Site 1 : un routeur R2, un switch 2960, deux PCs (PC2, PC3) sur le réseau 192.168.2.0/24. L'interface WAN de R2 est configurée sur le lien 198.51.100.4/30.

### 4 — Ajout du routeur Internet

Un troisième routeur 2911 est placé entre R1 et R2 pour simuler le réseau Internet. Il est relié aux interfaces WAN de R1 et R2 par des câbles croisés (connexion routeur-routeur). Ses deux interfaces sont configurées sur les liens WAN correspondants.

### 5 — Diagnostic avant routage

A ce stade, un ping de PC0 vers PC2 échoue avec "Destination host unreachable". La commande `show ip route` sur R1 montre qu'il ne connaît que ses réseaux directement connectés (192.168.1.0/24 et 198.51.100.0/30). Il n'a aucune information sur le réseau 192.168.2.0/24 du Site 2.

Un routeur ne route que vers les réseaux qu'il connaît. Sans route configurée, il abandonne le paquet.

### 6 — Configuration des routes statiques

Routes ajoutées sur chaque routeur pour assurer la connectivité complète.

Sur R1 — vers le Site 2, next-hop 198.51.100.2 (routeur Internet) :

```
R1(config)# ip route 192.168.2.0 255.255.255.0 198.51.100.2
R1(config)# ip route 198.51.100.4 255.255.255.252 198.51.100.2
R1(config)# ipv6 route 2001:db8:babe:babe::0/64 2001:db8:cafe:feed::2
R1(config)# ipv6 route 2001:db8:babe:feed::0/64 2001:db8:cafe:feed::2
```

Sur R2 — vers le Site 1, next-hop 198.51.100.6 (routeur Internet) :

```
R2(config)# ip route 192.168.1.0 255.255.255.0 198.51.100.6
R2(config)# ip route 198.51.100.0 255.255.255.252 198.51.100.6
R2(config)# ipv6 route 2001:db8:cafe:cafe::0/64 2001:db8:babe:feed::2
R2(config)# ipv6 route 2001:db8:cafe:feed::0/64 2001:db8:babe:feed::2
```

Sur Internet — vers les deux LANs :

```
Internet(config)# ip route 192.168.1.0 255.255.255.0 198.51.100.1
Internet(config)# ip route 192.168.2.0 255.255.255.0 198.51.100.5
Internet(config)# ipv6 route 2001:db8:cafe:cafe::/64 2001:db8:cafe:feed::1
Internet(config)# ipv6 route 2001:db8:babe:babe::/64 2001:db8:babe:feed::1
```

### 7 — Vérification finale

Ping de PC0 (192.168.1.10) vers PC2 (192.168.2.10) en IPv4 : connectivité OK.
Ping de PC0 vers PC2 (2001:db8:babe:babe::10) en IPv6 : connectivité OK.

La table de routage de R1 après configuration montre les routes statiques (S) en plus des routes connectées (C) et locales (L).

## Commandes de référence

| Commande | Fonction |
|----------|----------|
| `show ip route` | Afficher la table de routage IPv4 |
| `show ipv6 route` | Afficher la table de routage IPv6 |
| `show ip interface brief` | Vérifier l'état des interfaces |
| `ip route [dest] [masque] [next-hop]` | Ajouter une route statique IPv4 |
| `ipv6 route [dest/prefix] [next-hop]` | Ajouter une route statique IPv6 |
| `ipv6 unicast-routing` | Activer le routage IPv6 sur un routeur |
| `no shutdown` | Activer une interface |
| `write memory` | Sauvegarder la configuration |

## Environnement

- Cisco Packet Tracer
- Routeurs : Cisco 2911
- Switches : Cisco 2960-24TT
- Protocoles : IPv4, IPv6, ICMP
- Routage : statique

## Auteur

Zinedine Balamane — Formation TSSR, Wild Code School
