# MODE OPÉRATOIRE – TEST DE PERFORMANCE

## Table des matières
1. [Contexte](#contexte)
2. [Objectif](#objectif)
3. [Architecture](#architecture)
4. [Prérequis](#prérequis)
5. [Tests de performance](#tests-de-performance)
    - [3.1. Mesure du débit nominal entre Datacenter et internet](#31-mesure-du-débit-nominal-entre-datacenter-et-internet)
    - [3.2. Détermination de la valeur du MTU](#32-détermination-de-la-valeur-du-mtu)
    - [3.3. Détermination du débit maximal sur les liaisons Internet et MPLS](#33-détermination-du-débit-maximal-sur-les-liaisons-internet-et-mpls)
6. [Tests de résilience](#tests-de-résilience)

## Contexte
Dans un contexte où les environnements réseau évoluent rapidement et où la qualité de service constitue un enjeu stratégique, MTN doit garantir des performances WAN optimales pour assurer la continuité de ses services et la satisfaction de ses clients. Les liaisons interdatacenters et les connexions Internet représentent des éléments critiques de l’infrastructure, et leur performance réelle doit être mesurée régulièrement afin d’identifier d’éventuels points de congestion et d’optimiser la capacité du réseau.

Dans ce cadre, MTN souhaite mettre en place une campagne de tests utilisant **iPerf**, un outil de référence pour l’évaluation du débit et de la latence sur les réseaux IP. L’objectif est de mesurer avec précision le débit maximal atteignable, de surveiller les variations de latence, et de vérifier la stabilité des liaisons WAN reliant les différents datacenters ainsi que les points de sortie Internet. En plus de cela, des tests de résilience seront effectués afin de déterminer le temps de bascule sur une autre liaison en cas de non disponibilité de la liaison primaire.

## Objectif
- Mesurer le débit nominal entre les Datacenters
- Déterminer la latence nominale entre les Datacenters
- Déterminer la valeur MTU sur le réseau Backbone
- Mesurer le débit nominal vers INTERNET
- Déterminer la latence nominale vers INTERNET (upload et download)
- Mesurer le débit Maximal atteignable
- Test de résilience des liens

## Architecture
L’architecture sera composée de clients iPerf déjà configurés qui seront installés sur les Datacenters à savoir :
- Yopougon
- Bingerville
- Yakro

Le serveur iPerf sera quant à lui déployé sur le site de Marcory.

## Prérequis
| Responsable | Action |
| :--- | :--- |
| **KONDRO NETWORKS** | Fourniture du PC client iPerf |
| **KONDRO NETWORKS** | Fourniture du serveur iPerf |
| **MTN CI** | Fourniture des adresses IP des clients et serveur iPerf |
| **MTN CI** | Ouverture du port TCP/UDP 5201 entre les clients et le serveur |
| **MTN CI** | Présence sur les sites |
| **MTN CI** | Autorisation ICMP bidirectionnelle entre les clients et le serveur |
| **MTN CI** | Accès à Internet pour les clients et le serveur |
| **MTN CI** | Présence physique de MTN sur site |
| **MTN CI** | Fournir la valeur MTU utilisée sur le réseau IP/MPLS |

## Tests de performance
Les PC clients et Serveurs ont été déjà installés.

### Configuration côté serveur
```bash
iperf3 -s
```

### Configuration côté client

#### 3.1. Mesure du débit nominal entre Datacenter et internet

**Mesure client vers serveur en utilisant TCP :**
```bash
# Exemples de cas pratique avec cette commande :

# 1. Mesure du débit TCP standard entre un client et le serveur pendant 30 minutes avec 10 flux parallèles :
iperf3.exe -c 192.168.1.10 -t 1800 -i 3 -f G -P 10

# 2. Mesure du débit TCP avec ajustement de la taille de fenêtre TCP à 1Mo (1048576 octets) :
iperf3.exe -c 192.168.1.10 -t 1800 -i 3 -f G -P 10 -w 1048576

# 3. Mesure sur un autre serveur cible avec une fenêtre TCP de 512Ko
iperf3.exe -c 10.0.0.50 -t 1800 -i 3 -f G -P 10 -w 524288

# 4. Exécution avec adaptation de la fréquence d’affichage des résultats toutes les 10 secondes
iperf3.exe -c 192.168.100.5 -t 1800 -i 10 -f G -P 10

# 5. Mesure standard mais en affichant la bande passante en Mégaoctets
iperf3.exe -c 192.168.1.10 -t 1800 -i 3 -f M -P 10
```

**Explication des options :**
- `-t` : Faire le test pendant 30 minutes (1800 secondes)
- `-i` : Afficher les statistiques toutes les 3 secondes
- `-f` : Afficher la bande passante en Gigaoctets
- `-P` : Lancer 10 streams en parallèle
- `-w` : Taille de la fenêtre TCP

**Mesure serveur vers client en utilisant TCP :**
```bash
iperf3.exe -c @IP_Serveur -R
```

**Mesure client vers serveur en utilisant UDP :**
```bash
# Pourquoi la commande prend autant de temps ?
# La commande suivante lance un test iPerf3 d'une durée de 30 minutes (=1800 secondes) :
# Exemples pratiques de la commande iPerf3 en UDP :

# 1. Test de bande passante UDP à 100 Mbit/s pendant 30 minutes :
iperf3.exe -c 192.168.1.10 -u -b 100M -t 1800 -i 3 -f G -P 10

# 2. Test de bande passante UDP à 1 Gbit/s pendant 10 minutes avec 5 flux :
iperf3.exe -c 192.168.1.10 -u -b 1G -t 600 -i 3 -f G -P 5

# 3. Test UDP à 500 Mbit/s pendant 5 minutes :
iperf3.exe -c 10.0.0.50 -u -b 500M -t 300 -i 3 -f G -P 10
# → Le paramètre `-t 1800` (soit 1800 secondes) configure la durée du test à 30 minutes, d'où le temps d'exécution long.
```

**Explication des options :**
- `-t` : Faire le test pendant 30 minutes
- `-i` : Afficher les statistiques toutes les 3 secondes
- `-f` : Afficher la bande passante en Gigaoctets
- `-P` : Lancer 10 streams en parallèle
- `-u` : Mode UDP
- `-b` : Débit de la liaison

**Activation mesure UDP sur le serveur :**
```bash
iperf3 -s
```
> [!NOTE]
> `iperf3` gère TCP et UDP sur le même port par défaut.

##### Dépannage – erreur « unable to read from stream socket: Resource temporarily unavailable »

Cette erreur apparaît typiquement en UDP (`-u`) lors de la phase initiale de création des streams.

- **Perte ou blocage des premiers paquets UDP** : le client envoie un paquet au serveur (port 5201) qui doit répondre ; si ce paquet ou la réponse est perdu (latence, congestion, pare‑feu), iperf3 timeout et affiche cette erreur.
- **Surcharge réseau / paramètres trop agressifs** : combiner `-P` élevé (plusieurs flux parallèles) et un `-b` très important (500M, 1G, 1T) crée une « tempête » UDP qui sature rapidement buffers et équipements.
- **Mauvaise configuration réseau** : serveur iperf3 non démarré, port UDP 5201 filtré, NAT/pare‑feu qui laisse passer ICMP mais pas UDP.

**Vérifications rapides :**
- S’assurer que le serveur tourne bien : `iperf3 -s` sur l’IP cible et que le port 5201/UDP est ouvert.
- Tester la connectivité basique : `ping @IP_Serveur` puis, côté serveur, un simple `tcpdump -i any udp port 5201` pour vérifier la réception des paquets.
- Tester en TCP (sans `-u`) pour confirmer que la connectivité de base est correcte.

**Recette de commande « sûre » pour démarrer :**
```bash
iperf3.exe -c @IP_Serveur -u -b 10M -P 1 --set-mss 1200 -t 30
```
- `-b 10M` : débit modéré, peu de risque de saturation.
- `-P 1` : un seul flux pour simplifier le diagnostic.
- `--set-mss 1200` : taille de paquet réduite pour limiter la fragmentation.

Si ce test fonctionne, augmenter progressivement :
1. Augmenter d’abord `-b` (ex. 50M, 100M, 200M) avec `-P 1`.
2. Ensuite seulement augmenter `-P` (2, 3, 5, 10) en surveillant CPU, buffers et pertes paquets.
3. Si l’erreur réapparaît, revenir au dernier couple (`-b`, `-P`) stable et considérer qu’on a atteint la limite raisonnable de la liaison/équipement.

#### 3.2. Détermination de la valeur du MTU

> **La commande `iperf3.exe -c @IP_Serveur -m` n'est pas valable pour déterminer la valeur du MTU.**
>
> L’option `-m` dans iPerf3 affiche les paramètres du socket mais ne mesure ni ne détermine la valeur du MTU.  
> Pour déterminer le MTU, il faut utiliser la commande `ping` avec l'option `-f` (ne pas fragmenter) et ajuster la taille du paquet jusqu'à ce qu'aucune fragmentation ne se produise. Par exemple :
>
> ```bash
> ping @IP_Serveur -f -l 1472
> ```
> 
> (où 1472 = 1500 - 28 octets d'en-tête IP/ICMP)
>
```

#### 3.3. Détermination du débit maximal sur les liaisons Internet et MPLS
```bash
# Côté serveur (si nécessaire avec -R)
iperf3 -c @IP_Serveur -u -b 1T -R 
# Côté client
iperf3 -c @IP_Serveur -u -b 1T
```

> [!IMPORTANT]
> - L’adresse **IP Serveur** représente l’IP du serveur iPerf côté réseau local pour les mesures entre les Datacenters.
> - L’adresse **IP Serveur** représente l’IP du serveur iPerf côté internet pour les mesures sur internet.

**DURÉE DES ACTIONS :**
- Deux (2) heures par paire de site (par exemple Marcory et Bingerville constituent une paire de site).

## Tests de résilience
Les tests de résilience consisteront à exécuter un ping continu (ping long) vers le Datacenter secondaire (Bingerville, Yopougon ou Yamoussoukro), puis à effectuer les actions suivantes :
1. **Couper la liaison primaire**, observer le comportement du trafic, et vérifier le basculement automatique (failover) vers la liaison secondaire.
2. **Rétablir la liaison primaire**, observer le retour automatique du trafic (fallback), et vérifier la stabilité et la continuité du ping.
3. **Couper la liaison secondaire**, observer le maintien du service via la liaison primaire.
4. **Rétablir les deux liaisons**, confirmer la restauration complète de la redondance, et valider les temps de convergence et l’absence de perte excessive de paquets.

> [!NOTE]
> Pour les tests de redondance, MTN déterminera les fenêtres de maintenance et nous fournirons les différents segments pour les tests. Nous fournirons les différents segments après reconstitution de l’architecture.


