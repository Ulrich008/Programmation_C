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
/*  for (int i = 0; i < nb_lignes; i++) {
        coef[i] = mat_calloc((size_t)nb_colonnes, sizeof *coef[i]);    // nb_colonnes entiers
        if (coef[i] == NULL) {
            for (int k = i - 1; k >= 0; k--) {
                free(coef[k]); // lignes déjà obtenues
            } 
            free(coef);                       // puis le tableau
            return MAT_ERR_ALLOCATION;   
        }                                                           
    } */
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

### Exercice 2 : Entrées-sorties
#### Code écrit
* **2.a :**
```c
for (int i = 0; i < m->nb_lignes; i++) {
    for (int j = 0; j < m->nb_colonnes; j++)
        printf(" %8d", m->coef[i][j]);
    putchar('\n');
}
```
* **2.b :**
```c
for (int i = 0; i < m->nb_lignes; i++) {
    for (int j = 0; j < m->nb_colonnes; j++) {
        if (fprintf(f, j == 0 ? "%d" : " %d", m->coef[i][j]) < 0)
            return MAT_ERR_ECRITURE;
    }
    if (fputc('\n', f) == EOF)
        return MAT_ERR_ECRITURE;
}
```
* **2.c :**
```c
if (nb_lignes < 1 || nb_lignes > MAT_DIM_MAX
    || nb_colonnes < 1 || nb_colonnes > MAT_DIM_MAX)
    return MAT_ERR_FORMAT;
```
* **2.d :**
```c
Matrice tmp;
err = mat_allouer(&tmp, nb_lignes, nb_colonnes);
if (err != MAT_OK)
    return err;
```
* **2.e :**
```c
for (int i = 0; i < nb_lignes; i++) {
    for (int j = 0; j < nb_colonnes; j++) {
        err = lire_entier(f, &tmp.coef[i][j]);
        if (err != MAT_OK) {
            mat_detruire(&tmp);
            return err;
        }
    }
}
```
* **2.f :**
```c
*res = tmp;
return MAT_OK;
```

#### Réponses aux questions
* **Q4.1 :** mat_lire remplit d'abord une matrice locale tmp avant de l'affecter à *res pour garantir la règle R3. Si une erreur survient pendant la lecture des coefficients (fichier tronqué, donnée invalide), la fonction peut libérer tmp proprement sans laisser *res dans un état partiellement construit. L'appelant reçoit soit une matrice complète et valide, soit MATRICE_VIDE en cas d'échec.
* **Q4.2 :** Non, la lecture ne vérifie pas que chaque ligne de la matrice est sur sa propre ligne de texte. La fonction lire_entier saute tous les blancs (espaces, tabulations, sauts de ligne) grâce à fscanf("%31s", mot). Ce n'est pas gênant car le format du fichier est tolérant en lecture : seule la séquence des entiers importe. En revanche, l'écriture est stricte (format canonique) pour garantir que le fichier produit est lisible et modifiable par un humain.
* **Q4.3 :** Après mat_lire(f, &m), le flux f est positionné juste après le dernier coefficient. On peut donc stocker plusieurs matrices dans un même fichier en les écrivant successivement, puis en les relisant avec plusieurs appels consécutifs à mat_lire sur le même flux ouvert. Par exemple, un fichier contenant "2 2\n1 2\n3 4\n3 3\n1 2 3\n4 5 6\n7 8 9\n" permet de lire deux matrices sans fermer le fichier.
* **Q4.4 :** mat_ecrire appelle fflush avant de renvoyer MAT_OK pour forcer l'écriture du tampon dans le fichier et détecter une erreur d'écriture (disque plein, par exemple). Cela ne dispense pas l'appelant de tester fclose : une erreur peut encore survenir lors de la fermeture du fichier (mise à jour des métadonnées du système de fichiers). La fonction sauvegarder dans main.c teste donc fclose et renvoie MAT_ERR_ECRITURE si nécessaire.

### Sortie Du Test (test_ex2)
```text
=== Exercice 2 : affichage (verification visuelle) ===
M (4 x 3) :
       1        2        3
       4        5        6
       7        8        9
      10       11       12
(2.a) Vous devez voir ci-dessus 4 lignes : 1 2 3 / 4 5 6 / 7 8 9 / 10 11 12
=== Exercice 2 : ecriture ===
[OK]    ouverture de test_ex2_sortie.txt en ecriture
[OK]    (2.b) mat_ecrire renvoie MAT_OK
[OK]    (2.b) le fichier contient exactement le format attendu
[OK]    (2.b) mat_ecrire refuse une matrice vide
=== Exercice 2 : relecture ===
[OK]    (2.f) mat_lire relit le fichier : MAT_OK
[OK]    (2.e) la matrice relue est identique
[OK]    (2.e) espaces et sauts de ligne en trop sont toleres
=== Exercice 2 : fichiers incorrects ===
[OK]    (2.c) fichier vide
[OK]    (2.c) dimension nulle
[OK]    (2.c) dimension negative
[OK]    (2.e) coefficient manquant
[OK]    (2.e) donnee non numerique
[OK]    (2.e) entier trop grand pour un int
=== Exercice 2 : echec d'allocation simule ===
[OK]    (2.d) mat_lire renvoie MAT_ERR_ALLOCATION proprement

Bilan : 14 / 14 verifications reussies.
```



