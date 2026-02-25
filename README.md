# SHOM Raster Marine - Relais MapProxy

Relais MapProxy pour le service WMTS **RASTER_MARINE_3857** du SHOM (Service Hydrographique et Océanographique de la Marine).

## 📋 Description

Ce projet configure un serveur MapProxy qui agit comme relais/cache pour les tuiles marines raster du SHOM. Il permet de redistribuer les cartes marines via les protocoles WMS, WMTS et TMS tout en mettant en cache les tuiles pour améliorer les performances.

## 🚀 Démarrage rapide

### Prérequis

- Docker
- Docker Compose

### Installation

1. Clonez ce dépôt :
```bash
git clone <url-du-depot>
cd SHOM_RASTER_MARINE
```

2. Démarrez les services :
```bash
docker-compose up -d
```

3. Accédez à MapProxy :
   - Interface web : http://localhost:8080
   - Service WMTS : http://localhost:8080/wmts
   - Service WMS : http://localhost:8080/service
   - Service TMS : http://localhost:8080/tms

## 📁 Structure du projet

```
.
├── docker-compose.yml          # Configuration Docker Compose
├── mapproxy/
│   ├── mapproxy.yaml          # Configuration MapProxy
│   ├── seed.yaml              # Configuration de pré-seeding (optionnel)
│   └── cache_data/            # Répertoire de cache (non versionné)
└── nginx/                      # Configuration Nginx (optionnel)
```

## ⚙️ Configuration

### Source WMTS

Le service source est le WMTS du SHOM :
- **URL** : https://services.data.shom.fr/clevisu/wmts
- **Layer** : RASTER_MARINE_3857_WMTS
- **TileMatrixSet** : 3857 (Web Mercator)
- **Format** : image/png

**Note importante** : Le service SHOM nécessite l'en-tête HTTP `Referer: https://data.shom.fr/` pour fonctionner correctement.

### Couches disponibles

- `RASTER_MARINE_3857` : Cartes marines raster en projection Web Mercator (EPSG:3857)

### Cache

Le cache est configuré avec :
- **Type** : File system
- **Format** : PNG
- **Meta-size** : 4x4 (pré-téléchargement de tuiles voisines)
- **Répertoire** : `mapproxy/cache_data/shom/`

## 🔧 Utilisation

### Visualisation dans QGIS

1. Ouvrir QGIS
2. Ajouter une connexion WMS/WMTS :
   - **URL** : `http://localhost:8080/service?` (pour WMS)
   - **URL** : `http://localhost:8080/wmts/1.0.0/WMTSCapabilities.xml` (pour WMTS)
3. Ajouter la couche `RASTER_MARINE_3857`

### Intégration dans OpenLayers / Leaflet

**OpenLayers** :
```javascript
import TileLayer from 'ol/layer/Tile';
import WMTS from 'ol/source/WMTS';
import WMTSTileGrid from 'ol/tilegrid/WMTS';

const layer = new TileLayer({
  source: new WMTS({
    url: 'http://localhost:8080/wmts',
    layer: 'RASTER_MARINE_3857',
    matrixSet: 'webmercator',
    format: 'image/png',
    projection: 'EPSG:3857',
    tileGrid: new WMTSTileGrid({
      origin: [-20037508.34, 20037508.34],
      resolutions: [...], // Résolutions Web Mercator
      matrixIds: [...] // 0-18
    })
  })
});
```

**Leaflet** :
```javascript
L.tileLayer('http://localhost:8080/tms/1.0.0/RASTER_MARINE_3857/webmercator/{z}/{x}/{y}.png', {
  attribution: '© SHOM',
  tms: true
}).addTo(map);
```

### Pré-seeding du cache

Pour pré-télécharger des tuiles dans le cache :

```bash
docker exec -it shom_mapproxy mapproxy-seed -f /mapproxy/mapproxy.yaml -s /mapproxy/seed.yaml
```

## 📝 Notes importantes

### Conditions d'utilisation

⚠️ **Important** : Ce service relaye les données du SHOM. Veuillez respecter les [conditions d'utilisation du SHOM](https://data.shom.fr/) et les licences applicables aux données marines.

### Performance

- Le cache améliore significativement les performances pour les zones fréquemment consultées
- Le meta-tiling (4x4) réduit le nombre de requêtes vers le serveur source
- Le cache peut devenir volumineux selon les zones couvertes

### Limitations

- Les données sont en projection EPSG:3857 uniquement (Web Mercator)
- Le cache n'est pas automatiquement mis à jour (configurer un rafraîchissement si nécessaire)

## 🛠️ Maintenance

### Gestion du cache

```bash
# Voir la taille du cache
du -sh mapproxy/cache_data/shom/

# Nettoyer le cache
docker-compose down
rm -rf mapproxy/cache_data/shom/*
docker-compose up -d
```

### Logs

```bash
# Voir les logs en temps réel
docker-compose logs -f mapproxy
```

## 📚 Ressources

- [Documentation MapProxy](https://mapproxy.org/docs/latest/)
- [SHOM Data](https://data.shom.fr/)
- [Spécifications WMTS](https://www.ogc.org/standards/wmts)

## 📄 Licence

Ce projet est sous licence [MIT](LICENSE) (ou autre licence de votre choix).

Les données marines sont propriété du SHOM et soumises à leurs propres conditions d'utilisation.

## 👤 Auteur

[Votre nom]

## 🤝 Contribution

Les contributions sont les bienvenues ! N'hésitez pas à ouvrir une issue ou une pull request.
