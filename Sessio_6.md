---
layout: page
title: Azure Deployment
---
---

# Del desenvolupament a la producció.

Ara toca desplegar l'aplicació a un servidor de producció. En aquest cas utilitzarem Azure, ja que com a estudiants
de la Universitat de Barcelona hi teniu un crèdit de 100E anuals. Vigileu de no gastar-los tots!

## Creant el Docker Compose file.
Docker Compose ens permet crear un conjunt de serveis que s'executen en un entorn aïllat. En el nostre cas, crearem un servei per a la base de dades, un altre per a l'aplicació web fastApi i un altre per l'aplicació de forntend VUE.
Necessitarem un Docker file per als serveis de backend i frontend i un Docker Compose file per a tots els serveis.
En l'arrel del vostre repositori de Github creeu el directori backend i el directori frontend
i guardeu-hi els codis fonts que teniu fets fins ara (*.py) en un directori src
Hauríeu de tenir una cosa que sigui similar en això:
```
├── docker-compose.yml
└── services
    ├── backend
    │   ├── Dockerfile
    │   ├── requirements.txt
    │   └── src
    │       └── main.py
    └── frontend
        ├── .gitignore
        ├── Dockerfile
        ├── README.md
        ├── babel.config.js
        ├── jsconfig.json
        ├── package-lock.json
        ├── package.json
        ├── public
        │   ├── favicon.ico
        │   └── index.html
        ├── src
        │   ├── App.vue
        │   ├── assets
        │   │   └── logo.png
        │   ├── components
        │   │   └── HelloWorld.vue
        │   ├── main.js
        │   ├── router
        │   │   └── index.js
        │   └── views
        │       ├── AboutView.vue
        │       └── HomeView.vue
        └── vue.config.js
```

### Docker-compose.yml

Creeu el següent fitxer docker_compose.yml a l'arrel del vostre repositori

```yml
version: '3.8'

services:

  backend:
    build: ./services/backend
    ports:
      - 8000:8000
    environment:
      - DATABASE_URL=postgresql://postgres:postgres@db:5432/appdb
    volumes:
      - ./services/backend:/app
    command: uvicorn src.main:app --reload --host 0.0.0.0 --port 8000 --env-file path/fitxer/.env
    depends_on:
      - db
    networks:
        - app-network
 
  frontend:
    build: ./services/frontend
    volumes:
      - './services/frontend:/app'
      - '/app/node_modules'
    ports:
      - 8080:8080
    networks:
        - app-network
      
  
  db:
    image: postgres:15.1
    expose:
      - 5432
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=appdb
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    networks:
        - app-network

volumes:
  postgres_data:

networks:
  app-network:
```

### Dockerfile per al servei de backend
En el subdirectori del backend poseu el següent fitxer Dockerfile
```dockerfile
FROM python:3.11-buster
RUN mkdir app
WORKDIR /app
ENV PATH="${PATH}:/root/.local/bin"
ENV PYTHONPATH=.
COPY requirements.txt .
RUN pip install --upgrade pip
RUN pip install -r requirements.txt

COPY src/ .
```

Comproveu que teniu l'última versió de requirements.txt amb totes les llibreries que feu servir en vostre codi

### Dockerfile per al servei de frontend
```dockerfile
FROM node:lts-alpine

WORKDIR /app

ENV PATH /app/node_modules/.bin:$PATH

RUN npm install @vue/cli@5.0.8 -g
RUN npm install -g postcss-loader postcss postcss-import postcss-url
RUN npm install -g serve

COPY package.json .
COPY package-lock.json .
RUN npm install
RUN npm run build   

CMD ["npm", "run", "serve"]
```

### Afegir script serve a package.json

Afegiu el següent script a `package.json` del frontend

```json
"scripts": {
     "serve": "serve -s dist -p 8080"
}
```
Heu d'instal·lar el serve amb `npm install -g serve`



### Nova configuració de CORS:
Canviem el CORS a main.py per tal que només accepti peticions del frontend

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:8080"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```
### Variables d'entorn i configuració de l'entorn de producció

En el utils.py afegim una variable `production: bool ` que ens permetrà canviar el comportament de l'aplicació en funció de si estem en producció o no.
En el fitxer .env afegim la variable d'entorn `PRODUCTION=True`
En el fitxer `database.py` feu que la variable `DATABASE_URL` s'obtingui de la variable d'entorn `DATABASE_URL` en cas que estiguem en producció.
La variable està en l'environment definit a docker_compose.yml: `DATABASE_URL=postgresql://postgres:postgres@db:5432/appdb`.
Per obtenir una variable de l'entorn de producció podeu fer servir la funció `os.getenv("DATABASE_URL")`
Pot ser que hàgiu de canviar la definició del model de la base de dades perquè funcioni amb la base de dades de producció.
Alguns tipus poden no funcionar en postgres o pot ser que hàgiu d'especificar algun paràmetre més. En tot cas
mireu bé els errors de postgres que us pugui donar quan ho proveu primer en local.
Afegiu al requirements.txt la llibreria `psycopg2-binary` que és la que ens permetrà connectar-nos a la base de dades de postgres.


També segons si estem en producció o no, canvieu l'endpoint root de l'aplicació. En el cas de producció no ha de retornar res.
En aquest cas, tampoc cal que munteu el subdirectori static ni el directori templates.
El fitxer de BD de sqlite el podeu treure del projecte, ja que en producció no el farem servir. 

### Execució en local de que tot funcioni
Executeu el docker-compose en local per comprovar que tot funciona correctament.
`docker-compose up -d --build`

Proveu el link a 

 http://localhost:8080/.


Aprofiteu per crear l'usuari admin i guardar-lo en la base de dades de postgres, en cas contrari l'haureu de crear
des de l'entorn de producció.
Ara que ja heu provat que tot és correcte podeu actualitzar el vostre repositori de github amb els canvis.

### Creant la imatge docker
Un cop tot funcioni en local, pugeu el vostre projecte docker a dockerhub. Si no teniu encara un usuari de docker hub, creeu-vos-ne un.
Heu de pujar mínim dues imatges, una per al backend i una per al frontend.
Ho podeu fer des de la línia d'ordres amb el client de docker o des del web de dockerhub.


``` bash
docker login
docker-compose build
docker tag <nom_imatge> <usuari_dockerhub>/<nom_imatge> # Nom imatge serà 2023 seguit del nom del vostre grup i backend o frontend
docker push <usuari_dockerhub>/<nom_imatge>
```
Un cop tingueu la vostra imatge al dockerhub, ja podeu desplegar el vostre projecte a Azure.



## Desplegant a Azure
Ara ja estem preparats per portar el nostre projecte a Azure:

* Entreu a Azure.com i registreu-vos amb el vostre correu de la UB.
* A continuació crearem un nou recurs donant-li al símbol + a la part superior esquerra de la pantalla.

Farem servir 3 serveis diferents d'azure per desplegar els nostres serveis:

1. Un servei Static Web App per al frontend:

* Poseu la URL del backend en el fitxer .env del frontend amb el nom de variable d'entorn `VUE_APP_BACKEND_URL`
* En les crides a Axios del frontend agafeu la URL del backend amb la funció de vue `process.env.VUE_APP_BACKEND_URL`
* Compileu tot el vostre projecte frontend i pugeu-lo a una branca anomenada deploy del vostre repositori de github.
Crearem un nou recurs a Azure, però aquest cop triarem un servei Static Web App per al frontend:

* Triarem el mateix Resource Group que l'anterior servei.
* Poseu el nom de frontend
* Trieu el pla gratuït
* Trieu la regió West Europe
* Escollirem el repositori de github on tenim el nostre projecte en l'Organització SoftwareDistribuitUB-2023
* Trieu la branca deploy
* A Build Presets escolliu Vue.js
* A App location poseu /frontend

Ja podeu crear el servei.
Aneu al recurs i veureu la URL del Frontend.
Apunteu-la perquè la necessitareu més endavant.


2. Una base de dades de postgresql:

Finalment crearem una base de dades de postgresql:
* Afegiu un nou recurs i trieu Database for PostgreSQL.
* Trieu el mateix Resource Group que l'anterior servei.
* Trieu les opcions més bàsiques, reduint el preu al mínim.
* Apunteu-vos el nom de la base de dades, la URL i el nom d'usuari i contrasenya que heu triat, ho necessitareu més endavant.


3. Un servei Web App per al backend:
Abans de pujar el backend a Azure, canvieu el fitxer de configuració de la base de dades perquè apunti a la base de dades de producció, canvieu les variables d'entorn perquè apuntin ala base de dades i canvieu el fitxer de configuració de CORS perquè només accepti peticions del frontend que heu creat abans.
Genereu el nou Dockerfile per al backend i pugeu-lo a dockerhub.

* Triarem una Web App per a crear. 
* Omplirem els camps bàsics:
  * Subscription: Azure for Students
  * Resource Group: Triarem el que hem creat abans
  * Name: Poseu-li 2023 i el nom del vostre grup
  * Publish: Docker Container
  * Operating System: Linux
  * Runtime stack: Python 3.9 (el que hàgiu fet servir en local)
  * Operating System: Linux
  * region: West Europe
  * pricing tier: F1 Free (Important canviar-ho o pagareu diners!!!)
* Omplirem els camps de Docker:
  * Options: Single Container
  * Image Source: Docker Hub
  * Access Type: Public
  * Image and tag: Poseu el nom de la imatge del Backend que heu pujat a dockerhub
  * Startup Command: Poseu el mateix que heu posat al fitxer docker-compose.yml: `uvicorn src.main:app --host 0.0.0.0 --port 80 --env-file path/fitxer/.env`
* Omplirem els camps de Networking:
  * Enable public access: On
* Omplirem els camps de Monitoring:
  * Enable Application Insights: No
* Anem a Review + create i creem la web app.
* Un cop creat anem al recurs i copiem la URL del backend.

Ja podeu provar que tot funcioni bé amb la URL del backend que podeu veure a Domains o prement el botó Browse de la part superior de la pantalla.

Finalment canvieu la URL del backend al fitxer de configuració del frontend i feu un commit i push al vostre repositori de github a la branca de deploy del frontend


Comproveu que el vostre codi funciona correctament, especialment que els diners i el nombre d'entrades s'actualitzen de forma correcta.
En les funcions que no siguin async no hi hauria d'haver problemes. En les funcions que siguin async, pot ser que hàgiu  de sincronitzar parts del vostre codi per evitar condicions de carrera.
En producció és important que la majoria de codi sigui async per tal de poder servir el major nombre de clients
i que no hi hagi esperes innecessàries. Aquí teniu una implementació de Lock per si us fes falta:

`lock.py` :

```python
import threading

class my_Lock(object):
   __singleton_lock = threading.Lock()
   __singleton_instance = None

   @classmethod
   def getInstance(cls):
      if not cls.__singleton_instance:
         with cls.__singleton_lock:
            if not cls.__singleton_instance:
               cls._singleton_instance = cls()
      return cls.__singleton_instance

   def __init__(self):
      """ Virtually private constructor. """
      if my_Lock.__singleton_instance != None:
         raise Exception("This class is a singleton!")
      else:
         my_Lock.__singleton_instance = self
         self.lock = threading.Lock()

lock = my_Lock.getInstance()
```

per exemple a `

```python
from lock import lock
....
    @app.route('/api/v1.0/....', methods=['POST'])
     async def post_...():
        #codi codi codi Async
       
        with lock.lock: 
        	#codi Sync
            ....
         #codi codi codi Async
		 return ....


```
