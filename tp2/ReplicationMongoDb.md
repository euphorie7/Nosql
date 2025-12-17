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

## Questions / Réponses sur la réplication MongoDB

### Question 1 : Qu’est-ce qu’un Replica Set ?
Un Replica Set est un ensemble d’instances MongoDB qui conservent une copie des mêmes données.  
Les écritures sont centralisées sur un Primary et répliquées sur les Secondaries via l’oplog, garantissant la haute disponibilité.

### Question 2 : Quel est le rôle du Primary ?
Le Primary est le seul nœud autorisé à accepter les écritures.  
Il enregistre les opérations dans l’oplog afin qu’elles soient rejouées par les Secondaries.

### Question 3 : Rôle des Secondaries
Les Secondaries répliquent les données depuis l’oplog du Primary, peuvent servir les lectures et sont capables de devenir Primary lors d’une élection.

### Question 4 : Pourquoi les écritures sont interdites sur un Secondary ?
Autoriser des écritures multiples provoquerait des conflits.  
MongoDB impose un point d’écriture unique pour maintenir la cohérence.

### Question 5 : Qu’est-ce que la cohérence forte ?
La cohérence forte garantit que toute lecture reflète les écritures validées, grâce à :
- `readPreference: primary`
- `writeConcern: majority`
- `readConcern: majority`

### Question 6 : Différence entre `readPreference` primary et secondary
- **Primary** : données à jour, mais charge concentrée.
- **Secondary** : réduit la charge, mais légère latence.

### Question 7 : Quand lire sur un Secondary ?
- Pour l’analyse de données
- Le reporting
- Quand la fraîcheur des données n’est pas critique

### Question 8 : Initialisation d’un Replica Set
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

### Question 10 : Vérifier l’état du Replica Set
```javascript
rs.status()
```

### Question 11 : Identifier le rôle d’un nœud
Les commandes `db.isMaster()` ou `rs.status()` indiquent le rôle (Primary, Secondary, Arbiter).

### Question 12 : Forcer une bascule du Primary
```javascript
rs.stepDown(60)
```

### Question 13 : Qu’est-ce qu’un Arbitre ?
Un Arbitre ne stocke aucune donnée et participe uniquement aux votes lors des élections.

### Question 14 : Secondary avec délai de réplication
```javascript
rs.add({
  host: "mongo-delayed:27017",
  priority: 0,
  hidden: true,
  slaveDelay: 120
})
```

### Question 15 : Absence de majorité
Sans majorité, aucun Primary ne peut être élu. Les écritures sont alors impossibles.

### Question 16 : Critères de choix du Primary
MongoDB privilégie le nœud appartenant à la majorité, ayant la priorité la plus élevée et le plus à jour.

### Question 17 : Qu’est-ce qu’une élection ?
Une élection est un processus automatique permettant de désigner un nouveau Primary par vote majoritaire.

### Question 18 : Auto-dégradation
Un Primary se rétrograde en Secondary lorsqu’il perd la majorité afin d’éviter le split-brain.

### Question 19 : Pourquoi un nombre impair de nœuds ?
Un nombre impair facilite l’obtention d’une majorité claire lors des élections.

### Question 20 : Effets d’une partition réseau
Seule la partition conservant la majorité peut maintenir ou élire un Primary.

### Question 21 : Primary injoignable avec un Arbitre
Le Secondary et l’Arbitre forment la majorité, permettant l’élection d’un nouveau Primary.

### Question 22 : Utilité d’un `slaveDelay`
Il permet de revenir à un état antérieur en cas d’erreur humaine récente.

### Question 23 : Lecture toujours à jour
Il est recommandé d’utiliser :
- `writeConcern: majority`
- `readPreference: primary`
- `readConcern: majority`

### Question 24 : Confirmer une écriture sur deux nœuds
Utiliser :
```javascript
{ writeConcern: { w: 2 } }
// ou
{ writeConcern: "majority" }
```

### Question 25 : Lecture obsolète sur un Secondary
Le retard de réplication explique l’obsolescence. Lire sur le Primary évite ce problème.

### Question 26 : Identifier le Primary
```javascript
rs.status().members.map(m => ({ m.name, m.stateStr }))
```

### Question 27 : Bascule manuelle contrôlée
Vérifier la réplication puis utiliser :
```javascript
rs.stepDown()
```


### Question 28 : Ajouter un Secondary en production
Démarrer le nœud, puis l’ajouter avec :
```javascript
rs.add()
```

### Question 29 : Retirer un nœud
```javascript
rs.remove("mongo4:27017")
```

### Question 30 : Nœud caché
Un nœud caché n’est pas utilisé pour les lectures applicatives et sert aux tâches spécifiques (ex : sauvegarde, `slaveDelay`, etc.).

### Question 31 : Modifier la priorité
Le nœud ayant la priorité la plus élevée sera favorisé lors des élections.

### Question 32 : Vérifier le retard de réplication
```javascript
rs.printSlaveReplicationInfo()
```

### Question 33 : Rôle de `rs.freeze()`
Empêche temporairement un nœud de devenir Primary.

### Question 34 : Redémarrage sans perte
La configuration est conservée tant que le répertoire de données est maintenu.

### Question 35 : Surveillance
Les commandes `rs.status()` et les logs MongoDB permettent le suivi de l’état du Replica Set.

### Question 36 : (optionnelle) Logs
La consultation des logs aide à diagnostiquer les problèmes de réplication.

### Question 37 : Pourquoi un Arbitre ne stocke pas de données ?
Il sert uniquement aux votes afin de réduire les coûts et la complexité.

### Question 38 : Vérifier la latence de réplication
Utiliser :
```javascript
rs.printSlaveReplicationInfo()
```

### Question 39 : Afficher le retard de réplication
Commande :
```javascript
rs.printSlaveReplicationInfo()
```

### Question 40 : Réplication synchrone vs asynchrone
MongoDB utilise une réplication **asynchrone** avec des **garanties configurables**.

### Question 41 : Modifier la configuration à chaud
La commande suivante permet une modification sans redémarrage :
```javascript
rs.reconfig()
```

### Question 42 : Secondary très en retard
Il ne peut pas devenir Primary et peut nécessiter une **resynchronisation complète**.

### Question 43 : Gestion des conflits
Les conflits sont évités car **seul le Primary accepte les écritures**.

### Question 44 : Plusieurs Primary possibles ?
**Non**, le mécanisme de **majorité empêche** ce scénario.

### Question 45 : Pourquoi ne pas écrire sur un Secondary ?
Les écritures seraient **écrasées** par la réplication du Primary.

### Question 46 : Effets d’un réseau instable
Un réseau instable provoque :
- des **élections fréquentes**
- des **erreurs d’écriture**
- un **retard accru** de réplication

---

## Conclusion

Ce TP a permis de mettre en œuvre un **Replica Set MongoDB**, de comprendre les mécanismes :
- d’**élection**
- de **réplication**
- de **tolérance aux pannes**

Le Replica Set constitue une solution essentielle pour garantir la **disponibilité** et la **cohérence des données** dans les systèmes modernes.

