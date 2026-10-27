# TD : Stockage de matrices en c

* **Membres du groupe :**
    * ZINOUBI Mohamed Iyadh (@iyadhzinoubi-byte)
    * SKHIRI Zeineb (@zeinebskhiri72-del)
    * ASSOGBA Ulrich (@Ulrich008)
* **Groupe de TD :** TD 3
* **Date de remise :**
## Table de matières
- [1. Introduction](#1-introduction)
- [2. Conception](#2-conception)
- [3. Réalisation](#3-réalisation)
- [4. Vérifications mémoire](#4-vérifications-mémoire)
- [5. Démonstration finale](#5-démonstration-finale)
- [6. Bilan et difficultés](#6-bilan-et-difficultés)

---
## 1. Introduction
Ce projet consiste à concevoir une bibliothèque en langage C pour la manipulation de matrices d'entiers allouées dynamiquement. Le code est découpé de manière modulaire afin d'isoler la gestion bas niveau de la mémoire, les opérations d'entrées-sorties dans des fichiers et l'arithmétique matricielle.
**Statut du projet à la remise :** 

## 2. Conception
### Schéma des modules
Le programme principal `main.c` interagit avec les modules spécialisés selon la hiérarchie ci-dessous : 
```text
                     ┌─────────────────┐
                     │     main.c      │  (Programme principal)
                     └────────┬────────┘
                              │
            ┌─────────────────┴─────────────────┐
            ▼                                   ▼
  ┌──────────────────┐                ┌──────────────────┐
  │  matrice_calcul  │                │    matrice_io    │  (Calculs & Fichiers)
  └────────┬─────────┘                └────────┬─────────┘
           │                                   │
           └─────────────────┬─────────────────┘
                             ▼
                   ┌──────────────────┐
                   │  matrice_alloc   │                   (Allocation & Libération)
                   └────────┬─────────┘
                            ▼
                   ┌──────────────────┐
                   │     matrice      │                   (Structure & Types de base)
                   └──────────────────┘
```
### Graphe des inclusions des fichiers en-têtes

Le schéma ci-dessous illustre la dépendance des fichiers du projet vis-à-vis des en-têtes (`.h`) :

```text
       ┌───────────┐      ┌─────────────────┐      ┌──────────────┐
       │  main.c   │      │ test_ex1..3.c   │      │  matrice.c   │
       └─────┬─────┘      └────────┬────────┘      └──────┬───────┘
             │                     │                      │
   ┌─────────┼─────────┐           │                      │
   ▼         ▼         ▼           │                      │
┌──────┐ ┌──────┐ ┌────────┐       │                      │
│.calcul││ .io  │ │ .alloc │       │                      │
└──┬───┘ └──┬───┘ └───┬────┘       │                      │
   │        │         │            │                      │
   └────────┴────┬────┴────────────┴──────────────────────┘
                 │ (includes "matrice.h")
                 ▼
          ┌──────────────┐
          │  matrice.h   │  (Contrat d'API, Matrice, MatStatut)
          └──────────────┘
```
## 3. Realisation

### Exercice 1 : Allocation et destruction
#### Code écrit
* **1.a :** 
/* case MAT_ERR_DIMENSIONS: return "dimensions incompatibles";
   case MAT_ERR_DEPASSEMENT: return "dépassement de capacité" ;
*/
* **1.b**
/* return m!=NULL && m->coef!=NULL && 1 <= m->nb_lignes && m->nb_lignes <= MAT_DIM_MAX && 1 <= m->nb_colonnes && m->nb_colonnes <= MAT_DIM_MAX ; */
* **1.c**
/*  if ( nb_lignes < 1 || nb_lignes > MAT_DIM_MAX || nb_colonnes < 1 || nb_colonnes > MAT_DIM_MAX ) {
        return MAT_ERR_ARGUMENT ;
    }
*/
* **1.d** 
/* for (int i = 0; i < nb_lignes; i++) {
        coef[i] = mat_calloc((size_t)nb_colonnes, sizeof *coef[i]);    // nb_colonnes entiers
        if (coef[i] == NULL) {
            for (int k = i - 1; k >= 0; k--) {
                free(coef[k]); // lignes déjà obtenues
            } 
            free(coef);                       // puis le tableau
            return MAT_ERR_ALLOCATION;
            
        }                                                           
    }
   */
```text
coef (tableau alloué)
┌──────────┬──────────┬──────────┐
│ coef[0]  │ coef[1]  │ coef[2]  │
└────┬─────┴────┬─────┴────┬─────┘
     │          │          │
     ▼          ▼          X (mat_calloc renvoie NULL)
  [Ligne 0]  [Ligne 1]  (échec)
```
* **1.e**
*/
res->coef = coef;
    res->nb_lignes = nb_lignes;
    res->nb_colonnes = nb_colonnes;
    return MAT_OK; 
    */
* **1.f**
/* if (m->coef != NULL){
        for(int i=0 ; i< m->nb_lignes ; i++){
            free(m->coef[i]);
        }
        free(m->coef) ;
    }
*/

#### Réponses aux questions
* **Q3.1 :** Passer Matrice *m permet de réinitialiser la structure de l'appelant à MATRICE_VIDE (*m = MATRICE_VIDE), respectant la règle R6. En passant par valeur (Matrice m), la fonction ne modifierait qu'une copie locale : les lignes seraient libérées sur le tas, mais l'appelant conserverait des pointeurs "pendants" (dangling pointers), ce que nous avons vérifié lors de nos tests.
* **Q3.2 :** Pour une matrice n x p créée avec succès, il y a 1 + n appels à calloc (1 pour le tableau de pointeurs + n pour chaque ligne) et 1 + n appels à free lors de sa destruction. Si la k-ième ligne échoue lors de l'allocation, exactement k + 1 libérations (free) sont exécutées (k lignes + le tableau coef).
* **Q3.3 :** Appeler deux fois de suite mat_detruire(&a) est sans danger car le premier appel réinitialise a.coef à NULL , le second appel est immédiatement stoppé par la condition if (m->coef != NULL). Cependant, avec la copie Matrice b = a, l'adresse mémoire est dupliquée. Si on détruit a, b.coef pointe sur de la mémoire libérée. Un appel ultérieur à mat_detruire(&b) provoque un plantage critique
* **Q3.4 :** L'affectation *res = MATRICE_VIDE en début de fonction garantit le respect de la règle R3. Si la fonction échoue prématurément (arguments invalides ou échec mémoire), l'appelant est certain de récupérer une structure neutre et sécurisée.

### Sortie Du Test (test_ex1)
```text
=== Exercice 1 : matrice.c ===
[OK]    (1.a) message pour MAT_ERR_DIMENSIONS
[OK]    (1.a) message pour MAT_ERR_DEPASSEMENT
[OK]    (1.b) une matrice 1 x 2 est valide
[OK]    (1.b) NULL n'est pas valide
[OK]    (1.b) MATRICE_VIDE n'est pas valide
[OK]    (1.b) une matrice de 0 ligne n'est pas valide
=== Exercice 1 : matrice_alloc.c ===
[OK]    (1.c) mat_allouer refuse 0 ligne
[OK]    (1.c) mat_allouer refuse -2 colonnes
[OK]    (1.c) mat_allouer refuse une dimension > MAT_DIM_MAX
[OK]    (1.e) mat_allouer(&m, 3, 4) renvoie MAT_OK
[OK]    (1.e) m a 3 lignes et 4 colonnes
[OK]    (1.d) tous les coefficients valent 0
[OK]    (1.f) apres mat_detruire, m est vide
[OK]    (1.f) un second mat_detruire est sans danger
[OK]    (1.f) mat_detruire(NULL) ne plante pas
=== Exercice 1 : echecs d'allocation simules ===
[OK]    (1.d) chaque echec renvoie MAT_ERR_ALLOCATION et laisse *res vide

Bilan : 16 / 16 verifications reussies.
```


