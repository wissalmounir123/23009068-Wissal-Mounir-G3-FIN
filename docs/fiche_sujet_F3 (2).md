# Fiche du sujet F3 : Prévision de l'inflation

**Étudiante :** Wissal MOUNIR · **Code Apogée :** 23009068 · **Groupe :** G3 · **Filière :** S7 FIN
**Dépôt :** https://github.com/wissalmounir123/23009068-Wissal-Mounir-G3-FIN.git

## Question
Sur les données mensuelles américaines, le chômage et le taux directeur réduisent-ils l'erreur de prévision de l'inflation d'un an sur l'autre, 12 mois à l'avance, par rapport à un modèle qui n'utilise que l'inflation passée ?

## Type de tâche
Régression sur séries mensuelles (apprentissage supervisé). Algorithmes compatibles du support : 1, 2, 3, 6, 7, 16.
**Algorithme principal retenu : Ridge (n°2).** Comparaison : gradient boosting (n°7).

## Données
| Série | Description | Source | Unité |
| --- | --- | --- | --- |
| CPIAUCSL | Indice des prix à la consommation, désaisonnalisé (sert à calculer l'inflation) | https://fred.stlouisfed.org/series/CPIAUCSL | indice (1982-1984 = 100) |
| UNRATE | Taux de chômage | https://fred.stlouisfed.org/series/UNRATE | % de la population active |
| FEDFUNDS | Taux des fonds fédéraux effectif (taux directeur) | https://fred.stlouisfed.org/series/FEDFUNDS | % |

- Téléchargement : `https://fred.stlouisfed.org/graph/fredgraph.csv?id=<SÉRIE>`
- Date d'extraction : 17/11/2026
- Fréquence mensuelle, janvier 1947 à octobre 2026 (958 lignes brutes) ; 855 mois alignés (juillet 1955 à octobre 2026).
- Unité d'observation : un mois.
- Licence : conditions FRED, https://fred.stlouisfed.org/legal/ ; usage pédagogique avec citation de la source.

## Prétraitement
1. Aligner les trois séries sur les mois communs.
2. Calculer l'inflation annuelle : π_t = 100 × (CPI_t / CPI_(t-12) − 1).
3. Construire les retards (t, t−1, t−3, t−6, t−12) avec `shift(k)`, k positif.
4. Définir la cible y_t = π_(t+12) avec `shift(-12)`, puis retirer les lignes incomplètes (831 observations).
5. Découpage chronologique avec un trou de 12 mois (CUTOFF = décembre 2010), puis standardisation dans un `Pipeline` (ajustée sur l'apprentissage seulement).

## Modèles
- **Référence :** « dernière valeur » (l'inflation dans 12 mois sera égale à celle d'aujourd'hui).
- **Principal :** Ridge, α réglé par `GridSearchCV` (20 valeurs entre 10⁻⁴ et 10⁴).
- **Comparaison :** gradient boosting.
- Deux jeux de variables pour chaque modèle : inflation seule, puis inflation + chômage + taux directeur.

## Métriques et protocole
- Métrique principale : RMSE de test, en points d'inflation. Secondaires : MAE et R².
- Validation : `TimeSeriesSplit`, 5 plis, `gap = 12`.
- Test : origines de janvier 2011 à septembre 2025 (177 observations), utilisé une seule fois.
- Seuil de réussite : RMSE du Ridge inférieur d'au moins 10 % à celui de la référence, et Ridge complet meilleur que Ridge sur l'inflation seule.

## Risques et points d'attention
- **Séries révisées après publication** : biais d'anticipation (look-ahead). Les versions actuelles des séries sont utilisées ; l'archive ALFRED (https://alfred.stlouisfed.org/) conserve les versions publiées à chaque date.
- Changements de régime (années 1970-1980, après 2008, 2020-2022) : relation instable entre les variables.
- Un mois absent (octobre 2025) à l'intérieur de la période alignée : les décalages comptent des lignes, pas des mois.
- Mois consécutifs très dépendants : peu d'observations réellement indépendantes.
- Un seul pays ; un seul découpage principal (à confirmer avec d'autres CUTOFF).
- Fuite de données : toutes les règles de la section 5.2 de l'énoncé s'appliquent.
