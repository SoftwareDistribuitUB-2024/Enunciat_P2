# Sessió de prova
En aquesta sessió realitzarem les proves creuades entre els diferents grups ens centrarem en la realització proves automàtiques del codi del backend.

## Objectius
* Provar el funcionament de la nostra pràctica
* Veure el resultat dels companys

## Exercicis al laboratori

### Exercici 1: Preparació de la fase de proves

Assegura't que vas realitzar l'exercici 4 de la sessió passada, que generava la documentació del teu projecte utilitzant
GitHub Pages. Assegura't que pots veure la documentació a l'URL:

https://softwaredistribuitub-2024.github.io/practica2-XYY/

Si no ho has fet, **afegeix a l'inici** de la documentació, l'URL del front-end i back-end desplegats a Render.com.

### Exercici 2: Dades de prova

Per tal de facilitar les proves durant la sessió, assegura't que a la teva aplicació hi ha **com a mínim** les dades 
de les següents taules, afegint els valors que vulgueu als camps que no es proporcionen:

#### Usuaris

| email                           | password         | is_superuser |
|---------------------------------|------------------|--------------|
| test_admin@sd.ub.edu            | .test_1dm3n!     | 1            |
| test_user@sd.ub.edu             | .test_5s2r!      | 0            |
| test_rich_user@sd.ub.edu        | .test_r3ch!      | 0            |
| test_poor_user@sd.ub.edu        | .test_p44r!      | 0            |
| test_organizer_user@sd.ub.edu   | .test_4rg1n3z2r! | 0            |


#### Account

Assigna un saldo disponible als següents dos usuaris.

| email                    | saldo   |
|--------------------------|---------|
| test_rich_user@sd.ub.edu | 1000000 |
| test_poor_user@sd.ub.edu | 50      |


#### Competicions

Afegeix les següents competicions. L'identificador només s'adjunta com a referència (IdRef), vosaltres tindreu els vostres.
L'organitzador de les dues competicions ha de ser l'usuari **test_organizer_user@sd.ub.edu**.

| IdRef* | name                 | category | sport      |
|--------|----------------------|----------|------------|
| C1     | Paris Olympic Games  | Senior   | Basketball |
| C2     | SD Empty Competition | Senior   | Basketball | 


#### Equips

Afegeix els següents equips. L'identificador només s'adjunta com a referència (IdRef), vosaltres tindreu els vostres.

| IdRef* | name      | country   | 
|--------|-----------|-----------|
| T1     | Australia | Australia |
| T2     | USA       | USA       |
| T3     | France    | France    |
| T4     | Germany   | Germany   |
| T5     | Japan     | Japan     |
| T6     | Canada    | Canada    |

#### Partits

A més a més, cal afegir els següents partits, afegint el límit en el nombre total d'entrades si s'ha implementat.

| local | visitor | competition | date             | price | entrades |
|-------|---------|-------------|------------------|-------|----------|
| T1    | T3      | C1          | 2024-07-29 11:00 | 325   | 100      |
| T2    | T4      | C1          | 2024-07-29 15:00 | 632   | 20       |
| T5    | T6      | C1          | 2024-07-30 10:00 | 50    | 3        |


### Exercici 3: Realització de les proves creuades

Durant aquesta sessió utilitzarem l'eina [Uptitude](https://uptitude.netlify.app/). Podeu entrar a la plataforma utilitzant el vostre **correu i el NIUB** com a contrasenya.
Com a lliurament en aquesta plataforma caldrà que pugis un **fitxer TXT** amb la següent informació:

```text
Grup
URL documentació (la de l'exercici 1 d'aquesta guia)
Nom i cognoms dels membres del grup
```
**NOTA:** En cas que tinguis problemes amb el desplegament a Render.com, afegeix en aquest fitxer la IP del teu ordinador,
i arranca la teva pràctica en el teu ordinador.

Si trobes errors durant la prova de les pràctiques dels teus companys, cal que ho indiquis a la teva memòria, i que els
hi comuniquis a ells.

Si els teus companys et comuniquen errors de la teva pràctica, cal que generis un **issue** per cada error detectat,
indicant en el títol de l'issue quin grup te l'ha reportat.

Caldrà aplicar el següent **protocol de proves**:

1. Navegar pel catàleg, revisant la informació que es mostra.
2. Verificar que no es pot afegir productes a la cistella sense estar autenticat.
3. Autenticar-se amb l'usuari test_rich_user@sd.ub.edu i comprovar que es pot comprar entrades.
Comproveu que el nombre d'entrades s'actualitza correctament.
4. Verificar que no es pot comprar més entrades de les disponibles d'un dels partits.
5. Tancar la sessió i comprovar que ja no es pot comprar.
6. Autenticar-se amb l'usuari test_poor_user@sd.ub.edu i comprovar que no se li permet comprar entrades dels dos primers
partits per falta de saldo.

A parti d'aquí podeu fer les proves que considereu oportunes. Podeu per exemple:

* Proveu d'accedir simultàniament amb dos comptes diferents i fer compres.
* Proveu d'accedir amb el mateix compte des de dos ordinadors o navegadors diferents.
* Podeu revisar la documentació de l'API.

Explica les proves que has realitzat i quins resultats has obtingut.

## Tasques fora del laboratori

### Tasca 1: Introducció de dades

Crea un **fitxer amb les crides CURL** necessàries per afegir les dades de prova de l'exercici 1, i lliura'l junt amb la pràctica.
Comenta els canvis que calgui fer (per exemple els IDs de crides anteriors)

### Tasca 2: Anàlitzar i solucionar els problemes trobats

Analitza els **issues** que has anat afegint durant la fase de proves, i soluciona aquells que consideris importants.
Explica a la documentació quins has solucionat i quins no, raonant la resposta.
