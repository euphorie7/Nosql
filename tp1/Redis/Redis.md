#  Redis & NoSQL – Stockage Moderne Performant + CRUD

##  1) NoSQL — Définition
NoSQL = "Not Only SQL"
- base non relationnelle
- flexible, rapide, scalable
- données semi-structurées

### Points forts
- pas de schéma figé
- ultra scalable (horizontale)
- support Big Data
- performance extrême
- stockage JSON / Objets / Documents

### Familles NoSQL
| Type | Description | Exemples |
|---|---|---|
| Clé/valeur | rapide et minimal | Redis, DynamoDB |
| Documents | JSON, schéma libre | MongoDB, CouchDB |
| Colonnes | mass data / BI | Cassandra, HBase |
| Graphes | relations complexes | Neo4j, Neptune |

---

#  2) Redis — Base NoSQL in-memory

Redis = stockage clé-valeur rapide <br>
→ sert de cache, <br>
→ base DB, <br>
→ broker, <br>
→ file d’attente <br>

### Installation Linux
sudo apt install redis-server <br>
sudo systemctl start redis-server
redis-cli

---

#  3) CRUD Redis par type de données

### 3.1 STRINGS
```bash
SET key "value"      # C 
GET key              # R 
SET key "new"        # U 
DEL key              # D 
INCR count           # compteur auto 
```
---

### 3.2 LISTS

```bash
RPUSH queue "A"      # C 
LRANGE queue 0 -1    # R 
LSET queue 0 "X"     # U 
LPOP queue           # D (suppr début) 
RPOP queue           # D (suppr fin) 
```
---

### 3.3 SETS (sans doublons)
```bash
SADD skills "redis" "nosql"         # C 
SMEMBERS skills                     # R 
SADD skills "cache"                 # U 
SREM skills "redis"                 # D 
```
---

### 3.4 SORTED SETS (triés)
```bash
ZADD rank 200 "hamza"               # C 
ZRANGE rank 0 -1 WITHSCORES         # R 
ZINCRBY rank 50 "hamza"             # U 
ZREM rank "hamza"                   # D 
```
---

### 3.5 HASHES (objet JSON-like)
```bash
HSET user:1 name "Hamza" age 22     # C 
HGETALL user:1                      # R 
HSET user:1 age 23                  # U 
DEL user:1                          # D 
```
---

# 4) Features avancées

### Expiration TTL
```bash
SET token "ABC" 
EXPIRE token 60 
TTL token
```
### Pub/Sub
```bash
SUBSCRIBE news 
PUBLISH news "nouveau message" 
```

### Performance
latence = nanosecondes <br>
débit ≈ 4GB/s <br>
RAM native

---

#  5) Cas d’usage
- cache API et requêtes lourdes <br>
- sessions utilisateur / tokens <br>
- messagerie temps réel (pub/sub) <br> 
- leaderboard / classements <br>
- analytics en live <br>
- microservices haute charge <br>


---

