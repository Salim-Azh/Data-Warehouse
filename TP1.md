# TP1 Data Warehouse

## 1

<img src="https://github.com/Salim-Azh/Data-Warehouse/blob/main/schema-star.png" width="50%">


## 2

``` sql
select localite, niveau.niveau, annee_naissance, notion.notion, count(idex) as nb_exo
from etablissement, prof, niveau, notion, exercice
where exercice.idex = notion.idexo
and exercice.proprietaire = prof.idp
and prof.rne = etablissement.rne
and exercice.niveau = niveau.niveau
group by localite, niveau.niveau, annee_naissance, notion.notion;
```

``` sql
select localite, niveau.niveau, annee_naissance, notion.notion, count(idex) as nb_exo
from etablissement, prof, niveau, notion, exercice
where
exercice.idex = notion.idexo
and exercice.proprietaire = prof.idp
and prof.rne = etablissement.rne
and exercice.niveau = niveau.niveau
group by cube(localite, niveau.niveau, annee_naissance, notion.notion);
```

``` sql
select localite, niveau.niveau, annee_naissance, notion.notion, count(idex) as nb_exo
from etablissement, prof, niveau, notion, exercice
where
    exercice.idex = notion.idexo
and exercice.proprietaire = prof.idp
and prof.rne = etablissement.rne
and exercice.niveau = niveau.niveau
group by rollup(localite, niveau.niveau, annee_naissance, notion.notion);
```

``` sql
select localite, niveau.niveau, annee_naissance, notion.notion, count(idex) as nb_exo
from etablissement, prof, niveau, notion, exercice
where
exercice.idex = notion.idexo
and exercice.proprietaire = prof.idp
and prof.rne = etablissement.rne
and exercice.niveau = niveau.niveau
group by grouping sets((localite, niveau.niveau, annee_naissance, notion.notion), ());
```

```sql
select
    decode(localite, null, 'Toutes localités') as localite,
    niveau.niveau,
    annee_naissance,
    decode(notion.notion, null, 'Toutes notions confondues'),
    count(idex) as nb_exo
from etablissement, prof, niveau, notion, exercice
where
exercice.idex = notion.idexo
and exercice.proprietaire = prof.idp
and prof.rne = etablissement.rne
and exercice.niveau = niveau.niveau
group by cube(localite, niveau.niveau, annee_naissance, notion.notion)
```

## 3

```text
...
```

## 4

```sql
select
    decode(localite, null, 'Toutes localités') as localite,
    niveau.niveau,
    annee_naissance,
    decode(notion.notion, null, 'Toutes notions confondues') as notion,
    count(idex) as nb_exo,
    rank() over (order by count(*) desc) as rank
from etablissement, prof, niveau, notion, exercice
where
exercice.idex = notion.idexo
and exercice.proprietaire = prof.idp
and prof.rne = etablissement.rne
and exercice.niveau = niveau.niveau
group by cube(localite, niveau.niveau, annee_naissance, notion.notion);

```

```sql
select eleve.idel, eleve.niveau, count(*) as nb_expass, ratio_to_report(count(*)) over (partition by eleve.niveau) as ratio
from eleve, passage, devoir, exercice, contenu
where
    eleve.idel = passage.idel
and passage.idd = devoir.idd
and contenu.idd = devoir.idd
and contenu.idex = exercice.idex
and exercice.niveau = eleve.niveau
group by eleve.idel, eleve.niveau
order by eleve.idel;
```

```sql
SELECT niveau.niveau,COUNT(*) AS nb_exercice_niveau,SUM(COUNT(*)) OVER (ORDER BY niveau.niveau DESC) AS cumul
FROM exercice,niveau
WHERE
    exercice.niveau = niveau.niveau
GROUP BY niveau.niveau
```

```sql
select devoir.niveau as d_niveau, exercice.niveau as ex_niveau, count(*) as nb
from exercice, contenu, devoir
where
    exercice.idex = contenu.idex
and devoir.idd = contenu.idd
and devoir.niveau <> exercice.niveau
group by grouping sets((devoir.niveau, exercice.niveau), ())
```

## 5

```sql
select *
from (select
        decode(localite, null, 'Toutes localités') as localite,
        niveau.niveau,
        annee_naissance,
        decode(notion.notion, null, 'Toutes notions confondues') as notion,
        count(idex) as nb_exo,
        rank() over (order by count(*) desc) as rank
    from etablissement, prof, niveau, notion, exercice
    where
    exercice.idex = notion.idexo
    and exercice.proprietaire = prof.idp
    and prof.rne = etablissement.rne
    and exercice.niveau = niveau.niveau
    group by cube(localite, niveau.niveau, annee_naissance, notion.notion)
    )
where rank<3;
```

```sql
select distinct *
from (select
        decode(localite, null, 'Toutes localités') as localite,
        niveau.niveau,
        annee_naissance,
        decode(notion.notion, null, 'Toutes notions confondues') as notion,
        count(idex) as nb_exo,
        rank() over (order by count(*)) as rank
    from etablissement, prof, niveau, notion, exercice
    where exercice.idex = notion.idexo
    and exercice.proprietaire = prof.idp
    and prof.rne = etablissement.rne
    and exercice.niveau = niveau.niveau
    group by cube(localite, niveau.niveau, annee_naissance, notion.notion)
    )
where rank<3
FETCH FIRST 2 ROWS ONLY;
```

```sql
select notion, nb_exo, rank
from (select
        localite,
        exercice.niveau,
        annee_naissance,
        notion.notion as notion,
        count(idex) as nb_exo,
        rank() over (order by count(*) desc) as rank
    from etablissement, prof, niveau, notion, exercice
    where exercice.idex = notion.idexo
    and exercice.proprietaire = prof.idp
    and prof.rne = etablissement.rne
    group by grouping sets((localite, exercice.niveau, annee_naissance, notion.notion),())
    )
where rank<3
group by notion, nb_exo, rank;
```

## 6

```text
Les vues materialisées permettent de masquer l'organisation des données à l'utilisateur.
```

```sql
CREATE MATERIALIZED VIEW OLAPV_EXO
ENABLE QUERY REWRITE
AS
SELECT localite, exercice.niveau,annee_naissance,notion.notion, COUNT(idex) AS nb_exo
FROM etablissement, prof,notion, exercice
WHERE exercice.idex = notion.idexo
AND exercice.proprietaire = prof.idp
AND prof.rne = etablissement.rne
GROUP BY localite, exercice.niveau,annee_naissance,notion.notion;
```

```sql
/*
On cree la vue materialisé en utilisant les clause
REFRESH ON DEMAND et
START WITH ... NEXT ...
*/
CREATE MATERIALIZED VIEW OLAPV_EXO
REFRESH ON DEMAND
start with sysdate next sysdate + 1
ENABLE QUERY REWRITE
AS
/*...*/

/*
Si on veux rafraichir une fois par jour, on doit specifier refresh on demand et fixer le prochain rafraichissement
en tant que*/ sysdate + 1
*/

```

## 7

```text
Slow Changing Dimension : Dimension qui stocke et gère les données actuelles et historiques au fil du temps dans un entrepôt de données.

Dans le cadre des données de bibliothèques, il s'agit de stocker qui emprunte le livre.

Solutions :
- Écrasement de l’ancienne valeur / remplacer
- Création d'une nouvelle ligne
- Création d'une nouvelle colonne
- Valeur d’origine / valeur courante
- hybride

Si l'on choisit de garder une trace des livres empruntés, il faudra utilisé la solution de création de nouvelle ligne en ajoutant les dates d'emprunts.
Si l'on choisit de ne pas garder une trace des livres emprutés, il faudra plutôt écraser l'ancien emprunteur et le remplacer par le nouveau.
```

## 8

```text
Un ETL est un processus qui extrait des données sources les transforment dans des formats demandés et qui finalement charge les données formatée dans un Data Warehouse.
```

```text
L'étape de transformation consiste à exécuter une série de fonctions et à appliquer des ensembles de règles aux données extraites, pour les convertir dans un format standard afin de répondre aux exigences de schéma de la base de données cible.

Les différents transformations sont :
* Convertir (entre différentes unités euros/ dollards, metres/feet)

* Reformater (remplacer les données vide par null)

* Trier ou filtrer (Par ville ou autres)

* Effacer les doublons

* Traduire d'une langue à l'autre
```

```text
Business Object Data Integrator:
Data Integrator, intégrée ou non à la suite Business Objects XI, dispose d'une fonctionnalité de centralisation des metadonnées et un module complet de qualité des données (normalisation, correction...).

Data Manager:
Les metadonnées sont gérées en central et stockées dans un SGBDR tiers. L'extraction des données s'effectue par batch et plusieurs connecteurs ERP sont disponibles en natif (PeopleSoft, SAP et Oracle).

Clover.ETL:
En gérant les processus légers (threads) de façon indépendante, Clover.ETL partage la charge de transformation sur différents processeurs et s'adapte ainsi aux multiples configurations matérielles

Sybase ETL:
Le module Sybase ETL est commercialisé indépendamment de Data Integration Suite, comprenant en outre des briques de réplication et de fédération de données hétérogènes


```