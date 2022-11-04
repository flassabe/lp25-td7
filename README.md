# LP25 - C TD n°2

Ce TD comporte deux parties :

- une brève initiation à l'utilisation de la commande make et l'écriture de fichiers Makefile pour définir les règles de compilation.
- l'utilisation de la mémoire dynamique.

## Travail préalable

Écrire un programme nommé **swap.c** qui comporte une fonction dont la signature est `void swap(int *a, int *b)` et qui permet d'échanger les valeurs contenues par les variables `a` et `b`. Les `*` dans la signature signifient que les variables a et b sont des pointeurs, donc des adresses mémoire. En déréférençant le pointeur `a` grâce à la syntaxe `*a`, on obtient la valeur de la variable pointée. Par exemple :

```c
int main() {
	int a = 5;
	int *p = &a; // p est un pointeur qui pointe sur a, donc son adresse est &a
	printf("%p\n", p); // Affiche 0x.... l'adresse de a en mémoire
	printf("%d\n", *p); // Affiche la valeur de a : 5
	int b = *p * 2; // b est égal à 2*a, donc 10
	return 0;
}
```

## Compilation séparée

Avant de se lancer dans l'utilisation de `make` et de l'écriture des `Makefile`, commençons par la compilation séparée en elle-même.

Il est possible et même souhaitable de décomposer un code complexe en plusieurs fichiers de code. Plusieurs impacts positifs à cette décomposition existent, parmi lesquels :

 - Vitesse de compilation : seuls les fichiers ayant été modifiés ont besoin d'être compilés
 - Modularité : en décomposant votre code en fichiers selon une logique de fonctionnalités ou de rôles, vous en facilitez la compréhension : il suffit de lire les entêtes pour utiliser une autre unité de compilation, seul le code pertinent au moment où vous y travaillez vous est visible.
 - Travail en équipe : en se répartissant les tâches sur des fichiers différents, vous réduisez les conflits lors de la fusion de vos travaux respectifs.

Le travail qui suit est basé sur l'exemple suivant :

swap.h :
```c
#ifndef _SWAP_H_
#define _SWAP_H_

void swap(int *a, int *b);

#endif //_SWAP_H_
```

swap.c :
```c
#include "swap.h"

void swap(int *a, int *b) {
	int tmp = *a;
	*a = *b;
	*b = tmp;
}
```

main.c :
```c
#include "swap.h"

int main() {
	int a=5, b=6;
	swap(&a, &b);
	return 0;
}
```

Dans cet exemple, `swap.h` expose la fonction `swap` à ses utilisateurs. `swap.c` définit le comportement de la fonction `swap`, et la fonction `main` utilise la fonction `swap`. Pour utiliser cette fonction, le fichier `main.c` doit notamment include le fichier d'entête `swap.h`.

Dans les étapes de la compilation, nous allons donc nous intéresser à la compilation proprement dite et à l'édition de liens (le reste des étapes étant moins concerné par la compilation séparée).

Tout d'abord, il faut générer le fichier objet du code des fichiers `swap.h` et `swap.c`. On utilise pour cela l'option `-c` de gcc, et on lui donne un fichier de sortie (option `-o`) suffixé par un `.o`. Puis on fait de même avec le main. On peut alors procéder à l'édition des liens, consistant ici à permettra au compilateur de lier l'appel à `swap` fait dans la fonction `main` à son adresse (obtenue par sa position dans le fichier binaire final). On procède à l'ensemble de ces opération avec les commandes suivantes :

```bash
gcc -c -o swap.o swap.c
gcc -c -o main.o main.c
gcc -o main main.o swap.o
```

**Pratique**

Tester l'exemple ci dessus puis parcourir et essayer de comprendre ce que vous retourne l'exécution de la commande `objdump -d` sur `swap.o`, `main.o` et `main`. 

**Exercice 1**

Séparer de la même manière que l'exemple, le code de résolution du polynôme du TP précédent, de celui du main.

## make et les Makefile

`make` est une commande permettant la compilation séparée grâce à des règles écrites dans un fichier nommé par défault `Makefile`. Ces règles peuvent être dédiées à un fichier, ou bien représenter des traitements suivant des motifs.

### Règles

Une règle est de la forme :
```
nom: dépendances
	commande
```
Par exemple, l'édition des liens de l'exemple précédent peut s'écrire de la manière suivante :

```
main: main.o swap.o
	gcc -o main main.o swap.o
```

Les dépendances signifient que la cible doit être construite si une de ses dépendances a changé.

**Exercice 2**

Écrire les règles pour produire les fichiers objet.

### Règles génériques

Imaginons maintenant que le main dépende de 10 autres fichiers, tous indépendants les uns des autres par ailleurs. Comment éviter de répéter la règle permettant de compiler le fichier objet ?

Il est possible d'utiliser des motifs dont :

 - `%` dans une cible ou une dépendance, remplace une séquence de caractères. Par exemple `%.o` signifie tous les .o
 - `$@` est le nom de la cible dans la commande qui lui correspond
 - `$<` est la liste de fichiers de dépendance de la règle du makefile

**Exercice 3**

Modifiez votre Makefile pour le rendre générique pour la compilation des fichiers objet. Testez le et pousser le sur le dépôt git.

**Observation**

Supprimer les fichiers `main`, `main.o` et `swap.o`, puis lancez la commande `make main` et observez ce qu'il se passe.

### Paramétrage du Makefile

Vous pouvez également définir des variables dans votre Makefile pour faciliter son écriture et son évolution. Une variable se définit comme en shell `NOM_VAR=value_var`. Ces variables sont ensuite utilisées avec la syntaxe `$(NOM_VAR)`. Il est courant de rendre paramétrables le compilateur, les options de compilation, ainsi que les chemins de production des fichiers.

**Exercice 4**

En utilisant les variables, paramétrer le Makefile pour que les fichiers objet soient créés dans un répertoire nommé `objects` et que le fichier binaire soit créé dans le répertoire `bin`. Poussez cette dernière version du Makefile dans le dépôt git.

## Les pointeurs

Les pointeurs sont un type de variable particulier, dont la valeur est une adresse à laquelle on trouvera une donnée du type pointé, ou une séquence de données du type pointé.

Deux notions sont importantes avec les pointeurs :

- l'acquisition de la mémoire d'une variable
- le déréférencement du pointeur (l'accès à la variable pointée)

L'acquisition d'un pointeur se fait de 3 manières :

- avec l'opérateur `&` sur une variable
- avec le retour de la fonction `malloc`/`realloc`
- avec l'identifiant d'un tableau (un tableau est assimilable à un pointeur)

Le pointeur de valeur `NULL` permet de définir un pointeur qui n'a actuellement aucune cible. Déréférencer ce pointeur (i.e. accéder à la mémoire pointée) provoque une erreur de segmentation.

Il est nécessaire de manipuler les pointeurs avec prudence car ils peuvent mener à des défaillances (crash) des programmes, ainsi qu'à une saturation de la mémoire. À cet effet, il est conseillé :

- de limiter l'accès à la mémoire dynamique au strict nécessaire (séquences de taille non connues à l'avance, variables créées dans un scope et transférées à un autre scope)
- de vérifier le déréférencement des pointeurs avant leur usage. Notamment, il est important de se poser la question suivante au début d'une fonction recevant des paramètres sous forme de pointeurs : *"Que se passe-t-il si tout ou partie des pointeurs est `NULL` ?"*

### Exercice 1

Dans le code suivant, trouver l'erreur potentielle.

```c
char *remove_leading_spaces(char *str) {
	while (isspace(*str))
		++str;
	return str;
}
```

### Exercice 2

Dans ce code, que pensez-vous de l'usage de la fonction `malloc` ?

```c
int generate_code() {
	int *tab = malloc(sizeof(int) * 4);
	int code = 0;
	for (int i=0; i<4; ++i)
		tab[i] = rand();
	code = (code << 8) ^ (tab[i] >> 8);
	return code;
}
```

### Exercice 3

Écrire un programme `dyn_alloc.c` qui prend en paramètre une valeur numérique entière que l'on appellera `n` (vérifier la validité du paramètre pour le convertir), qui crée un tableau d'entiers de taille égale à `n`, et le remplit des valeurs `1` à `n`.

### Exercice 4

Écrire un programme nommé `split.c` qui prend en paramètre une chaîne de caractères au format CSV (des valeurs séparées par des points-virgule). Pour passer la chaîne de caractères au programme, si elle contient des espaces, l'encadrer par des doubles quotes.

Le programme doit séparer cette chaîne en un tableau de sous-chaînes délimitées par les `;`. Pour cela, le travail se décompose en 3 étapes :

 - parcourir la chaîne pour y compter le nombre `s` de séparateurs (i.e. le nombre de `;` contenus par la chaîne de caractères)
 - créer un tableau de `char *` d'une taille égale à `s+1`.
 - parcourir la chaîne pour en extraire les éléments entre les `;`. Vous pouvez utiliser l'arithmétique des pointeurs et la fonction `strncpy` pour réaliser cette étape.

Le programme doit afficher les éléments ligne par ligne et se terminer.

## Bilan

Dans ce TP, vous avez appris :

- la compilation séparée avec la génération de modules de code et leur compilation en `.o`
- l'écriture de règles de compilation dans un fichier `Makefile` pour automatiser les étapes de la compilation
- l'utilisation de la commande `make` pour exécuter les étapes de compilation définies par un `Makefile`
- la signification des pointeurs, c'est-à-dire des adresses en mémoire
- l'obtention de mémoire dynamiquement avec la fonction `malloc`
- la libération avec `free` de mémoire obtenue dynamiquement par `malloc`
- l'arithmétique des pointeurs
- la fonction `strncpy` pour copier une chaîne de caractères avec une limite de taille
