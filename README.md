---
author: Xevi Baró, Blai Ras Jimenez, Eloi Puertas
date: Abril 2024
title: Pràctica 2 - FastAPI and Vue application (Software distribuït)
---

# Enunciat_P2

Introducció
============

En aquesta pràctica farem una pàgina web capaç de mostrar 
informació sobre esdeveniments esportius i reservar-ne les entrades.
Però primer de tot desenvoluparem una REST-API capaç de proporcionar-nos la informació necessària. També desenvoluparan les funcions relacionades amb la gestió d'aquestes dades com
afegir, eliminar o modificar les dades emmagatzemades.

Amb aquest propòsit utilitzarem el Framework anomenat FastAPI.

Important
---------

Aquest exercici guiat suposa que ja teniu alguna versió de Python de versió mínima 3.10
instal·lada i esteu familiaritzats amb la instal·lació de paquets mitjançant pip.

Tingueu en compte que el pip command mostrat en aquest tutorial correspondrà a pip3 
segons quantes versions diferents de python tingueu
instal·lat al vostre ordinador.

Es recomana altament l'ús de la IDE [PyCharm Professional](https://www.jetbrains.com/pycharm/) per al desenvolupament d'aquesta pràctica. Si no el teniu instal·lat, existeix una versiñó per a estudiants que podeu obtenir si us enregistreu amb el correu de la UB. 

Enviament dels Exercicis
------------------------
Es treballarà de la mateixa manera que a la Pràctica 1. 

- Els exercicis i deures s'enviaran com a Pull Requests al repositori del Github Classroom. 
- Si n'hi ha, les respostes a les preguntes 
s'enviaran com a commentaris en el Pull Requests. 
- S'ha de fer Peer Review dels Pull Requests per part de l'altre membre del grup.
- Durant la realització de la pràctica haureu d'anar documentant el vostre progrés en la memòria que trobareu en el directori `docs` del template de la pràctica.  

Avaluació pràctica 2.
---------------------------
- 30% Fer els pull requests setmanalment.
- 70% Lliurament final: 20% Sessió de Test, 50% codi final, 30% Memòria progrès.


Objectius Pràctica
-------------------

#### Back-End: 
* Crear aplicació CRUD amb autentificació per a gestionar (CRUD: Create, Read, Update i Delete) de la part de model d'aplicació 

#### Front-End: 
L'aspecte de l'aplicació ha de ser semblant en aquest:

- **Vista LOG IN**

![image](figures/sessio-3_final-signin.png)

- **Vista Matches sense usuari Loggat**

![image](figures/sessio-3_final-matches.png)

- **Vista Matches amb l'usuari loggat**

![image](figures/sessio-3_final-matches-user.png)

- **Vista de la Cistella de la Compra**

![image](figures/sessio-3_final-shopping-cart.png)


Calendari
==========


|  Sessió  |  Data | Tema  |  Guia | Slides |
|---|---|---|---|---|
|  1 | 17/04/24 |  Backend i Deplegament| Creació Models i Endpoints |
|  2 | 24/04/24 | Frontend  | Creació del Frontend  | 
|  3 | 08/05/24  | Seguretat |  Autenticació i protecció d'endpoints| 
|  4 | 22/05/24  |  Depuració, Testing i Performance per a múltiples usuaris | Testing i Depuració   | 
| 5  | 29/05/24  | Sessió de Testing | Guia Sessió de testing |



Sessió 1
=========
17 i 18 d'Abril de 2024

### Objectius
* Entendre FastAPI
* Llegir i Entendre el Template d'aplicació web que us donem
* Llegir i Entendre la Guia que us donem.
* Solucionar els exercicis de la Guia adaptats al template que us doenm 

### Excercicis

1. Adapteu el mètode `@app.get("/team/{team_name}")`  per a que funcioni amb el model anterior.

2. Escriviu el mètode DELETE que elimini un equip de la llista d'equips. Torneu un missatge indicant si aquest usuari s'ha suprimit o no i el codi corresponent.

3. Escriviu el mètode PUT que, si no existeix aquest equip, crea un equip nou i l'afegeix a la llista d'equips. En cas contrari, modifica els valors de l'equip amb aquest identificador amb la informació del cos de la sol·licitud del PUT (aquesta informació pot ser completa o parcial!). Retorna l'equip creat o modificat.

4.  Escriviu els mètodes GET, POST, DELETE i PUT per a un model anomenat Competition amb la URL:

        '/competitions/
    A sota teniu exemples de competicions, definiu el model Pydantic segons els exemples.

5.  Escriviu els mètodes GET, POST, DELETE i PUT per a un model anomenat Match amb la URL:

        '/matches/'

    A sota teniu exemples de competicions, definiu el model Pydantic segons els exemples.

6. Feu els mètodes GET per a competicions i matches donat l'ID. 
    
7. Proveu i comproveu tots els mètodes desenvolupats amb `requests` o una altra eina de prova d'APIs.

8. Feu un test per a cada mètode desenvolupat i proveu que tot funciona correctament.

``` python
competitions = [
    {'id': 0,
    'name': "Women's European Championship",
    'category': 'Senior',
    'sport': 'Volleyball',
    'teams': []},
    {'id': 1,
    'name': "1st Division League",
    'category': 'Junior',
    'sport': 'Football',
    'teams': []},
    {'id': 2,
    'name': "Women's Copa Catalunya",
    'category': 'Senior',
    'sport': 'Basketball',
    'teams': []},
    {'id': 3,
    'name': "1st Division League",
    'category': 'Junior',
    'sport': 'Futsal',
    'teams': []}
]

teams = [
    {'name': "CV Vall D'Hebron",
     'country': 'Spain'},
    {'name': 'CE Sabadell',
     'country': 'Spain'},
    {'name': 'Club Juventut Les Corts',
     'country': 'Spain'},
    {'name': 'Volei Rubi',
     'country': 'Spain'}
]

matches = [
    {'id': 0,
     'local': "CV Vall D'Hebron",
     'visitor': 'Volei Rubi',
     'date': '2022-07-03',
     'price': 15.20}
]
``` 


