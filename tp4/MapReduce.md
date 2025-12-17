# TP4 – Analyse de Données avec MapReduce sur MongoDB

## Informations générales

- **Encadré par** : M. YOUCEF  
- **Réalisé par** : LAOUNI Hamza  
- **Spécialité** : Informatique  
- **Niveau** : 3ᵉ année Cycle d’Ingénieur  
- **Année universitaire** : 2025–2026

---


## 1. Introduction

Ce travail pratique vise à introduire le **paradigme MapReduce** dans le contexte de MongoDB.  
MapReduce est un modèle de calcul distribué permettant de traiter efficacement de **grands volumes de données** en deux étapes :

- **Map** : génère des paires clé–valeur à partir de chaque document.
- **Reduce** : agrège les valeurs associées à une même clé.

Nous utiliserons une **collection de films** contenant le titre, les genres, les réalisateurs, les acteurs, l’année et les notes.  
L’objectif est de réaliser diverses analyses en appliquant MapReduce pour obtenir des statistiques et des regroupements pertinents.

---

## 2. Exercices MapReduce

### 1. Nombre total de films
```js
db.movies.mapReduce(
  function () { emit("total", 1); },
  function (key, values) { return Array.sum(values); },
  { out: "totalFilms" }
);
```

### 2. Nombre de films par genre
```js
db.movies.mapReduce(
  function () {
    if (this.genres) {
      this.genres.forEach(g => emit(g, 1));
    }
  },
  function (key, values) { return Array.sum(values); },
  { out: "filmsParGenre" }
);
```

### 3. Nombre de films par réalisateur
```js
db.movies.mapReduce(
  function () {
    if (this.director) {
      emit(this.director, 1);
    }
  },
  function (key, values) { return Array.sum(values); },
  { out: "filmsParRealisateur" }
);
```

### 4. Nombre d’acteurs uniques
```js
db.movies.mapReduce(
  function () {
    if (this.cast) {
      this.cast.forEach(a => emit(a, 1));
    }
  },
  function (key, values) { return 1; },
  { out: "acteursUniques" }
);
```

### 5. Nombre de films par année
```js
db.movies.mapReduce(
  function () {
    if (this.year) {
      emit(this.year, 1);
    }
  },
  function (key, values) { return Array.sum(values); },
  { out: "filmsParAnnee" }
);
```

### 6. Note moyenne par film
```js
db.movies.mapReduce(
  function () {
    if (this.grades) {
      const avg = this.grades.reduce((a, b) => a + b.score, 0) / this.grades.length;
      emit(this.title, avg);
    }
  },
  function (key, values) { return Array.sum(values) / values.length; },
  { out: "noteMoyenneParFilm" }
);
```

### 7. Note moyenne par genre
```js
db.movies.mapReduce(
  function () {
    if (this.grades && this.genres) {
      const avg = this.grades.reduce((a, b) => a + b.score, 0) / this.grades.length;
      this.genres.forEach(g => emit(g, avg));
    }
  },
  function (key, values) { return Array.sum(values) / values.length; },
  { out: "noteMoyenneParGenre" }
);
```

### 8. Note moyenne par réalisateur
```js
db.movies.mapReduce(
  function () {
    if (this.director && this.grades) {
      const avg = this.grades.reduce((a, b) => a + b.score, 0) / this.grades.length;
      emit(this.director, avg);
    }
  },
  function (key, values) { return Array.sum(values) / values.length; },
  { out: "noteMoyenneParRealisateur" }
);
```

### 9. Film avec la note maximale
```js
db.movies.mapReduce(
  function () {
    if (this.grades) {
      const max = Math.max(...this.grades.map(g => g.score));
      emit(this.title, max);
    }
  },
  function (key, values) { return Math.max(...values); },
  { out: "noteMaxParFilm" }
);
```

### 10. Nombre de notes supérieures à 70
```js
db.movies.mapReduce(
  function () {
    if (this.grades) {
      const count = this.grades.filter(g => g.score > 70).length;
      emit("notes_sup_70", count);
    }
  },
  function (key, values) { return Array.sum(values); },
  { out: "nbNotesSup70" }
);
```

### 11. Acteurs par genre sans doublons
```js
db.movies.mapReduce(
  function () {
    if (this.genres && this.cast) {
      this.genres.forEach(g => {
        this.cast.forEach(a => emit(g, a));
      });
    }
  },
  function (key, values) {
    return Array.from(new Set(values));
  },
  { out: "acteursParGenre" }
);
```

### 12. Acteurs les plus présents
```js
db.movies.mapReduce(
  function () {
    if (this.cast) {
      this.cast.forEach(a => emit(a, 1));
    }
  },
  function (key, values) { return Array.sum(values); },
  { out: "nbFilmsParActeur" }
);
```

### 13. Classement des films par grade majoritaire
```js
db.movies.mapReduce(
  function () {
    if (this.grades) {
      const freq = {};
      this.grades.forEach(g => {
        freq[g.grade] = (freq[g.grade] || 0) + 1;
      });
      const major = Object.keys(freq).reduce((a, b) => freq[a] > freq[b] ? a : b);
      emit(major, this.title);
    }
  },
  function (key, values) { return values; },
  { out: "classementParGradeMajoritaire" }
);
```

### 14. Note moyenne par année
```js
db.movies.mapReduce(
  function () {
    if (this.grades && this.year) {
      const avg = this.grades.reduce((a, b) => a + b.score, 0) / this.grades.length;
      emit(this.year, avg);
    }
  },
  function (key, values) { return Array.sum(values) / values.length; },
  { out: "noteMoyenneParAnnee" }
);
```

### 15. Réalisateurs avec une moyenne supérieure à 80
```js
db.movies.mapReduce(
  function () {
    if (this.director && this.grades) {
      const avg = this.grades.reduce((a, b) => a + b.score, 0) / this.grades.length;
      emit(this.director, { sum: avg, count: 1 });
    }
  },
  function (key, values) {
    let sum = 0, count = 0;
    values.forEach(v => {
      sum += v.sum;
      count += v.count;
    });
    return { sum: sum, count: count };
  },
  {
    out: "realisateursMoyenneSup80",
    finalize: function (key, value) {
      const avg = value.sum / value.count;
      return avg > 80 ? avg : null;
    }
  }
);
```

---

## 3. Conclusion

Ce TP nous a permis d’explorer le **modèle MapReduce** dans MongoDB à travers des cas concrets.  
Nous avons appris à structurer des traitements distribués en deux étapes (Map et Reduce) pour analyser efficacement de grandes collections.  
Bien que MapReduce soit aujourd’hui souvent remplacé par les pipelines d’agrégation pour des raisons de performance, il constitue une **excellente base pédagogique** pour comprendre les fondements du calcul distribué.

