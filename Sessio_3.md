# Sessió 3
En aquesta sessió activarem l'autenticació dels usuaris i l'autorització d'aquests per accedir a l'API. 

## Objectius
* Protecció de dades a l'aplicació
* Entendre els conceptes d'autenticació i autorització
* Conèixer les tecnologies bàsiques per a l'autenticació de peticions i gestió d'usuaris
* Desenvolupar la seguretat de l'aplicació
  * Autenticació dels usuaris
  * Protecció dels endpoints de l'API
* Finalitzar la implementació de l'aplicació.
  

## Guies
Per realitzar els exercicis d'aquesta sessió, teniu disponibles les guies següents:
* [Introducció Seguretat de les Dades](guies/intro_seguretat_dades.md)
* [Autenticació i Autorització](guies/autenticacio.md)

## Exercicis al laboratori

Durant aquesta sessió utilitzarem l'eina **curl** per fer proves de crides a l'API des de 
la línia de comandes.

Podeu instal·lar aquesta eina fàcilment en la majoria de les distribucions
linux:

```bash
sudo apt install curl
```

També hi ha versió per a Windows. Podeu instal·lar-lo des d'[aquest enllaç](https://curl.se/windows/). 

Una aplicació que també us pot facilitar aquest tipus de proves és [Postman](https://www.postman.com/).

### Exercici 1: Creació usuari

En el codi de la pràctica per a la gestió d'usuaris ```api/routes/users.py``` teniu definits els endpoints per a la gestió 
d'usuaris, i accessibles a través de l'url ```/api/v1/users/```. Reviseu els mètodes disponibles a través de l'ajuda
autogenerada ```/docs```.

Primer de tot, anem a llistar tots els usuaris que hi ha a la base de dades:

```bash
curl -X "GET" "http://127.0.0.1:8000/api/v1/users/" -H "accept: application/json"
``` 

Fixeu-vos que quan fem una crida amb **curl**, li donem un URL on connectar i li podem indicar un verb amb el modificador **-X** i 
una capçalera (o varies) amb el modificador **-H**. En aquest cas el modificador **-X** no caldria, ja que el valor per defecte és ```GET```.
Pel que fa a les capçaleres, s'indica una propietat (accept) i el seu valor (application/json) separats per dos punts. Teniu informació sobre les
capçaleres disponibles [en aquest enllaç](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept). En aquest 
cas, li estem indicant al servidor (l'API), que "acceptem" com a retorn informació en format "JSON" i, per tant, li demanem
que ens serialitzi els resultats en aquest format.

Ara crearem un nou usuari, i li assignarem un correu electrònic i una contrasenya (podeu canviar els valors):

```bash
curl -X "POST" "http://127.0.0.1:8000/api/v1/users/" -H "accept: application/json" -H "Content-Type: application/json" -d "{\"full_name\": \"John Smith\", \"password\": \"my_password\", \"email\": \"admin@p2.sd.ub.edu\"}"
```

Fixeu-vos que en aquest cas el verb que utilitzem és ```POST```, i afegim nous modificadors. Primerament, li estem donant
una nova capçalera per a la propietat "Content-Type", indicant al servidor que el contingut de la petició que li enviem
està codificat en format JSON. Finalment, li afegim el contingut amb el modificador **-d**. 

Si utilitzes **PyCharm** pots obrir el fitxer amb la base de dades SQLite des del panell **Database**. En el cas 
de **Visual Studio Code** podeu utilitzar el complement [SQLite Viewer](https://marketplace.visualstudio.com/items?itemName=qwtel.sqlite-viewer).
Comprova les dades del nou usuari a la base de dades, i verifica si les dades que li has passat al crear-lo
corresponen a les que s'han guardat. Hi ha algun canvi? Què ha passat amb el camp ```password```?

Revisa els mètodes **create_user** i **update_user** del mòdul ```app.api.crud.user```, fixa't que s'utilitza el mètode
**get_password_hash** definit en el mòdul ```app.core.security```. Què s'està fent? Per què? Revisa la [guia sobre seguretat
de les dades](./guies/intro_seguretat_dades.md) per a més detalls. 

### Exercici 2: Obtenció token autenticació

En el mòdul ```app.api.routes.login``` teniu definits els endpoints relacionats amb l'autenticació. Mireu la definició
del mètode **login_access_token**, accessible a l'URL ```/api/v1/login/access-token```. Executa la següent comanda amb
les credencials de l'usuari creat en l'exercici anterior:

```bash
curl -X "POST" "http://127.0.0.1:8000/api/v1/login/access-token" -H "accept: application/json" -H "Content-Type: application/x-www-form-urlencoded" -d "username=admin@p2.sd.ub.edu&password=my_password"
```

Fixa't que en aquest cas li estem passant la informació en format de formulari codificat (application/x-www-form-urlencoded) 
i no pas un JSON. Si tot ha anat bé, us hauria de retornar un token d'accés. Prova també de fer una crida amb una contrasenya
o un usuari incorrectes, a veure què passa.

Ara mireu el mètode **test_token**, accessible a l'URL ```/api/v1/login/test-token```. Aquest mètode permet provar els 
tokens. Primer, crideu a aquest mètode sense passar cap token, per tant, sense
estar autenticat:

```bash
curl -X "POST" "http://127.0.0.1:8000/api/v1/login/test-token" -H "accept: application/json"
```

Per fer una crida autenticada amb el token anterior, cal afegir una nova capçalera per a la propietat "Authorization". Li 
indicarem que tenim un token de tipus "Bearer" i el seu valor:

```bash
curl -X "POST" "http://127.0.0.1:8000/api/v1/login/test-token" -H "accept: application/json" -H "Authorization: Bearer **<token>**"
```
on **<token>** és el valor obtingut en la crida anterior. Comprova el contingut del token a la pàgina [jwt.io](https://jwt.io).


### Exercici 3: Protecció endpoints

Obre el mòdul ```app.api.routes.users``` i fixa't en el paràmetre **dependencies** que teniu
comentat tant en el mètode **read_users** com en el mètode **create_user**. Aquest paràmetre permet limitar
l'accés a determinats mètodes. En aquest cas, el mètode ```get_current_active_superuser``` exigeix
que per accedir a aquest mètode l'usuari estigui actiu i sigui administrador. Això correspon a dir que els camps
**is_active** i **is_superuser** del model **User** estiguin actius.

Anem a provar el seu funcionament. Primer de tot, verifica que pots llistar tots els usuaris amb al comanda:

```bash
curl -X "GET" "http://127.0.0.1:8000/api/v1/users/" -H "accept: application/json"
``` 

Ara elimina el comentari en el mètode **read_users**. Reinicia el servidor i torna a executar la comanda:

```bash
curl -X "GET" "http://127.0.0.1:8000/api/v1/users/" -H "accept: application/json"
``` 

Què ha passat? Has pogut accedir-hi? Repeteix els passos de l'exercici 2 per obtenir un token d'accés. Ara fes
la crida amb aquest token:

```bash
curl -X "GET" "http://127.0.0.1:8000/api/v1/users/" -H "accept: application/json" -H "Authorization: Bearer **<token>**"
```
on **<token>** és el token d'accés. Modifica els camps **is_active** i **is_superuser** per
veure la resposta en les diferents variacions.

Repeteix els mateixos passos per al mètode **create_user**.


### Exercici 4: Implementació autenticació al frontend

Ara que ja tenim el backend protegit, caldrà modificar el frontend perquè ho tingui en compte. 
Comencem pel disseny del component. En primer lloc, creeu un nou component anomenat **Login** en el fitxer
```components/Login.vue```. 

```javascript
<template>
  <form>
    <div class="form-label-group">
      <label for="inputEmail">Username</label>
      <input type="text" id="inputUsername" class="form-control"
      placeholder="Username" required autofocus v-model="username">
    </div>
    <div class="form-label-group">
      <br>
      <label for="inputPassword">Password</label>
      <input type="password" id="inputPassword" class="form-control"
      placeholder="Password" required v-model="password">
    </div>
    <div>
      <b-button @click="login_user" variant="primary">Sign In</b-button>
      <b-button @click="register_user" variant="success">Create Account</b-button>
      <b-button @click="back_matches" variant="secondary">Back To Matches</b-button>
    </div>
  </form>
</template>

<script>
export default {
  name: 'Login',
  data () {
    return {
      username: null,
      password: null,
      token: null,
      is_authenticated: false
    }
  },
  methods: {
    login_user (event) {
    },
    register_user (event) {
    },
    back_matches (event) {
    }
  }
}
</script>
```
Registreu la ruta dins el fitxer ```router/index.js```:

```javascript
import Vue from 'vue'
import Router from 'vue-router'
import Shows from '@/components/Shows.vue'
import Login from '@/components/Login.vue'

Vue.use(Router)

export default new Router({
  mode: 'history',
  base: process.env.BASE_URL,
  routes: [
    {
      path: '/',
      name: 'Matches',
      component: Matches
    },
    {
      path: '/userlogin',
      name: 'Login',
      component: Login
    }
  ]
})
```

Un cop l'estructura estigui llesta, podem dissenyar la vista d'inici de sessió com per exemple:

![image](figures/sessio-5_login.png)

Com podeu veure, tenim dos textos d’entrada i tres botons.

Afegeix el següent codi a l'acció associada al botó "Sign In":

```javascript
login_user () {
  const parameters = 'username=' + this.username + '&password=' + this.password
  const config = {
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  const path = 'http://localhost:8000/api/v1/login/access-token'
  axios.post(path, parameters, config)
    .then((res) => {
      this.logged = true
      this.token = res.data.token.access_token
      this.$router.push({ path: '/', query: { username: this.username, logged: this.logged, token: this.token } })
    })
    .catch((error) => {
      // eslint-disable-next-line
      console.error(error)
      alert('Username or Password incorrect')
    })
}
```
Recorda a importar el component 'axios' amb:

```javascript
import axios from 'axios'
```

Observem que estem fent servir una eina per canviar la ruta actual del component un altre:

```javascript
this.$router.push({ path: '/', query: { username: this.username, logged: this.logged, token: this.token } })
```

Els paràmetres que passem ens permeten donar a la vista principal la informació sobre l’usuari. "username" conté el nom d'usuari actual, "logged" és un booleà que mostra si l'usuari ha iniciat la sessió correctament i "token" és la string del token per a poder enviar requests autoritzades.

Per consumir la informació de la consulta des de la vista `Matches`, hem d'inicialitzar els atributs a data i podem utilitzar les línies següents a `created()`:

```javascript
created () {
  this.logged = this.$route.query.logged === 'true'
  this.username = this.$route.query.username
  this.token = this.$route.query.token
  if (this.logged === undefined) {
    this.logged = false
  }
```
        
### Exercici 5: Implementació autenticació al frontend

A partir de l'exemple de l'exerercici 4, crea un nou servei **AuthService** a l'estil del component **TeamService** que 
vàrem crear a la sessió 2. Mou l'obtenció del token a aquest servei.

## Tasques fora del laboratori

### Tasca 1: Protegir gestió dades de l'aplicació

Afegeix protecció a l'aplicació per garantir que:

- Només els administradors poden crear competicions i equips
- Només els organitzadors d'una competició i els administradors poden crear partits
- Qualsevol usuari registrat ha de poder comprar entrades

### Tasca 2: Protegir dades

Modifica la teva aplicació per tal que els diners disponibles a un compte es guardin de forma encriptada a la base de dades.

### Tasca 3: [Opcional] Transaccions a la base de dades
Modifica el backend per tal d'introduir transaccions en la compra d'entrades i salari. Podeu trobar la informació en [aquesta guia](https://docs.sqlalchemy.org/en/20/orm/session_transaction.html). 

### Tasca 4: [Opcional] Ús d'emmagatzematge local
Modifica el servei d'autenticació per tal que el token es guardi en l'emmagatzematge local, evitant haver-lo d'anar passant com a paràmetre.
Teniu la informació en aquest [enllaç](https://es.vuejs.org/v2/cookbook/client-side-storage).

