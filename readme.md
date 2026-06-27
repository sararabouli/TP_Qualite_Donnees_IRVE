# TP — Qualité des données IRVE

## 1. Présentation du projet

Ce projet a été réalisé dans le cadre d’un TP sur la qualité des données.
Il porte sur un dataset IRVE, relatif aux infrastructures de recharge pour véhicules électriques.

L’objectif est de mettre en place une démarche complète de qualité des données :

* audit initial du fichier ;
* identification des anomalies ;
* nettoyage et enrichissement ;
* validation automatique avec Great Expectations ;
* génération de Data Docs ;
* mise en place d’un monitoring qualité entre deux livraisons.

Le projet est réalisé dans un notebook Jupyter avec Python.

---

## 2. Objectifs du TP

Le TP répond aux objectifs suivants :

1. Analyser la qualité initiale du dataset.
2. Identifier les valeurs manquantes, doublons, formats incorrects et incohérences métier.
3. Contrôler les coordonnées GPS et les données géographiques.
4. Nettoyer et enrichir les données.
5. Formaliser des règles qualité avec Great Expectations.
6. Générer une documentation de validation automatique.
7. Comparer deux livraisons du fichier.
8. Définir des seuils d’alerte pour surveiller la qualité dans le temps.

---

## 3. Structure du dépôt

La structure finale du projet est la suivante :

```text
TP_Qualite_Donnees_IRVE/
│
├── data/
│   └── README.md
│
├── outputs_final/
│   ├── rapport_profiling_irve_initial.html
│   ├── tableau_suivi_qualite_irve.csv
│   └── regles_alerte_qualite_irve.csv
│
├── TP_Qualite_Donnees_IRVE_final.ipynb
├── README.md
├── requirements.txt
└── .gitignore
```

### Description des principaux éléments

| Élément                                       | Description                                           |
| --------------------------------------------- | ----------------------------------------------------- |
| `TP_Qualite_Donnees_IRVE_final.ipynb` | Notebook principal contenant tout le traitement du TP |
| `requirements.txt`                            | Liste des bibliothèques nécessaires                   |
| `README.md`                                   | Documentation du projet                               |
| `.gitignore`                                  | Fichiers et dossiers exclus du dépôt GitHub           |
| `data/`                                       | Dossier destiné au fichier source IRVE                |
| `outputs_final/`                              | Dossier contenant les livrables générés               |
| `rapport_profiling_irve_initial.html`         | Rapport de profiling initial                          |
| `tableau_suivi_qualite_irve.csv`              | Tableau de monitoring qualité                         |
| `regles_alerte_qualite_irve.csv`              | Règles d’alerte proposées                             |

---

## 4. Données utilisées

Le fichier source utilisé est :

```text
consolidation-etalab-schema-irve-statique-v-2.3.1-20260625.csv
```

Ce fichier contient des données relatives aux points de charge IRVE, notamment :

* identifiants de points de charge ;
* coordonnées GPS ;
* communes ;
* codes postaux ;
* puissances nominales ;
* dates de mise à jour ;
* informations techniques et administratives.

Le fichier source n’est pas inclus dans le dépôt GitHub car il est volumineux.

Pour exécuter le notebook, il doit être placé manuellement dans le dossier `data/` ou à l’emplacement attendu par le notebook, selon la version utilisée.

---

## 5. Installation

Créer un environnement virtuel :

```bash
python -m venv .venv
```

Activer l’environnement sous Windows :

```bash
.venv\Scripts\activate
```

Installer les dépendances :

```bash
pip install -r requirements.txt
```

Lancer Jupyter :

```bash
jupyter notebook
```

Ouvrir ensuite le fichier :

```text
TP_Qualite_Donnees_IRVE_final.ipynb
```

---

## 6. Bibliothèques utilisées

Les principales bibliothèques utilisées sont :

| Bibliothèque         | Utilisation                                      |
| -------------------- | ------------------------------------------------ |
| `pandas`             | Manipulation et analyse des données tabulaires   |
| `numpy`              | Calculs numériques                               |
| `matplotlib`         | Visualisations graphiques                        |
| `ydata-profiling`    | Rapport automatique de profiling                 |
| `great-expectations` | Validation automatique de la qualité des données |
| `folium`             | Cartographie interactive                         |
| `geopy`              | Tests de géocodage inverse                       |
| `geopandas`          | Traitements géospatiaux                          |
| `pyogrio`            | Lecture de données géographiques                 |
| `shapely`            | Manipulation de géométries                       |
| `tqdm`               | Barres de progression                            |
| `ftfy`               | Correction de problèmes d’encodage               |
| `requests`           | Accès à des ressources externes                  |
| `jupyter`            | Exécution du notebook                            |
| `ipykernel`          | Kernel Python pour Jupyter                       |

---

## 7. Démarche suivie

### 7.1 Audit initial

La première étape consiste à analyser la qualité initiale du dataset.

Les contrôles réalisés portent notamment sur :

* le nombre de lignes et de colonnes ;
* les valeurs manquantes ;
* les doublons ;
* les formats de colonnes ;
* les coordonnées GPS ;
* les codes postaux ;
* les puissances nominales ;
* les dates de mise à jour.

Un rapport automatique de profiling a été généré afin d’obtenir une vision globale du dataset.

---

### 7.2 Nettoyage et enrichissement

Plusieurs traitements ont été appliqués :

* normalisation de certaines colonnes ;
* correction du format des codes postaux ;
* contrôle des coordonnées GPS ;
* vérification des puissances nominales ;
* analyse des doublons ;
* enrichissement des communes manquantes.

Le principal problème identifié concernait la colonne `consolidated_commune`, qui contenait beaucoup de valeurs manquantes.

Avant correction, la complétude des communes était de :

```text
60,43 %
```

---

### 7.3 Enrichissement spatial des communes

Pour améliorer la complétude des communes, une jointure spatiale a été utilisée.

Le principe est le suivant :

1. Transformer les coordonnées GPS des points de charge en objets géographiques.
2. Charger un référentiel géographique des communes.
3. Croiser les points GPS avec les contours communaux.
4. Affecter la commune correspondant au polygone dans lequel se situe chaque point.
5. Conserver les valeurs restantes comme anomalies résiduelles lorsqu’aucune commune ne peut être déterminée de manière fiable.

Cette méthode permet d’éviter une imputation arbitraire et repose sur une logique géographique explicable.

Après enrichissement, il reste :

```text
142 communes manquantes
```

La complétude des communes atteint :

```text
99,91 %
```

---

## 8. Validation avec Great Expectations

Une suite de validation Great Expectations a été créée afin de contrôler automatiquement la qualité du dataset final.

Les règles portent notamment sur :

* l’unicité des identifiants ;
* la validité des coordonnées GPS ;
* la cohérence des puissances ;
* le format des codes postaux ;
* la complétude des dates de mise à jour ;
* la complétude des communes ;
* le nombre minimal de lignes.

La validation finale est réussie :

```text
Validation Great Expectations : True
```

Les Data Docs Great Expectations ont été générés afin de fournir une preuve visuelle des validations.

Certaines validations intermédiaires peuvent apparaître en échec dans les Data Docs. Elles correspondent aux essais réalisés avant correction complète des données. Les dernières validations réussies confirment que le dataset final respecte les règles qualité définies.

---

## 9. Monitoring qualité

La dernière partie du TP consiste à surveiller la qualité dans le temps.

Deux livraisons sont comparées :

* **J-n** : état du dataset avant enrichissement spatial ;
* **J** : état final après nettoyage, enrichissement et validation.

Les indicateurs suivis sont :

* complétude globale ;
* complétude des communes ;
* taux de doublons ;
* pourcentage de lignes valides.

### Résultats du monitoring

| Livraison                | Nombre de lignes | Nombre de colonnes | Complétude globale (%) | Complétude communes (%) | Taux de doublons (%) | Lignes valides (%) |
| ------------------------ | ---------------: | -----------------: | ---------------------: | ----------------------: | -------------------: | -----------------: |
| J-n avant enrichissement |          160 255 |                 72 |                  89,93 |                   60,43 |                 0,07 |              60,43 |
| J final                  |          160 255 |                 73 |                  90,22 |                   99,91 |                 0,07 |              99,91 |

La qualité s’améliore nettement entre les deux livraisons, en particulier grâce à l’enrichissement spatial des communes.

---

## 10. Règles d’alerte proposées

Des seuils d’alerte ont été définis afin de détecter automatiquement une dégradation de la qualité sur les futures livraisons.

| Indicateur                    | Seuil d’alerte | Justification                                                                  |
| ----------------------------- | -------------- | ------------------------------------------------------------------------------ |
| Complétude globale            | < 90 %         | Une baisse de complétude peut limiter l’exploitation fiable du fichier         |
| Complétude communes           | < 95 %         | La commune est indispensable pour les analyses territoriales                   |
| Taux de doublons              | > 1 %          | Un taux élevé de doublons peut fausser le comptage des points de charge        |
| Lignes valides                | < 95 %         | Une part trop importante de lignes invalides rend le fichier moins exploitable |
| Validation Great Expectations | False          | Un échec indique qu’au moins une règle qualité critique n’est pas respectée    |

Sur la livraison finale :

```text
Statut monitoring : OK
Aucune alerte détectée sur la livraison finale.
```

---

## 11. Livrables générés

Les livrables finaux sont regroupés dans le dossier `outputs_final/`.

```text
outputs_final/
│
├── rapport_profiling_irve_initial.html
├── tableau_suivi_qualite_irve.csv
└── regles_alerte_qualite_irve.csv
```

Le fichier `dataset_irve_nettoye_enrichi.csv` est généré localement mais n’est pas inclus dans le dépôt GitHub s’il dépasse la limite autorisée.

---

## 12. Difficultés rencontrées

Plusieurs difficultés ont été rencontrées pendant le projet.

### 12.1 Volume du fichier

Le fichier source est volumineux, ce qui rend son partage sur GitHub difficile.
GitHub classique limite les fichiers de grande taille. Le fichier source n’est donc pas inclus dans le dépôt.

### 12.2 Communes manquantes

La principale difficulté qualité concernait les valeurs manquantes dans la colonne `consolidated_commune`.

Les premières tentatives d’enrichissement par code INSEE ou par coordonnées identiques n’étaient pas suffisantes.
La solution retenue a été une jointure spatiale avec les contours communaux.

### 12.3 Coordonnées GPS

Certaines coordonnées pouvaient être imprécises, absentes ou situées hors des contours communaux.
Les 142 communes encore manquantes après correction correspondent probablement à ces cas résiduels.

### 12.4 Great Expectations

Plusieurs erreurs sont apparues lors de la réexécution du notebook, car Great Expectations conserve les objets déjà créés : datasource, asset, batch, suite et validation.

Le notebook a été corrigé pour récupérer les objets existants au lieu de les recréer systématiquement.

### 12.5 Organisation des fichiers

Plusieurs versions intermédiaires du notebook ont été produites pendant les tests.
Le dépôt final conserve uniquement la version stable :

```text
TP_Qualite_Donnees_IRVE_final.ipynb
```

Les anciennes versions, caches, environnements virtuels et fichiers temporaires sont exclus via `.gitignore`.

---

## 13. Synthèse finale

La synthèse finale du notebook est la suivante :

```text
SYNTHÈSE FINALE DU TP QUALITÉ DES DONNÉES IRVE
Nombre de lignes final : 160,255
Nombre de colonnes final : 73
Communes encore manquantes : 142
Complétude communes : 99.91 %
Validation Great Expectations : True
Statut monitoring : OK
```

---

## 14. Conclusion

Ce TP met en œuvre une démarche complète de qualité des données appliquée à un dataset IRVE.

Le travail a permis :

* d’identifier les principales anomalies ;
* de nettoyer les données ;
* d’enrichir les communes manquantes ;
* de valider automatiquement le dataset avec Great Expectations ;
* de générer une documentation de validation ;
* de proposer un dispositif de monitoring qualité.

La qualité des données s’améliore fortement entre J-n et J.
La complétude des communes passe de 60,43 % à 99,91 %, et la validation finale est réussie.

Le dataset final est donc plus fiable, mieux documenté et plus exploitable pour des analyses territoriales sur les infrastructures de recharge pour véhicules électriques.
