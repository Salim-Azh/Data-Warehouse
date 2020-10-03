# TP1 Data Warehouse

## 1

```text
Schéma : Etoile
cf ...
```

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
select exercice.niveau, count(*), sum(count(*)) as cumul
from exercice
group by exercice.niveau
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

## 6

```text
Les vues materialisées permettent de masquer l'organisation des données à l'utilisateur.
```

## 7

```text
Slow Changing Dimension : Il s'agit d'une dimensions qui va évoluer
lentement dans le temps.
```

## 8

```text
C'est un processus qui extrait des données sources les transforment 
dans des formats demandés et qui finalement charge les données formatée 
dans un DW.
```
