# TP OpenMP - Calcul Matriciel

- [TP OpenMP - Calcul Matriciel](#tp-openmp---calcul-matriciel)
  - [Avant propos](#avant-propos)
  - [0. Serveur Turing](#0-serveur-turing)
  - [1. OpenMP](#1-openmp)
  - [2. Prise en main](#2-prise-en-main)
  - [3. Calcul Matriciel](#3-calcul-matriciel)
    - [3.1 - Implémenter  un  programme  séquentiel, la  taille  du  problème  sera  passé  en  paramètre  sur  la ligne de commande. Vérifier les résultats obtenus.](#31---impl%c3%a9menter-un-programme-s%c3%a9quentiel-la-taille-du-probl%c3%a8me-sera-pass%c3%a9-en-param%c3%a8tre-sur-la-ligne-de-commande-v%c3%a9rifier-les-r%c3%a9sultats-obtenus)
  - [3.2 - Identifier  les  parties  du  programme  adaptées  à  la  parallélisation,  puis utiliser  les directives OpenMP pour réaliser cela](#32---identifier-les-parties-du-programme-adapt%c3%a9es-%c3%a0-la-parall%c3%a9lisation-puis-utiliser-les-directives-openmp-pour-r%c3%a9aliser-cela)
  - [3.3 - Pour  chaque  variable  concernée  par  la  parallélisation,  indiquer elledoitêtre  privéeou partagée](#33---pour-chaque-variable-concern%c3%a9e-par-la-parall%c3%a9lisation-indiquer-elledoit%c3%aatre-priv%c3%a9eou-partag%c3%a9e)
  - [3.4 - Vérifier  que  le  programme  parallèle  donne  des résultats corrects en  comparantses  résultats avec  ceux  du  programme  séquentiel,  et  en  faisant  varier  le  nombre  de  threads  et  la  taille  de matrices.](#34---v%c3%a9rifier-que-le-programme-parall%c3%a8le-donne-des-r%c3%a9sultats-corrects-en-comparantses-r%c3%a9sultats-avec-ceux-du-programme-s%c3%a9quentiel-et-en-faisant-varier-le-nombre-de-threads-et-la-taille-de-matrices)
  - [3.5 - Vérifier  la  répartition  du  calcul  entre  les  threads  en  utilisant  le  numéro  de  thread  employé pour chaque itération, cecipour différent type d’ordonnancement schedule (dynamic/static).](#35---v%c3%a9rifier-la-r%c3%a9partition-du-calcul-entre-les-threads-en-utilisant-le-num%c3%a9ro-de-thread-employ%c3%a9-pour-chaque-it%c3%a9ration-cecipour-diff%c3%a9rent-type-dordonnancement-schedule-dynamicstatic)
  - [3.6 - Mesurer letemps  d’exécutiondu  programme  parallèle (en  variant le  nombre  de  threads  et  la taille  des  matrices),  puiscalculer la  performance  dela  parallélizationen  utilisant le  facteur d’accélération du programme(voir l’annexe).](#36---mesurer-letemps-dex%c3%a9cutiondu-programme-parall%c3%a8le-en-variant-le-nombre-de-threads-et-la-taille-des-matrices-puiscalculer-la-performance-dela-parall%c3%a9lizationen-utilisant-le-facteur-dacc%c3%a9l%c3%a9ration-du-programmevoir-lannexe)

## Avant propos

Je met à votre disposition un cmake pour facilité la compilation. Pour l'utiliser

- `cd build`
- `cmake ..`
- `make`

Les 2 executables `hello_omp` et `matrice` sont générés

## 0. Serveur Turing

Lu 

## 1. OpenMP

Lu

## 2. Prise en main

Fichier hello_omp.c
```c
#include "omp.h"
#include <sched.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    int tid = -1;
    char hostname[1024];
    gethostname(hostname, 1024);
    tid = omp_get_thread_num();
    printf("BeforePARALLEL REGIONTID %d: There are %d threads on CPU %d of %s\n\n", tid, omp_get_num_threads(),
           sched_getcpu(), hostname);
#pragma omp parallel num_threads() firstprivate(tid)
    {
        tid = omp_get_thread_num();
        if (!tid)
            printf("In the PARALLEL REGIONTID %d: There are %d threads in process\n", omp_get_thread_num(),
                   omp_get_num_threads());
        printf("Hello World from TID %d / %d on CPU %d of %s!\n\n", tid, omp_get_num_threads(), sched_getcpu(),
               hostname);
    }

    printf("After PARALLEL REGIONTID %d: There are %d threads\n\n", tid, omp_get_num_threads());
    return EXIT_SUCCESS;
}
```

- Compiler  le  programme  avec  : gcc -fopenmp  hello_omp.c -o  hello_omp;  Vous  pouvez ajouter les options habituelles de compilation
    - Compilation du programme : `gcc -fopenmp  hello_omp.c -o  hello_omp`
- Exécution directe du programme hello_omp : `./hello_omp`
- Interpréter le résultat d’exécution: qui (thread) exécute quoi (instruction)?
    - On nous indique qu'il y a 4 threads 
    - A chaque fois on a un printf "Hello World", en indiquant le numéro du thread et le host (turing.local.isima.fr)
- Modifier le nombre de threads 
    - la clause *num_threads* de *#pragma omp parallel* : `#pragma omp parallel num_threads(8)`
    - la variable d’environnement OMP_NUM_THREADS : `#pragma omp parallel firstprivate(tid)` et on lance le programme avec `OMP_NUM_THREADS=8 ./hello_omp` 
    - la fonction *void omp_set_num_threads(int num_threads)*  : `omp_set_num_threads(8);` et `#pragma omp parallel firstprivate(tid)`
- Si on remplace la clause *firstprivate* par *private*, que peut-il se passer?
     - Le *firstprivate* permet d'amener une valeur qui vient d'en dehors du context vers l'intérieur de la région parallèle
     - Le *private* ne récupère la valeur de la variable, la variable n'est pas initialisé et n'aura donc pas forcément la valeur demandé. La valeur est constante
- Supprimer la clause private, refaire les exécutions, que constatez vous ?
    - La valeur est maintenant aléatoire

## 3. Calcul Matriciel

### 3.1 - Implémenter  un  programme  séquentiel, la  taille  du  problème  sera  passé  en  paramètre  sur  la ligne de commande. Vérifier les résultats obtenus. 

Le calcul de matrice est déjà fait dans le fichier `prodMatriciel_dynamic_seq.c`. Par conséquent on effectue la commande suivant :   
`gcc prodMatriciel_dynamic_seq.c -o  matrice && ./matrice 3`

On effectue donc le calcul matriciel des matrices 

<div style="display:flex; justify-content: space-around; align-items:center">
    <div>
        <table>
            <tr>
                <td>0</td>
                <td>1</td>
                <td>2</td>
            </tr>
            <tr>
                <td>3</td>
                <td>4</td>
                <td>5</td>
            </tr>
            <tr>
                <td>6</td>
                <td>7</td>
                <td>8</td>
            </tr>
        </table>
    </div>
    <div>x</div>
    <div>
        <table>
            <tr>
                <td>0</td>
                <td>1</td>
                <td>2</td>
            </tr>
            <tr>
                <td>1</td>
                <td>2</td>
                <td>3</td>
            </tr>
            <tr>
                <td>2</td>
                <td>3</td>
                <td>4</td>
            </tr>
        </table>
    </div>
    <div>=</div>
    <div>
        <table>
            <tr>
                <td>5</td>
                <td>8</td>
                <td>11</td>
            </tr>
            <tr>
                <td>14</td>
                <td>26</td>
                <td>38</td>
            </tr>
            <tr>
                <td>23</td>
                <td>44</td>
                <td>65</td>
            </tr>
        </table>
    </div>
</div>

Le résultat est correct

## 3.2 - Identifier  les  parties  du  programme  adaptées  à  la  parallélisation,  puis utiliser  les directives OpenMP pour réaliser cela

Tous les temps données ont été calcul en enlevant l'affichage des matrices pour gagner en rapidité, sur des matrices de taille 1000.  
Les parties du programme adaptés à la parallélisation sont
- Lors du calcul de la multiplication : On peut ajouter l'instruction `#pragma omp parallel for private(i, j, k) shared(A, B, C)` au dessus des 3 boucles for, pour réduire le temps considérablement. 
- Lors de l'initialisation des matrices avec l'instruction `#pragma omp parallel for private(i, j) shared(A, B)`. En revanche, l'amélioration est minime comparait à celle lors du calcul

## 3.3 - Pour  chaque  variable  concernée  par  la  parallélisation,  indiquer elledoitêtre  privéeou partagée

Les matrices doivent etre partagées entre tous les threads avec `shared(A, B, C)`, car ils vont calculer différentes lignes donc il n'y aurais pas de conflit.  
En revanche les indices doivent être privés avec `private(i, j, k)` car chaque threads a besoin de faire ses propres iterations sur les boucles

## 3.4 - Vérifier  que  le  programme  parallèle  donne  des résultats corrects en  comparantses  résultats avec  ceux  du  programme  séquentiel,  et  en  faisant  varier  le  nombre  de  threads  et  la  taille  de matrices.

Les résultats ont été obtenus en mettant :
- Tous les affichages en commentaire
- En utilisant `omp_set_num_threads(omp_get_num_procs());` provenant de la bibliothèque `#include "omp.h"` au début du programme pour avoir les résultats avec le maximum de threads possible pour le temps en parallèle (sur turing = 192)
- Les résultats sont correct sur des matrices de taille 5x5 pour le séquentiel et pour le parallèle

| Taille Matrice | Temps en séquentiel | Temps en parallèle |
| -------------- | ------------------- | ------------------ |
| 250x250        | 0.086               | 0.083              |
| 500x500        | 0.845               | 0.101              |
| 1000x1000      | 6.987               | 0.215              |
| 2000x2000      | 122.964             | 1.454              |

Plus la taille de la matrice augmente, plus le temps d'éxécution augmente. En revanche, il augmente énormément en séquentiel et très peu en parallèle

| Taille Matrice | Nombre de threads | Temps |
| -------------- | ----------------- | ----- |
| 1000x1000      | 1                 | 6.928 |
| 1000x1000      | 2                 | 3.689 |
| 1000x1000      | 4                 | 2.045 |
| 1000x1000      | 8                 | 1.005 |
| 1000x1000      | 192               | 0.212 |

Plus le nombre de threads augmentent, moins le temps d'éxécution est élevé.

Par conséquent, il faut priviligié le parallèle avec un nombre de coeur élevé

## 3.5 - Vérifier  la  répartition  du  calcul  entre  les  threads  en  utilisant  le  numéro  de  thread  employé pour chaque itération, cecipour différent type d’ordonnancement schedule (dynamic/static).

Pour savoir qu'elle thread à été utilisé, j'initialise un tableau qui va contenir le nombre de fois ou le thread a été utilisé
```c
    int* threads_used = NULL;
    int cpu_number = omp_get_num_procs(); // 192 
    threads_used = (int*)malloc(sizeof(int) * cpu_number);
    int z;
    for (z = 0; z < cpu_number; z++) {
        threads_used[z] = 0;
    }
```

Ensuite je change la boucle de calcul de la matrice 

```c
#pragma omp parallel private(i, j, k) shared(A, B, C)
{
    #pragma omp for schedule(dynamic) // static ou dynamic
    for (i = 0; i < SIZE; i++) {
        threads_used[omp_get_thread_num()]++; // Le thread utilsé
        for (j = 0; j < SIZE; j++) {
            C[i][j] = 0;
            for (k = 0; k < SIZE; k++) {
                C[i][j] += A[i][k] * B[k][j];
            }
        }
    }
}
```

Ensuite on affiche le résultat 

```c
for (z = 0; z < cpu_number; z++) {
    printf("CPU %d used %d times\n", z, threads_used[z]);
}
```

Observations (pour une matrice de taille 1000x1000):
- Avec le mot clé `static`, les threads sont répartis uniformément. Il  y a 192 coeurs donc `192 * 5 = 960`. Il y a donc 40 threads qui sont utilisé 6 fois, et les autres 5 fois
- Avec `dynamic`, la répartition est aléatoire. La moyenne est autour de 5 fois utilisé, mais on peut aller de 3 à 10

## 3.6 - Mesurer letemps  d’exécutiondu  programme  parallèle (en  variant le  nombre  de  threads  et  la taille  des  matrices),  puiscalculer la  performance  dela  parallélizationen  utilisant le  facteur d’accélération du programme(voir l’annexe).

| Taille Matrice | Nombre de thread | Static | Dynamic |
| -------------- | ---------------- | ------ | ------- |
| 250x250        | 192              | 0.91   | 0.86    |
| 500x500        | 192              | 0.101  | 0.95    |
| 1000x1000      | 192              | 0.214  | 0.189   |
| 2000x2000      | 192              | 1.401  | 1.354   |

On observe donc que plus la taille de la matrice augmente, plus les temps augmentes. En revanche, le dynamic est toujours plus rapide

| Taille Matrice | Nombre de thread | Static | Dynamic |
| -------------- | ---------------- | ------ | ------- |
| 1000x1000      | 1                | 7.300  | 7.298   |
| 1000x1000      | 2                | 3.711  | 3.662   |
| 1000x1000      | 4                | 1.915  | 1.854   |
| 1000x1000      | 8                | 1.014  | 0.945   |

On observe donc que plus le nombre de thread augmente, plus les temps augmentes. En revanche, le dynamic est toujours plus rapide

Par conséquent, il faut privilégié le mot clé `dynamic` et un nombre de thread élevé