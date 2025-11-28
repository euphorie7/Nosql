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
SET key "value"      # C <br>
GET key              # R <br>
SET key "new"        # U <br>
DEL key              # D <br>
INCR count           # compteur auto <br>

---

### 3.2 LISTS
RPUSH queue "A"      # C <br>
LRANGE queue 0 -1    # R <br>
LSET queue 0 "X"     # U <br>
LPOP queue           # D (suppr début) <br>
RPOP queue           # D (suppr fin) <br>

---

### 3.3 SETS (sans doublons)
SADD skills "redis" "nosql"         # C <br>
SMEMBERS skills                     # R <br>
SADD skills "cache"                 # U <br>
SREM skills "redis"                 # D <br>

---

### 3.4 SORTED SETS (triés)
ZADD rank 200 "hamza"               # C <br>
ZRANGE rank 0 -1 WITHSCORES         # R <br>
ZINCRBY rank 50 "hamza"             # U <br>
ZREM rank "hamza"                   # D <br>

---

### 3.5 HASHES (objet JSON-like)
HSET user:1 name "Hamza" age 22     # C <br>
HGETALL user:1                      # R <br>
HSET user:1 age 23                  # U <br>
DEL user:1                          # D <br>

---

# 4) Features avancées

### Expiration TTL
SET token "ABC" <br>
EXPIRE token 60 <br>
TTL token

### Pub/Sub
SUBSCRIBE news <br>
PUBLISH news "nouveau message" 

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

