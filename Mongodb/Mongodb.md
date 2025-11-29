# Guide MongoDB – TP lesfilms

## Introduction
Ce document regroupe un ensemble de requêtes essentielles pour analyser et parcourir la base de données `lesfilms`. Il sert de support structuré pour apprendre à interroger efficacement la collection, filtrer des résultats, projeter des champs précis et extraire des informations ciblées.

## 1. Vérifier l’importation
```js
db.lesfilms.count()
```

## 2. Observer la structure d’un document
```js
db.lesfilms.findOne()
```

## 3. Films d’action
```js
db.lesfilms.find({ genre: "Action" })
```

## 4. Nombre de films d’action
```js
db.lesfilms.count({ genre: "Action" })
```

## 5. Films d’action produits en France
```js
db.lesfilms.find({ genre: "Action", pays: "France" })
```

## 6. Films d’action français en 1963
```js
db.lesfilms.find({ genre: "Action", pays: "France", annee: 1963 })
```

## 7. Sans le champ grades
```js
db.lesfilms.find({ genre: "Action", pays: "France" }, { grades: 0 })
```

## 8. Sans _id
```js
db.lesfilms.find({ genre: "Action", pays: "France" }, { _id: 0 })
```

## 9. Afficher uniquement titres et notes
```js
db.lesfilms.find({}, { _id: 0, titre: 1, grades: 1 })
```

## 10. Films ayant une note > 10
```js
db.lesfilms.find({ "grades.note": { $gt: 10 } })
```

## 11. Films dont toutes les notes sont > 10
```js
db.lesfilms.find({ grades: { $not: { $elemMatch: { note: { $lte: 10 } } } } })
```

## 12. Genres distincts
```js
db.lesfilms.distinct("genre")
```

## 13. Différents grades
```js
db.lesfilms.distinct("grades")
```

## 14. Films contenant artistes [4,18,11]
```js
db.lesfilms.find({ acteurs: { $in: ["artist:4","artist:18","artist:11"] } })
```

## 15. Films sans résumé
```js
db.lesfilms.find({ resume: { $exists: false } })
```

## 16. Films avec Leonardo DiCaprio en 1997
```js
db.lesfilms.find({ acteurs: "Leonardo DiCaprio", annee: 1997 })
```

## 17. Films avec DiCaprio ou en 1997
```js
db.lesfilms.find({ $or: [ { acteurs: "Leonardo DiCaprio" }, { annee: 1997 } ] })
```
## Conclusion
L’ensemble de ces requêtes offre une base solide pour parcourir et analyser la collection `lesfilms`. Elles permettent à la fois de filtrer des données, d’afficher sélectivement certains champs, d’examiner la structure des documents et de répondre à des besoins de recherche précis. Ce guide constitue ainsi un support clair pour maîtriser progressivement l’interrogation d’une base MongoDB.

