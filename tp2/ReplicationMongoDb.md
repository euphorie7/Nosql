# TP3 – Réplication MongoDB

## Informations générales

- **Encadré par** : M. YOUCEF  
- **Réalisé par** : LAOUNI Hamza  
- **Spécialité** : Informatique  
- **Niveau** : 3ᵉ année Cycle d’Ingénieur  
- **Année universitaire** : 2025–2026

---

## Introduction

Ce travail pratique est consacré à l’étude de la réplication dans MongoDB à travers la mise en place d’un *Replica Set*.  
L’objectif est de comprendre le rôle des différents nœuds, le mécanisme d’élection du Primary, la propagation des opérations ainsi que la tolérance aux pannes.

Un Replica Set regroupe plusieurs instances MongoDB partageant les mêmes données.  
Cette architecture permet d’assurer la haute disponibilité, la continuité de service et l’intégrité des données même en cas de défaillance d’un serveur.

---

## 2. Initialisation du Replica Set

Nous avons utilisé Docker pour créer trois instances MongoDB faisant partie du même réseau.

### 2.1 Création du réseau
```bash
docker network create mycluster
```

### 2.2 Lancement des conteneurs
```bash
docker run -d --name mongo1 --net mycluster -p 27017:27017 mongo --replSet
docker run -d --name mongo2 --net mycluster -p 27018:27017 mongo --replSet
docker run -d --name mongo3 --net mycluster -p 27019:27017 mongo --replSet
```

### 2.3 Initialisation du cluster
Depuis `mongo1` :
```bash
docker exec -it mongo1 mongosh
```
Puis :
```javascript
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "XXXX:27017" },
    { _id: 1, host: "XXXX:27018" },
    { _id: 2, host: "XXXX:27019" }
  ]
})
```
Vérification :
```javascript
rs.status()
```

---

## 3. Fonctionnement général

Un Replica Set est composé de :
- un **Primary** qui accepte toutes les écritures,
- des **Secondaries** qui répliquent l’oplog,
- un **Arbitre** éventuel pour les votes.

S’il y a une défaillance du Primary, une **élection automatique** permet à un Secondary de le remplacer (si une majorité est disponible).

### Avantages :
- Haute disponibilité
- Redondance automatique
- Répartition de la charge de lecture

---

## 4. Connexion au Replica Set

### 4.1 Connexion directe
```bash
mongosh "mongodb://IP:27017/?directConnection=true"
```

### 4.2 Connexion au cluster complet
```bash
mongosh "mongodb://IP:27017,IP:27018,IP:27019/?replicaSet=rs0"
```

MongoDB détecte automatiquement le Primary.

---

## 5. Questions / Réponses sur la réplication MongoDB

### Question 1 : Qu’est-ce qu’un Replica Set ?
Un ensemble d’instances MongoDB répliquant les mêmes données via un Primary et des Secondaries.

### Question 2 : Quel est le rôle du Primary ?
Reçoit toutes les écritures et alimente l’oplog.

### Question 3 : Rôle des Secondaries
Rejouent l’oplog du Primary et peuvent être promus en Primary.

### Question 4 : Pourquoi les écritures sont interdites sur un Secondary ?
Pour éviter les conflits. Un seul point d’écriture est autorisé.

### Question 5 : Qu’est-ce que la cohérence forte ?
- `readPreference: primary`  
- `writeConcern: majority`  
- `readConcern: majority`

### Question 6 : Différence entre Primary et Secondary
Primary = données à jour.  
Secondary = lecture possible avec latence.

### Question 7 : Quand lire sur un Secondary ?
Quand la fraîcheur n’est pas critique (reporting, analyse).

### Question 8 : Initialisation
```javascript
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "host:27017" },
    { _id: 1, host: "host:27018" },
    { _id: 2, host: "host:27019" }
  ]
})
```

### Question 9 : Ajouter un nœud
```javascript
rs.add("host:27020")
```

### Question 10 : Vérifier l’état
```javascript
rs.status()
```

### Question 11 : Identifier le rôle d’un nœud
```javascript
db.isMaster()
```
ou  
```javascript
rs.status()
```

### Question 12 : Forcer une bascule
```javascript
rs.stepDown(60)
```

### Question 13 : Qu’est-ce qu’un Arbitre ?
Nœud qui ne stocke pas de données et participe au vote.

### Question 14 : Secondary avec délai
```javascript
rs.add({
  host: "mongo-delayed:27017",
  priority: 0,
  hidden: true,
  slaveDelay: 120
})
```

### Question 15 : Absence de majorité
Aucun Primary ne peut être élu → écritures bloquées.

### Question 16 : Critères d’élection
Majorité + priorité la plus haute + données les plus à jour.

### Question 17 : Qu’est-ce qu’une élection ?
Vote entre les membres pour choisir un nouveau Primary.

### Question 18 : Auto-dégradation
Un Primary sans majorité se rétrograde en Secondary.

### Question 19 : Pourquoi nombre impair ?
Pour obtenir une majorité claire.

### Question 20 : Partition réseau
Seule la partition majoritaire peut élire un Primary.

### Question 21 : Arbitre + Secondary
Majorité suffisante pour élire un Primary.

### Question 22 : Utilité d’un `slaveDelay`
Protection contre erreurs récentes (retour en arrière possible).

### Question 23 : Lecture à jour
Utiliser :
- `writeConcern: majority`
- `readPreference: primary`
- `readConcern: majority`

### Question 24 : Écriture confirmée sur 2 nœuds
```javascript
{ writeConcern: { w: 2 } }
```

### Question 25 : Lecture obsolète
Lire sur un Primary pour des données à jour.

### Question 26 : Identifier le Primary
```javascript
rs.status().members.map(m => ({ m.name, m.stateStr }))
```

### Question 27 : Bascule manuelle
```javascript
rs.stepDown()
```

### Question 28 : Ajouter un Secondary en production
```javascript
rs.add("host:port")
```

### Question 29 : Retirer un nœud
```javascript
rs.remove("mongo4:27017")
```

### Question 30 : Nœud caché
Utile pour les backups et les lectures différées.

### Question 31 : Modifier la priorité
Permet de favoriser certains nœuds pour devenir Primary.

### Question 32 : Retard de réplication
```javascript
rs.printSlaveReplicationInfo()
```

### Question 33 : `rs.freeze()`
Empêche un nœud de devenir Primary temporairement.

### Question 34 : Redémarrage sans perte
Configuration conservée si le répertoire de données est intact.

### Question 35 : Surveillance
- `rs.status()`
- logs MongoDB

### Question 36 : Logs
Analyse des erreurs et de la réplication.

### Question 37 : Pourquoi l’Arbitre ne stocke rien ?
Réduction des ressources utilisées. Il sert uniquement au vote.

### Question 38 : Latence de réplication
```javascript
rs.printSlaveReplicationInfo()
```

### Question 39 : Afficher le retard
```javascript
rs.printSlaveReplicationInfo()
```

### Question 40 : Synchrone vs asynchrone
MongoDB utilise la **réplication asynchrone** avec **writeConcern configurable**.

### Question 41 : Modifier à chaud
```javascript
rs.reconfig()
```

### Question 42 : Secondary très en retard
Ne peut pas être promu Primary, resynchronisation possible.

### Question 43 : Gestion des conflits
Aucun conflit car un seul point d’écriture : le Primary.

### Question 44 : Plusieurs Primary ?
Non, la **majorité empêche** cette situation.

### Question 45 : Pourquoi ne pas écrire sur un Secondary ?
Les données seraient **écrasées** par la réplication.

### Question 46 : Réseau instable
- Élections fréquentes
- Échecs d’écriture
- Retard accru

---

## 6. Graphiques (à insérer si hébergés)

### 📊 Retard de réplication
![Retard de réplication](replication_lag.png)

### ⚡ Temps d’élection
![Temps d'élection](election_time.png)

---

## 7. Conclusion

Ce TP nous a permis de :
- Créer un cluster MongoDB avec Docker
- Configurer un Replica Set
- Comprendre le mécanisme d’élection
- Tester la bascule automatique en cas de panne

La réplication dans MongoDB est essentielle pour garantir la **haute disponibilité** et la **cohérence des données** dans un environnement distribué.  
Le Replica Set constitue une solution fiable, automatique et bien adaptée aux systèmes modernes.


