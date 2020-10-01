# 1
```
Schéma : Etoile
cf ...
```

# 2
``` sql
select localite, niveau.niveau, annee_naissance, notion.notion, count(idex) as nb_exo
from etablissement, prof, niveau, notion, exercice
where
exercice.idex = notion.idexo
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

# 3
```
...
```

# 4
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