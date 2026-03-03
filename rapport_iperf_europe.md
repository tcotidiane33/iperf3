# Rapport de Tests iperf3 - Serveurs Publics Europe

## 1. Résumé des tests
Ce rapport présente les résultats des mesures de performance réseau effectuées le 3 mars 2026 depuis votre machine vers des serveurs iperf3 publics situés en Europe.

| Serveur | Latence Moyenne | Débit Max Observé | État du Service |
| :--- | :--- | :--- | :--- |
| **speedtest.iway.ch** | 143.7 ms | N/A | Hors service (Timeout) |
| **89.187.165.1** | 124.6 ms | 53.7 Mbits/sec | Instable (Congestion rapide) |

---

## 2. Analyse détaillée - 89.187.165.1

### Test TCP Standard (Upload)
```text
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-1.00   sec  3.88 MBytes  32.4 Mbits/sec                  
[  5]   1.00-2.00   sec  6.38 MBytes  53.7 Mbits/sec                  
[  5]   2.00-3.00   sec   384 KBytes  3.14 Mbits/sec                  
[  5]   3.00-4.00   sec  0.00 Bytes  0.00 bits/sec                  
```

**Interprétation :**
- **Démarrage (0-2s)** : Le débit grimpe jusqu'à ~54 Mbps. Cela correspond probablement à la capacité réelle du lien ou au burst TCP.
- **Effondrement (2-4s)** : Le débit chute brutalement à 0 bps. 
- **Causes possibles** :
    1. **Congestion du serveur** : Le serveur public est partagé. S'il reçoit trop de trafic, il coupe les flux.
    2. **Throttling ISP** : Votre fournisseur d'accès ou celui du serveur détecte un flux iperf3 volumineux et le bride (protection anti-DoS).
    3. **Bufferbloat** : Les buffers réseau saturent, provoquant une perte massive de paquets et une fenêtre TCP qui tombe à zéro.

---

## 3. Analyse détaillée - speedtest.iway.ch

**Résultat** : `Operation timed out`
**Interprétation** : 
Bien que le serveur réponde au `ping` (ICMP), il refuse les connexions sur le port standard `5201`. Cela signifie que le service iperf3 n'est pas démarré, ou qu'un pare-feu bloque le trafic applicatif. 

---

## 4. Concepts clés pour mieux comprendre

### Latence vs Débit
La latence (~125ms) est élevée car les serveurs sont en Europe. Une latence élevée limite la vitesse de montée en charge du TCP (Slow Start). Plus la latence est grande, plus le protocole TCP met du temps à confirmer la réception des paquets (ACK), ce qui ralentit le débit global sur un seul flux.

### Serveurs Publics : Pourquoi est-ce difficile ?
Les serveurs iperf3 publics sont souvent limités à **un seul utilisateur à la fois**. Si vous voyez l'erreur `"server is busy"`, cela signifie qu'une autre personne dans le monde effectue un test au même moment. 

### Utilisation Recommandée
Pour des tests fiables, il est préférable d'avoir son propre serveur (ex: au bureau ou sur un VPS) pour éviter les interférences avec d'autres utilisateurs.

---
*Rapport généré par Antigravity.*
