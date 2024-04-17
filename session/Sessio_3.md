Sessió 3
=========

Diseeny Web
-----

En aquesta segona part de la pràctica, desenvoluparem el frontend de la nostra aplicació.
Al final de la sessió 5 hauríem de veure una pàgina web amb els mateixos elements que el següent exemple:

**Vista LOG IN**

![image](figures/sessio-3_final-signin.png)

**Vista Matches sense usuari Loggat**

![image](figures/sessio-3_final-matches.png)

**Vista Matches amb l'usuari loggat**

![image](figures/sessio-3_final-matches-user.png)

**Vista de la Cistella de la Compra**

![image](figures/sessio-3_final-shopping-cart.png)

Introducció a Vue
--------

Vue és un framework progressiu per construir interfícies d'usuari, és a dir,
al costat del client. Tot i que Vue és més senzill que React o Angular, és
extremadament potent i es pot utilitzar per crear aplicacions avançades i
ofereix la creació de projectes de manera estructurada. Consisteix en un conjunt de
biblioteques opcionals i biblioteques de tercers, i compta amb una
comunitat en creixement. A més, Vue s'està convertint en molt popular i té un futur brillant.

Instal·lació de Vue
-----
Els següents passos s'han desenvolupat a la versió  5.0.4 de vue/cli, node.js versió 14.16.1 i npm versió v6.14.12.
Després d’instal·lar Vue, comprovem la versió del node:

	node --version

Si encara no teniu el node instal·lat o teniu una versió antiga, aneu a
<https://nodejs.org/en/download/> i instal·leu el paquet. Per verificar la instal·lació, feu servir les línies d’ordres següents:

	node --version
    npm --version
    
En cas que estigueu a Linux, haureu d'instal·lar Node Version Manager (NVM).
Us permetrà triar una versió de node específica. Per instal·lar-lo, podeu seguir aquesta guia:
<https://phoenixnap.com/kb/install-latest-node-js-and-nmp-on-ubuntu>. 

Hi ha alguns mètodes d'instal·lació per instal·lar Vue, en aquest cas l'instal·larem a través de Vue CLI.

 	 npm install -g @vue/cli
     npm install -g @vue/cli-init
    
Per obtenir més assistència, consulteu la següent guia d'instal·lació:
<https://cli.vuejs.org/guide/installation.html>. 
Després de la instal·lació, comproveu amb la línia d'ordres:

    vue --version

Crear i configurar un entorn de projecte
----
Primer de tot, aneu a la carpeta desitjada on voleu guardar el projecte. Després d'això, executeu a la línia d'ordres

    vue init webpack <name-project>

per exemple: 
	
	vue init webpack frontend
Seleccioneu la configuració tal com es mostra a
imatge següent:

![image](figures/sessio-3_vue-init.png)

Podeu interactuar amb les fletxes, les tecles d'entrada i l'espai.
Un cop creat el projecte, parem atenció als fitxers principals. La carpeta `/ src` té la següent estructura:

![image](figures/sessio-3_src.png)

- **App.vue**: s'encarrega de representar els components

- **assets**: on deseu tots els recursos, com ara imatges

- **components**: tots els components dels projectes. Tots els components tenen la seva pròpia plantilla html i codi JavaScript

- **main.js**: inicialitza i configura l'aplicació Vue

- **index.js**: enruta els components del vostre projecte

Hello world in Vue
-----------
Per executar la nostra primera aplicació Vue:

    cd <name-project>
    npm run dev
   
En cas que feu servir el WebStorm de JetBrains, obriu el projecte anomenat frontend i creu una configuració  amb el boto de Run. Feu que sigui del tipus npm i l'script dev.

Després d'executar aquestes línies d'ordres, aneu a <http://localhost:8080/> al navegador.

![image](figures/sessio-3_vue-cli.png)

![image](figures/sessio-3_vue.png)

Crear un component
------

Hem vist com executar un projecte. Ara veurem com crear el nostre propi component. Canvieu la plantilla html del fitxer `App.vue` com:

```html
<template>
    <router-view/>
</template>

```

A més, creeu un component nou anomenat `Matches.vue` a la carpeta de components. En aquest fitxer, copieu i enganxeu el codi següent:

```html
<template>
  <div id="app">
    <h1> {{ message }} </h1>
  </div>
</template>

<script>

export default {
  data () {
    return {
      message: 'My first component'
    }
  }
}

</script>

```

Com podeu veure aquí, tenim dos blocs anomenats "template" i "script". Com hem esmentat anteriorment, el primer bloc correspon a la visualització en html i el segon pertany al codi JavaScript. A "return" podem declarar les variables que utilitzarem al codi. També podeu interactuar incloent-hi les referències de codi a la plantilla html.
Paral·lelament, aneu a `index.js` per encaminar el nou component:

```html
import Vue from 'vue'
import Router from 'vue-router'
import Matches from '@/components/Matches'

Vue.use(Router)

export default new Router({
  mode: 'history',
  base: process.env.BASE_URL,
  routes: [
    {
      path: '/',
      name: 'Matches',
      component: Matches
    }
  ]
})

```
Comprovem el nostre nou component <http://localhost:8080/>

![image](figures/sessio-3_first-component.png)

Bootstrap
---------

Bootstrap (<https://www.w3schools.com/whatis/whatis_bootstrap.asp>) és el framework CSS més popular per al desenvolupament de llocs web responsius i per a mòbils. Conté plantilles de disseny basades en CSS i JavaScript per a tipografia, formularis, botons, navegació i altres components de la interfície.


Per instal·lar-lo al nostre projecte, executeu la comanda següent en la línia d'ordres:

	npm install --save bootstrap-vue
	
A més, descarregueu els fitxers compilats [aquí](https://getbootstrap.com/docs/4.0/getting-started/download/). Extreu els fitxers a una carpeta nova anomenada bootstrap dins del vostre projecte:

![image](figures/sessio-3_bootstrap.png)

Després de la instal·lació, configureu el fitxer `main.js` important el Bootstrap:


```html
import BootstrapVue from 'bootstrap-vue'
import 'bootstrap/dist/css/bootstrap.css'
import Vue from 'vue'
import App from './App.vue'
import router from './router'

Vue.use(BootstrapVue)
Vue.config.productionTip = false

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  components: { App },
  template: '<App/>'
})

```

Ara podem consumir les plantilles Bootstrap predefinides.

Mètodes
-----

A part de les variables, també podem definir mètodes i utilitzar-los en un bloc d'html. Definim una variable anomenada `tickets_bought` i un mètode
per incrementar aquest valor mitjançant un botó de Bootstrap a `Matches.vue`.

```html
<template>
  <div id="app">
    <h1> {{ message }} </h1>
    <button class="btn btn-success btn-lg" @click="buyTicket"> Buy ticket </button>
    <h4> Total tickets bought: {{ tickets_bought }} </h4>
  </div>
</template>

<script>

export default {
  data () {
    return {
      message: 'My first component',
      tickets_bought: 0
    }
  },
  methods: {
    buyTicket () {
      this.tickets_bought += 1
    }
  }
}

</script>
```

Com podeu veure, també podeu cridar a mètodes en un bloc html mitjançant `@click = "buyTickets"`, com en aquest cas.

### Exercici 1

Declareu un mètode per restar el total d’entrades comprades de les entrades totals. (i el botó d’interacció que s’anomena `Return Ticket`). A més, afegiu una variable `money_available` i `price_match` i resteu el preu de cada espectacle a la variable `money_available`. Utilitzeu el preu, els diners disponibles i les entrades diponibles que vulgueu. Mostra-ho al lloc web.

### Exercici 2

Desactiveu el botó de compra si els diners disponibles no són suficients per comprar un ticket.
D’altra banda, desactiveu el botó de ticket si el total de bitllets és 0.
Per fer-ho, podeu utilitzar la propietat `:disabled="variable1 < variable2"` en els botons.

Conectant Vue i FASTApi
--------------

Ara hem de connectar el Framework de Frontend de VUE (node.js) amb el Framework de Backend de FASTApi (python). Però primer haurem de permetre que es puguin fer "requests" des del client (javascript del navegador) al servidor web. Per això haurem de permetre-ho en el nostre backend de FAST-API.

### Activant CORS (Cross-Origin Resource Sharing)

CORS és el mecanisme per gestionar les sol·licituds d’origen creuat. Una sol·licitud de recurs de fora de l’origen es coneix com a sol·licitud d’origen creuat. En aquesta pràctica, crearem una aplicació de frontend que sol·licitarà informació a FASTApi. 

Importeu el CORSMiddleWare i configureu l'aplicació per tal d'acceptar sol·licituds de tots els orígens:

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/")
async def main():
    return {"message": "Hello World"}
...
```

Així doncs, així concediu accés a l’aplicació FastAPI des de l’exterior.

### Configurant els directoris static de FASTAPI.


Primer de tot, aneu a la vostra aplicació Vue i compileu el vostre codi amb la línia següent:

	npm run build
	
Amb WebStorm podeu crear una nova configuració d'execució npm run i amb la comanda build.

Es crearà una nova carpeta anomenada **dist** on podrem trobar el codi frontend del nostre projecte (HTML, CSS, JS). Ara, la nostra aplicació Vue té l'estructura següent:

![image](figures/sessio-3_dist.png)


Ara és hora d’indicar a FASTapi els camins a consumir des de `main.py`:

```python
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles

app = FastAPI()

app.mount("/static", StaticFiles(directory="frontend/dist/static"), name="static")
```

I declareu una ruta per renderitzar la plantilla:

```python
from fastapi import FastAPI, Request
from fastapi.templating import Jinja2Templates
from fastapi.staticfiles import StaticFiles


templates = Jinja2Templates(directory="frontend/dist") 
@app.get("/")
async def serve_index(request: Request):
    return templates.TemplateResponse("index.html", {"request": request})
```


Si executeu l'aplicació FASTapi i obriu <http://127.0.0.1:8000/>, hauríeu de veure l'última configuració de Vue però allotjada a l'aplicació FastAPI:

![image](figures/sessio-3_buy-ticket.png)
