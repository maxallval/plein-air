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
  name: "Camping Mont-Tremblant",
  region: "Laurentides",
  maxCapacity: 6,
  dogsAllowed: true,
  maxDogs: 2,
  
  // Frais fixes
  accessFee: 5,              // Par personne
  reservationFee: 15,        // Fixe
  
  types: ["Tente", "VR", "Prêt-à-camper"],
  
  // Tarification par saison
  pricing: {
    basse: { base: 35, weekend: 40 },
    moyenne: { base: 45, weekend: 55 },
    haute: { base: 55, weekend: 65 }
  },
  
  // Extras optionnels
  extras: [
    { id: 1, name: "Bois", price: 15, billedPer: "séjour" },
    { id: 2, name: "Chien +", price: 10, billedPer: "nuit" },
    { id: 3, name: "Canot", price: 40, billedPer: "jour" }
  ]
}
```

### Formule de calcul

```
1. Nuits = ceil((dateDepart - dateArrivee) / 86400)

2. Tarif par nuit:
   - Saison = "haute" si juin-août, "moyenne" si déc-jan, "basse" sinon
   - Jour = weekend si vendredi/samedi
   - Prix = pricing[saison].weekend (si weekend) OU pricing[saison].base

3. Sous-total:
   - Hébergement = Σ(tarif par nuit)
   - Accès = accessFee × numPersonnes
   - Réservation = reservationFee (fixe)
   - Extras = Σ(prix × quantité × multiplier)
   
4. Taxes = Sous-total × 0.15  (TPS 5% + TVQ ~10%)

5. TOTAL = Sous-total + Taxes
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
  id: 5,
  name: "Votre Camping",
  region: "Région",
  maxCapacity: 6,
  dogsAllowed: true,
  maxDogs: 2,
  accessFee: 5,
  reservationFee: 20,
  types: ["Tente", "VR"],
  pricing: {
    basse: { base: 30, weekend: 38 },
    moyenne: { base: 42, weekend: 50 },
    haute: { base: 52, weekend: 62 }
  },
  extras: [
    { id: 1, name: "Bois", price: 14, billedPer: "séjour" },
    { id: 2, name: "Chien +", price: 9, billedPer: "nuit" }
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

### Scénario 1 : Famille weekend
- 2 adultes + 1 enfant
- 27-28 août (haute saison, fin de semaine)
- Camping Mont-Tremblant, Tente
- 1 bois de chauffage

**Résultat** :
```
Hébergement: 65 × 1 = 65$
Accès: 5 × 3 = 15$
Réservation: 15$
Bois: 15$
Sous-total: 110$
Taxes (15%): 16.50$
TOTAL: 126.50$
Par personne: 42.17$
```

### Scénario 2 : Semaine basse saison
- 4 adultes
- 15-20 avril (basse saison)
- Camping Lac-Mégantic, VR
- 2 chiens supplémentaires

**Résultat** :
```
Hébergement: 30 × 5 = 150$
Accès: 4 × 4 = 16$
Réservation: 20$
Chiens (5 nuits × 2): 8 × 10 = 80$
Sous-total: 266$
Taxes: 39.90$
TOTAL: 305.90$
Par personne: 76.48$
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
