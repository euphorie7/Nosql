# TP3 –  Partitionnement (Sharding) MongoDB

## Informations générales

- **Encadré par** : M. YOUCEF  
- **Réalisé par** : LAOUNI Hamza  
- **Spécialité** : Informatique  
- **Niveau** : 3ᵉ année Cycle d’Ingénieur  
- **Année universitaire** : 2025–2026

---


## 1. Introduction

Ce travail pratique est consacré à l’étude du **sharding (partitionnement)** dans MongoDB.  
Contrairement à la réplication, qui vise principalement la **haute disponibilité**, le sharding permet de **répartir les données sur plusieurs serveurs** afin de gérer de très grands volumes de données et d’améliorer les performances.

L’objectif de ce TP est de mettre en place un **cluster MongoDB shardé à l’aide de Docker**, puis de comprendre le rôle de chaque composant ainsi que les mécanismes de distribution et d’équilibrage des données.

---

## 2. Architecture d’un cluster shardé

L’architecture d’un cluster MongoDB shardé repose sur trois composants principaux :

- **Config Servers** : stockent les métadonnées du cluster (répartition des chunks, configuration globale).
- **Shards** : stockent les données effectives, chaque shard contenant une partie du jeu de données.
- **Routeur mongos** : point d’entrée des applications, il redirige les requêtes vers les shards appropriés.

Cette architecture permet une **montée en charge horizontale** du stockage et des performances.

---

## 3. Mise en place du cluster shardé avec Docker

### 3.1 Préparation (réseau et volumes)

```bash
docker rm -f mongos configsvr shard1 shard2 mongodb 2>NUL
docker network rm mongo-shard-net 2>NUL
docker volume rm configsvrdb shard1db shard2db 2>NUL

docker network create mongo-shard-net
docker volume create configsvrdb
docker volume create shard1db
docker volume create shard2db
```

---

### 3.2 Démarrage du Config Server

```bash
docker run -it --rm --name configsvr --net mongo-shard-net -p 27019:27019 \
  -v configsvrdb:/data/db mongo:4.2.24 \
  mongod --configsvr --replSet replicaconfig --port 27019 --bind_ip_all
```

### Initialisation du Replica Set du Config Server

```bash
docker exec -it configsvr mongo --port 27019
```

```js
rs.initiate({
  _id: "replicaconfig",
  configsvr: true,
  members: [{ _id: 0, host: "configsvr:27019" }]
})
rs.status()
exit
```

---

### 3.3 Démarrage et initialisation des shards

#### Shard 1

```bash
docker run -it --rm --name shard1 --net mongo-shard-net -p 20004:20004 \
  -v shard1db:/data/db mongo:4.2.24 \
  mongod --shardsvr --replSet replicashard1 --port 20004 --bind_ip_all
```

```bash
docker exec -it shard1 mongo --port 20004
```

```js
rs.initiate({
  _id: "replicashard1",
  members: [{ _id: 0, host: "shard1:20004" }]
})
rs.status()
exit
```

#### Shard 2

```bash
docker run -it --rm --name shard2 --net mongo-shard-net -p 20005:20005 \
  -v shard2db:/data/db mongo:4.2.24 \
  mongod --shardsvr --replSet replicashard2 --port 20005 --bind_ip_all
```

```bash
docker exec -it shard2 mongo --port 20005
```

```js
rs.initiate({
  _id: "replicashard2",
  members: [{ _id: 0, host: "shard2:20005" }]
})
rs.status()
exit
```

---

### 3.4 Démarrage du routeur mongos

```bash
docker run -it --rm --name mongos --net mongo-shard-net -p 27017:27017 \
  mongo:4.2.24 mongos --configdb replicaconfig/configsvr:27019 --bind_ip_all
```

---

## 4. Configuration du sharding

### Connexion au routeur

```bash
docker exec -it mongos mongo --port 27017
```

### Ajout des shards

```js
sh.addShard("replicashard1/shard1:20004")
sh.addShard("replicashard2/shard2:20005")
sh.status()
```

### Activation du sharding

```js
sh.enableSharding("mabasefilms")

use mabasefilms
db.films.createIndex({ titre: 1 })

sh.shardCollection("mabasefilms.films", { titre: 1 })
sh.status()
```

---

## 5. Questions / Réponses sur le sharding MongoDB

### Question 1 : Qu’est-ce que le sharding ?
Le sharding est un **partitionnement horizontal** qui répartit les données d’une collection sur plusieurs serveurs appelés shards.

### Question 2 : Pourquoi utiliser le sharding ?
Pour gérer de très grands volumes de données et répartir la charge de lecture et d’écriture.

### Question 3 : Différence entre sharding et réplication
La réplication assure la **tolérance aux pannes**, le sharding assure la **scalabilité**.

### Question 4 : Composants d’un cluster shardé
Shards, Config Servers et routeur mongos.

### Question 5 : Rôle des config servers
Ils stockent les métadonnées du cluster shardé.

### Question 6 : Rôle du mongos
Il route les requêtes vers les shards concernés et agrège les résultats.

### Question 7 : Comment MongoDB choisit le shard ?
À partir de la **valeur de la shard key** du document.

### Question 8 : Qu’est-ce qu’une clé de sharding ?
Champ utilisé pour répartir les données entre shards.

### Question 9 : Critères d’une bonne shard key
- Forte cardinalité  
- Bonne distribution  
- Non monotone  
- Fréquemment utilisée dans les requêtes  

### Question 10 : Qu’est-ce qu’un chunk ?
Un chunk est une plage de valeurs de la shard key stockée sur un shard.

### Question 11 : Fonctionnement du splitting
MongoDB découpe un chunk trop volumineux en chunks plus petits.

### Question 12 : Rôle du balancer
Il équilibre les chunks entre shards.

### Question 13 : Déplacement des chunks
Le balancer migre des chunks d’un shard chargé vers un shard moins chargé.

### Question 14 : Hot shard
Shard recevant une charge excessive à cause d’une mauvaise shard key.

### Question 15 : Problèmes d’une clé monotone
Concentration des écritures sur un seul shard.

### Question 16 : Activation du sharding
Avec `sh.enableSharding()` puis `sh.shardCollection()`.

### Question 17 : Ajout d’un shard
```js
sh.addShard("replica/host:port")
```

### Question 18 : Vérification de l’état du cluster
```js
sh.status()
```

### Question 19 : Hashed sharding
Permet une distribution uniforme des écritures.

### Question 20 : Ranged sharding
Adapté aux requêtes par intervalle.

### Question 21 : Requêtes multi-shards
Le mongos effectue un *scatter-gather*.

### Question 22 : Optimisation des performances
Inclure la shard key dans les requêtes et créer des index adaptés.

### Question 23 : Indisponibilité d’un shard
Les données de ce shard deviennent inaccessibles.

### Question 24 : Migration d’une collection existante
Création d’index, activation du sharding, puis shardCollection.

### Question 25 : Outils de diagnostic
- `sh.status()`
- `db.collection.getShardDistribution()`
- Logs MongoDB
- `config.chunks`

---

## 6. Conclusion

Ce TP a permis de :
- Déployer un cluster MongoDB shardé avec Docker
- Comprendre l’architecture et les composants du sharding
- Configurer une base et une collection shardées
- Observer la distribution des données et le rôle du balancer

Le sharding est une solution essentielle pour assurer la **scalabilité horizontale** des bases MongoDB dans les systèmes modernes.

