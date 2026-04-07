# CampCalc — Calculateur de Camping Québec

**Single-file React application** pour calculer le coût complet d'un séjour en camping au Québec.

---

## 🚀 Démarrage rapide

### Installation
1. Téléchargez `campground-calculator.html`
2. Double-cliquez pour ouvrir dans le navigateur
3. **Aucune dépendance externe n'est requise** (tout est bundlé CDN)

### Déploiement GitHub Pages
```bash
# Dans votre repo GitHub
1. Créez un dossier /docs
2. Copiez campground-calculator.html dedans
3. Settings → Pages → Source = /docs
4. Accédez via: https://votre-username.github.io/repo-name/docs/campground-calculator.html
```

---

## 📋 Fonctionnalités principales

### Calculateur (Onglet 1)
- **Sélection du camping** : Cartes interactives avec tarifs
- **Dates** : Détection automatique haute/basse saison
- **Personnes** : Adultes + enfants (capacité max respectée)
- **Type d'hébergement** : Tente, VR, Prêt-à-camper
- **Extras** : Bois, chiens, équipement location, etc.
- **Calcul en temps réel** : TPS/TVQ incluses

### Résumé du coût
- Prix **total**
- Prix **par personne**
- Prix **par nuit**
- **Détail** : hébergement, accès, réservation, extras, taxes

### Gestion des campings (Onglet 2)
- Ajouter/éditer/supprimer des campings
- Configurer tarification (base, fin de semaine, saisons)
- Gérer frais fixes (accès, réservation)
- Définir extras (prix, facturation par nuit/séjour)
- **Données persistantes** : Sauvegardées en localStorage

---

## 🧮 Logique de tarification

### Structure de données (campground)
```javascript
{
  id: 1,
  name: "Parc du Poisson Blanc",
  region: "Outaouais",
  maxCapacity: 16,
  dogsAllowed: true,
  maxDogs: 2,
  
  // Frais fixes
  accessFee: 1.78,        // Par personne (frais conservation)
  reservationFee: 0,      // Fixe
  
  types: ["Tente"],
  
  // ✨ NOUVEAU: Tarification par type d'hébergement
  accommodationTypes: {
    "1tente": {
      label: "1 tente",
      maxCapacity: 4,
      pricing: {
        basse: { 
          weekday: 59.00,     // En semaine
          weekend: 79.00,     // Fin de semaine
          minNights: 2,       // Minimum de nuits
          maxNights: 7        // Maximum de nuits
        },
        haute: { 
          weekday: 89.00, 
          weekend: 89.00, 
          minNights: 2, 
          maxNights: 7 
        }
      }
    },
    "2tentes": { /* ... */ },
    "3tentes": { /* ... */ },
    "4tentes": { /* ... */ }
  },
  
  // Extras optionnels
  extras: [
    { id: 1, name: "Canot 2 bancs", price: 43.00, billedPer: "jour" },
    { id: 5, name: "Kayak de mer solo", price: 38.00, billedPer: "jour" },
    // ...
  ]
}
```

### Formule de calcul

```
1. Nuits = ceil((dateDepart - dateArrivee) / 86400)

2. Validation minNights/maxNights:
   - accType = accommodationTypes[selectedType]
   - Si nuits < accType.pricing[season].minNights → ERREUR
   - Si nuits > accType.pricing[season].maxNights → ERREUR

3. Tarif par nuit:
   - Saison = "haute" si juin-août, "basse" sinon
   - Jour = weekend si vendredi/samedi
   - Prix = accType.pricing[saison].weekend (si weekend)
           OU accType.pricing[saison].weekday
   
   Note: minNights/maxNights varient par saison et type
   - Exemple: 2 tentes basse saison = min 2 nuits en semaine, min 3 nuits fin de semaine

4. Sous-total:
   - Hébergement = Σ(tarif par nuit pour chaque jour)
   - Accès = accessFee × numPersonnes
   - Réservation = reservationFee (fixe, souvent 0)
   - Extras = Σ(prix × quantité × multiplier)
     * multiplier = nuits si billedPer="jour"
     * multiplier = 1 si billedPer="séjour"
   
5. Taxes = Sous-total × 0.15  (TPS 5% + TVQ ~10%)

6. TOTAL = Sous-total + Taxes
```

### Détection saisons (personnalisable)
- **Basse saison** : Jan-avril, Sept-Nov
- **Moyenne saison** : Décembre
- **Haute saison** : Juin-Août

💡 Modifiable dans `getSeason()` (ligne ~140)

---

## 🔧 Customisation

### Ajouter un campground par défaut
Éditer `DEFAULT_CAMPGROUNDS` (ligne ~500) :

```javascript
{
  id: 1,
  name: "Parc du Poisson Blanc",
  region: "Outaouais",
  maxCapacity: 16,
  dogsAllowed: true,
  maxDogs: 2,
  accessFee: 1.78,
  reservationFee: 0,
  types: ["Tente"],
  
  // Tarification par type d'hébergement
  accommodationTypes: {
    "1tente": {
      label: "1 tente",
      maxCapacity: 4,
      pricing: {
        basse: { weekday: 59, weekend: 79, minNights: 2, maxNights: 7 },
        haute: { weekday: 89, weekend: 89, minNights: 2, maxNights: 7 }
      }
    },
    "2tentes": {
      label: "2 tentes",
      maxCapacity: 8,
      pricing: {
        basse: { weekday: 69, weekend: 99, minNights: 2, maxNights: 7 },
        haute: { weekday: 109, weekend: 109, minNights: 2, maxNights: 7 }
      }
    }
    // Ajouter autant de types que nécessaire
  },
  
  extras: [
    { id: 1, name: "Canot 2 bancs", price: 43.00, billedPer: "jour" },
    { id: 5, name: "Kayak de mer solo", price: 38.00, billedPer: "jour" }
  ]
}
```

### Modifier les couleurs
Variables CSS (ligne ~10) :

```css
:root {
  --pine: #2d5016;        /* Vert foncé principal */
  --sage: #7a9b6f;        /* Vert pâle */
  --accent: #c67c4e;      /* Orange/brun accent */
  --stone: #5a5450;       /* Gris texte */
  /* ... */
}
```

### Ajuster la taxe
Ligne ~162, fonction `calculateCost()` :

```javascript
const taxRate = 0.15;  // Actuellement TPS+TVQ (15%)
// Changez à 0.05 pour TPS seule, 0.09975 pour TVQ seule
```

### Changer la saison Min-Max
Fonction `getSeason()` (ligne ~138) :

```javascript
const getSeason = (date) => {
  const month = new Date(date).getMonth();
  if (month >= 5 && month <= 7) return 'haute';  // Juin-Août
  if (month === 11 || month === 0) return 'moyenne';  // Déc-Jan
  return 'basse';  // Reste
};
```

---

## 📱 Structure du code

### Composants principaux
1. **CampgroundCalculator** (root)
   - Gère état global, localStorage, onglets

2. **CalculatorTab**
   - Formulaire entrée
   - Affichage résumé coûts
   - Gestion extras

3. **AdminTab**
   - Liste campings
   - Statistiques

4. **CampgroundModal**
   - Formulaire ajout/édition campground
   - Tarification multi-saisons

### Utilitaires clés
- `calculateCost()` : Engine de tarification central
- `calculateNights()` : Calcul nuits (gère saisons)
- `getSeason()` : Détection haute/basse saison
- `isWeekend()` : Vendredi-samedi
- `formatCAD()` : Formatting CAD

---

## 💾 Données localStorage

**Clé** : `campgrounds`  
**Format** : JSON array  
**Scope** : Domain only (`file://` ne persiste PAS)

⚠️ **Pour localhost + localStorage** :
- Utilisez un serveur local (`python -m http.server`)
- Ou GitHub Pages
- Pas `file://` direct

```bash
# Serveur Python simple
python -m http.server 8000
# Accédez via http://localhost:8000/campground-calculator.html
```

---

## 🐛 Gestion des erreurs

| Cas | Comportement |
|-----|-------------|
| Chiens sélectionnés + camping non permis | ⚠️ Banneau d'erreur |
| Dates invalides | Message "Minimum 1 nuit requis" |
| 0 personne | "Minimum 1 personne requis" |
| Camping non sélectionné | Type d'hébergement désactivé |
| Suppression camping actif | Réinitialise sélection |

---

## 🎨 Design

**Esthétique** : Canadienne minimaliste outdoorsy  
**Palette** : Verts forestiers (pine #2d5016) + stone + cream  
**Typographie** : Georgia (headings) + system-ui (body)  
**Responsive** : Mobile-first grid system

### Points clés UX
- Sticky calculator résumé (desktop)
- Cards cliquables pour campgrounds
- Animations fadeIn légères
- Inputs validés en temps réel
- Détail complet du coût toujours visible

---

## 📊 Exemples de scénarios

### Scénario 1 : Weekend en basse saison (2 tentes)
- 6 personnes (2 adultes + 4 enfants)
- 27-29 septembre (3 nuits, samedi-lundi = weekend)
- Parc du Poisson Blanc, 2 tentes
- 1 canot 2 bancs (3 jours)

**Résultat** :
```
2 tentes fin de semaine basse saison: 99 × 3 = 297$
Accès (frais conservation): 1.78 × 6 = 10.68$
Réservation: 0$
Canot (3 jours): 43 × 3 = 129$
Sous-total: 436.68$
Taxes (15%): 65.50$
TOTAL: 502.18$
Par personne: 83.70$
Par nuit: 167.39$
```

### Scénario 2 : Semaine en basse saison (1 tente)
- 3 personnes
- 15-19 avril (4 nuits, lun-ven = semaine)
- Parc du Poisson Blanc, 1 tente
- 1 kayak de mer solo (4 jours)

**Résultat** :
```
1 tente en semaine basse saison: 59 × 4 = 236$
Accès: 1.78 × 3 = 5.34$
Réservation: 0$
Kayak (4 jours): 38 × 4 = 152$
Sous-total: 393.34$
Taxes (15%): 59.00$
TOTAL: 452.34$
Par personne: 150.78$
Par nuit: 113.09$
```

### Scénario 3 : Haute saison (4 tentes)
- 12 personnes
- 20-23 juillet (3 nuits, mer-sam = mélange semaine/weekend)
- Parc du Poisson Blanc, 4 tentes
- 2 canots 2 bancs (3 jours)

**Résultat** :
```
4 tentes haute saison: 149 × 3 = 447$
Accès: 1.78 × 12 = 21.36$
Réservation: 0$
Canots (2 × 43 × 3): 258$
Sous-total: 726.36$
Taxes (15%): 108.95$
TOTAL: 835.31$
Par personne: 69.61$
Par nuit: 278.44$
```

---

## 🔐 Sécurité & Performance

- ✅ Aucun serveur requis (client-side only)
- ✅ Pas de requêtes API externes
- ✅ localStorage local (données jamais envoyées)
- ✅ Validation inputs côté client
- ✅ React 18 minified CDN

**Taille** : ~150KB HTML (incluant React CDN inline)

---

## 📈 Roadmap (futurs ajouts possibles)

- [ ] Filtre: chiens permis, prix max, bord de l'eau
- [ ] Export PDF devis
- [ ] Comparaison 2 campings side-by-side
- [ ] Sauvegarde scénarios nommés
- [ ] Intégration calendar 2025 jours fériés
- [ ] API backend (Node.js/Supabase optionnel)
- [ ] PWA mode offline
- [ ] Multi-langue (FR/EN)

---

## 📞 Support

**Questions fréquentes** :

**Q: Comment ajouter mes propres campgrounds?**  
A: Onglet "Gestion des campings" → Ajouter. Données sauvegardées localhost.

**Q: Comment exporter les données?**  
A: Browser DevTools → Application → localStorage → Copier JSON `campgrounds`

**Q: Peut-on ajouter plusieurs enfants?**  
A: Oui, champ "Enfants" accepte n'importe quel nombre (pas de limite logique)

**Q: Taxes TPS/TVQ exactes?**  
A: Actuellement 15% agrégé. Ajustez `taxRate = 0.15` ligne 162 si besoin précision.

---

## 📄 Licence

Code libre d'usage. Modifiez librement pour vos besoins.

---

**Créé avec ❤️ pour les campeurs du Québec.**
