# Guide d'intégration backend

Optionnel. L'app fonctionne standalone localStorage. Voici comment ajouter un backend.

---

## Option 1: Node.js + Express + SQLite (recommandé)

### Installation
```bash
npm init -y
npm install express sqlite3 cors dotenv
```

### Structure
```
backend/
├── server.js
├── database.js
├── routes/
│   └── campgrounds.js
└── data/
    └── schema.sql
```

### 1. database.js
```javascript
const sqlite3 = require('sqlite3');
const path = require('path');

const DB_PATH = path.join(__dirname, 'data', 'campgrounds.db');

const db = new sqlite3.Database(DB_PATH, (err) => {
  if (err) console.error('DB Error:', err);
  else console.log('Connected to SQLite');
});

// Création tables au démarrage
db.serialize(() => {
  db.run(`
    CREATE TABLE IF NOT EXISTS campgrounds (
      id INTEGER PRIMARY KEY,
      name TEXT NOT NULL,
      region TEXT,
      maxCapacity INTEGER,
      dogsAllowed BOOLEAN,
      maxDogs INTEGER,
      accessFee REAL,
      reservationFee REAL,
      types TEXT, -- JSON array
      pricing TEXT, -- JSON object
      extras TEXT, -- JSON array
      createdAt DATETIME DEFAULT CURRENT_TIMESTAMP,
      updatedAt DATETIME DEFAULT CURRENT_TIMESTAMP
    )
  `);
});

module.exports = db;
```

### 2. routes/campgrounds.js
```javascript
const express = require('express');
const router = express.Router();
const db = require('../database');

// GET tous les campgrounds
router.get('/', (req, res) => {
  db.all('SELECT * FROM campgrounds ORDER BY region, name', (err, rows) => {
    if (err) return res.status(500).json({ error: err.message });
    
    // Parse JSON columns
    const campgrounds = rows.map(row => ({
      ...row,
      types: JSON.parse(row.types),
      pricing: JSON.parse(row.pricing),
      extras: JSON.parse(row.extras)
    }));
    
    res.json(campgrounds);
  });
});

// GET un campground
router.get('/:id', (req, res) => {
  db.get(
    'SELECT * FROM campgrounds WHERE id = ?',
    [req.params.id],
    (err, row) => {
      if (err) return res.status(500).json({ error: err.message });
      if (!row) return res.status(404).json({ error: 'Not found' });
      
      const campground = {
        ...row,
        types: JSON.parse(row.types),
        pricing: JSON.parse(row.pricing),
        extras: JSON.parse(row.extras)
      };
      
      res.json(campground);
    }
  );
});

// POST nouveau campground
router.post('/', (req, res) => {
  const { name, region, maxCapacity, dogsAllowed, maxDogs, accessFee, reservationFee, types, pricing, extras } = req.body;
  
  if (!name || !region) {
    return res.status(400).json({ error: 'Name and region required' });
  }
  
  db.run(
    `INSERT INTO campgrounds (name, region, maxCapacity, dogsAllowed, maxDogs, accessFee, reservationFee, types, pricing, extras)
     VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)`,
    [
      name,
      region,
      maxCapacity,
      dogsAllowed ? 1 : 0,
      maxDogs,
      accessFee,
      reservationFee,
      JSON.stringify(types),
      JSON.stringify(pricing),
      JSON.stringify(extras)
    ],
    function(err) {
      if (err) return res.status(500).json({ error: err.message });
      
      res.status(201).json({
        id: this.lastID,
        ...req.body
      });
    }
  );
});

// PUT éditer campground
router.put('/:id', (req, res) => {
  const { name, region, maxCapacity, dogsAllowed, maxDogs, accessFee, reservationFee, types, pricing, extras } = req.body;
  
  db.run(
    `UPDATE campgrounds 
     SET name=?, region=?, maxCapacity=?, dogsAllowed=?, maxDogs=?, accessFee=?, reservationFee=?, types=?, pricing=?, extras=?, updatedAt=CURRENT_TIMESTAMP
     WHERE id=?`,
    [
      name,
      region,
      maxCapacity,
      dogsAllowed ? 1 : 0,
      maxDogs,
      accessFee,
      reservationFee,
      JSON.stringify(types),
      JSON.stringify(pricing),
      JSON.stringify(extras),
      req.params.id
    ],
    function(err) {
      if (err) return res.status(500).json({ error: err.message });
      if (this.changes === 0) return res.status(404).json({ error: 'Not found' });
      
      res.json({ id: req.params.id, ...req.body });
    }
  );
});

// DELETE campground
router.delete('/:id', (req, res) => {
  db.run('DELETE FROM campgrounds WHERE id=?', [req.params.id], function(err) {
    if (err) return res.status(500).json({ error: err.message });
    if (this.changes === 0) return res.status(404).json({ error: 'Not found' });
    
    res.json({ deleted: true, id: req.params.id });
  });
});

module.exports = router;
```

### 3. server.js
```javascript
require('dotenv').config();
const express = require('express');
const cors = require('cors');

const app = express();
const PORT = process.env.PORT || 5000;

app.use(cors());
app.use(express.json());

// Routes
const campgroundRoutes = require('./routes/campgrounds');
app.use('/api/campgrounds', campgroundRoutes);

// Health check
app.get('/api/health', (req, res) => {
  res.json({ status: 'OK' });
});

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

### 4. Lancer le serveur
```bash
node server.js
# http://localhost:5000/api/campgrounds
```

---

## Option 2: Supabase (serverless PostgreSQL)

### Setup
```bash
npm install @supabase/supabase-js
```

### supabase-client.js
```javascript
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = 'https://your-project.supabase.co';
const supabaseKey = 'YOUR_PUBLIC_ANON_KEY';

export const supabase = createClient(supabaseUrl, supabaseKey);

// Hook pour l'app React
export async function fetchCampgrounds() {
  const { data, error } = await supabase
    .from('campgrounds')
    .select('*')
    .order('region');
  
  if (error) throw error;
  return data;
}

export async function addCampground(campground) {
  const { data, error } = await supabase
    .from('campgrounds')
    .insert([campground])
    .select();
  
  if (error) throw error;
  return data[0];
}

export async function updateCampground(id, updates) {
  const { data, error } = await supabase
    .from('campgrounds')
    .update(updates)
    .eq('id', id)
    .select();
  
  if (error) throw error;
  return data[0];
}

export async function deleteCampground(id) {
  const { error } = await supabase
    .from('campgrounds')
    .delete()
    .eq('id', id);
  
  if (error) throw error;
}
```

### Table Supabase (SQL)
```sql
CREATE TABLE campgrounds (
  id BIGSERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  region TEXT,
  max_capacity INTEGER,
  dogs_allowed BOOLEAN,
  max_dogs INTEGER,
  access_fee DECIMAL(10,2),
  reservation_fee DECIMAL(10,2),
  types JSONB,
  pricing JSONB,
  extras JSONB,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

---

## Option 3: Firebase Firestore

### firebaseConfig.js
```javascript
import { initializeApp } from 'firebase/app';
import { getFirestore } from 'firebase/firestore';

const firebaseConfig = {
  apiKey: 'YOUR_API_KEY',
  projectId: 'your-project-id',
  // ... autres clés
};

const app = initializeApp(firebaseConfig);
export const db = getFirestore(app);
```

### hooks/useCampgrounds.js
```javascript
import { collection, getDocs, addDoc, updateDoc, deleteDoc, doc } from 'firebase/firestore';
import { useState, useEffect } from 'react';
import { db } from '../firebaseConfig';

export function useCampgrounds() {
  const [campgrounds, setCampgrounds] = useState([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    const fetchData = async () => {
      const snapshot = await getDocs(collection(db, 'campgrounds'));
      const data = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      setCampgrounds(data);
      setLoading(false);
    };
    
    fetchData();
  }, []);
  
  const addCampground = async (campground) => {
    const docRef = await addDoc(collection(db, 'campgrounds'), campground);
    setCampgrounds([...campgrounds, { id: docRef.id, ...campground }]);
  };
  
  const updateCampground = async (id, updates) => {
    await updateDoc(doc(db, 'campgrounds', id), updates);
    setCampgrounds(campgrounds.map(c => c.id === id ? { ...c, ...updates } : c));
  };
  
  const deleteCampground = async (id) => {
    await deleteDoc(doc(db, 'campgrounds', id));
    setCampgrounds(campgrounds.filter(c => c.id !== id));
  };
  
  return { campgrounds, loading, addCampground, updateCampground, deleteCampground };
}
```

---

## Intégration dans l'app React

### Remplacer localStorage par API

```javascript
// Avant (localStorage)
const [campgrounds, setCampgrounds] = useState(() => {
  const saved = localStorage.getItem('campgrounds');
  return saved ? JSON.parse(saved) : DEFAULT_CAMPGROUNDS;
});

// Après (API)
const [campgrounds, setCampgrounds] = useState([]);

useEffect(() => {
  fetch('http://localhost:5000/api/campgrounds')
    .then(r => r.json())
    .then(data => setCampgrounds(data))
    .catch(err => console.error(err));
}, []);

useEffect(() => {
  // Sync changes
  if (campgrounds.length > 0) {
    localStorage.setItem('campgrounds', JSON.stringify(campgrounds));
  }
}, [campgrounds]);
```

---

## Déploiement Backend

### Heroku (deprecated mais exemple)
```bash
git init
heroku create your-app-name
git push heroku main
```

### Railway
```bash
npm install -g railway
railway init
railway up
```

### Render
```bash
# Connecter repo GitHub → Render
# Déploiement auto sur chaque push
```

### Docker
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 5000
CMD ["node", "server.js"]
```

---

## Sécurité

### .env.example
```
PORT=5000
DB_PATH=./data/campgrounds.db
SUPABASE_URL=xxx
SUPABASE_KEY=xxx
FIREBASE_CONFIG=xxx
```

### CORS restrictif
```javascript
const cors = require('cors');

app.use(cors({
  origin: ['https://votre-domain.com', 'http://localhost:3000'],
  credentials: true
}));
```

### Validation inputs
```javascript
const validateCampground = (camp) => {
  if (!camp.name || camp.name.length < 2) throw new Error('Invalid name');
  if (!camp.region) throw new Error('Region required');
  if (camp.maxCapacity < 1) throw new Error('Invalid capacity');
  return true;
};
```

---

## Tests API

### cURL
```bash
# GET
curl http://localhost:5000/api/campgrounds

# POST
curl -X POST http://localhost:5000/api/campgrounds \
  -H "Content-Type: application/json" \
  -d '{"name":"New Camp","region":"Laurentides",...}'

# PUT
curl -X PUT http://localhost:5000/api/campgrounds/1 \
  -H "Content-Type: application/json" \
  -d '{"name":"Updated"}'

# DELETE
curl -X DELETE http://localhost:5000/api/campgrounds/1
```

### Postman
Importer collection JSON :
```json
{
  "info": { "name": "CampCalc API" },
  "item": [
    {
      "name": "Get Campgrounds",
      "request": {
        "method": "GET",
        "url": "http://localhost:5000/api/campgrounds"
      }
    }
  ]
}
```

---

**Recommandation** : Commencer localStorage (0 dépendance), puis évaluer backend si besoin multi-user.
