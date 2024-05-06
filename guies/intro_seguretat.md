Introducció a la seguretat fastAPI
=========
En aquesta guia es mostra un exemple de com utilitzar autenticació amb fastAPI. Tingueu en compte que aquesta guia **no segueix**
l'estructura de la pràctica. Utilitzeu el codi com a exemples de com es poden realitzar les diferents tasques.

### Crear un usuari
Ara redefinirem l'endpoint per a crear un usuari nou. Per això, modifiqueu el post d'Account de tal forma que segueixi la següent lògica.
Feu servir els esquemes que ja teniu o creeu-los de nou. Però tingueu en compte que el camp password no es pot retornar en la resposta. Per això, heu de crear un  esquema que no tingui el camp password. Aquest esquema el farem servir per retornar la resposta del post d'Account.

```python

from .utils import get_hashed_password

@app.post('/account', summary="Create new user", response_model=schemas.Account)
def create_user(data: schemas.AccountCreate):
    # querying database to check if username already exist
    # if user exist, raise an exception
    # if not create user
    user = {
        'username': data.username,
        'password': get_hashed_password(data.password)
       # set the rest of the fields
    }
    # saving user to database
    return user

```
En l'Schema AccountCreate definiu els camps username i password que farem servir en crear una compte. En el cas de password podeu definir com voleu que sigui aquest, per exemple podeu fer un string de mínim 8 lletres i màxim 24. 
A més, si definiu una descripció per a cada camp es farà servir per a la documentació automàtica.


```python:

```python
from pydantic import BaseModel, Field
class AccountCreate(BaseModel):
    username: str = Field(..., description="username")
    password: str = Field(..., min_length=8, max_length=24 ,description="user password")
```

### Endpoints habituals

Un conjunt habitual de mètodes a l'api per a la gestió d'usuaris pot ser:

```python
@app.get('/account/{username}', response_model=schemas.Account) #obtenir informació del compte amb un nom d'usuari
@app.post('/account') #creeu un compte nou passant `username` i `password' Utilitzeu `hash_ password` quan creeu un compte (primer heu de crear un usuari nou i després afegir una contrasenya hash mitjançant el mètode `.hash_ password (password)`). 
@app.delete('/account/{username}') #suprimiu un compte relacionat amb un nom d'usuari (recordeu també suprimir totes les comandes relacionades).
@app.put('/account/{username}') #actualitzeu la informació del compte amb un nom d'usuari
```
i també:

```python
@app.get('/accounts') #obtenir informació sobre tots els comptes
```

Recordeu que heu de retornar missatges descriptius i un codi si no es poden publicar algunes peticions.


Validació de contrasenya
------------------------
FastAPi té una manera de validar els camps de les peticions per complir amb els estàndards d'OpenAPI . Per això, heu de definir un esquema que tingui els camps que voleu validar i afegir-hi les restriccions que voleu. 
Afegiu el següent endpoint a main.py i feu la lògica demanada.
TokenSchema 

```python
from fastapi.security import OAuth2PasswordRequestForm
from fastapi.responses import RedirectResponse
from .utils import verify_password, create_access_token, create_refresh_token, get_hashed_password

@app.post('/login', summary="Create access and refresh tokens for user", response_model=TokenSchema)
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
   username = form_data.username
   password = form_data.password
   # get user from database
   # if user does not exist, raise an exception
   # if user exist, verify password using verify_password function
   # if password is not correct, raise an exception
   # if password is correct, create access and refresh tokens and return them
   return {
        "access_token": create_access_token(user['username']),
        "refresh_token": create_refresh_token(user['username']),
    }
```
Aquest endpoint és una mica diferent dels altres endpoints. Usem  `OAuth2PasswordRequestForm` com a dependència.
Aquesta dependència ens permet extraure les dades de la petició amb autentificació bàsica HTTP. Fa servir la llibreria `python-multipart`
assegureu-vos de tenir-la instal·lada amb un pip install python-multipart.
En la documentació de l'API podreu veure els inputs necessaris per a username i password.

L'Schema de TokenSchema és el següent:

```python
from pydantic import BaseModel
class TokenSchema(BaseModel):
    access_token: str
    refresh_token: str
```
Afegiu-ho al fitxer `schemas.py`.
En cas de resposta correcte veureu els tokens generats. 

Protegir els endpoints
----------------------
Ara que ja tenim suport per a l'autentificació, podem posar endpoints protegits per tal que només usuaris amb privilegis com l'administrador o usuaris registrats
puguin accedir-hi. Per això, heu de modificar els endpoints de la següent manera:
Per fer-ho crearem dependències que ens permetran verificar si l'usuari té accés a l'endpoint.
Aquestes dependències es poden afegir com a paràmetres a les funcions dels endpoints, s'executen abans que la funció i poden retornar un valor que s'utilitzarà com a paràmetre per a la funció.
Crea un nou fitxer anomenat `dependencies.py` i afegiu-hi el següent codi:
```python
from typing import Union, Any
from datetime import datetime
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from . import utils
from jose import jwt
from pydantic import ValidationError
from .schemas import TokenPayload, SystemAccount


@lru_cache()
def get_settings():
    return utils.Settings()

reuseable_oauth = OAuth2PasswordBearer(
    tokenUrl="/login",
    scheme_name="JWT"
)

async def get_current_user(token: str = Depends(reuseable_oauth),settings: Annotated[utils.Settings, Depends(get_settings)]) -> SystemAccount:
    try:
        payload = jwt.decode(
            token, settings.jwt_secret_key, algorithms=[settings.algorithm]
        )
        token_data = TokenPayload(**payload)
        
        if datetime.fromtimestamp(token_data.exp) < datetime.now():
            raise HTTPException(
                status_code = status.HTTP_401_UNAUTHORIZED,
                detail="Token expired",
                headers={"WWW-Authenticate": "Bearer"},
            )
    except(jwt.JWTError, ValidationError):
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Could not validate credentials",
            headers={"WWW-Authenticate": "Bearer"},
        )
    username: str = token_data.sub
    # get user from database
    # if user does not exist, raise an exception
    # if user exist, return user Schema with password hashed 
    return SystemAccount(**user)


```
Estem definint la `get_current_fucntion` com a dependència la qual agafa `OAuth2PasswordBearer` com a dependència.
Té dos paràmetres obligatoris, `tokenUrl` que és l'URL en la teva aplicació que s'encarrega del login i retorna Tokens i
`scheme_name` que és el nom de l'esquema que s'utilitza per a l'autentificació. En aquest cas, JWT.
Descodifiquem el token i comprovem que no estigui caducat. Si està caducat, retornem un error 401.
El token el guardem en l'schema TokenPayload que té els següents camps:
```python
class TokenPayload(BaseModel):
    sub: str = None
    exp: int = None
```
On `sub` és el subjecte del token, en el nostre cas era l'username i `exp` és el temps d'expiració del token.

SustemAccount és l'schema que representa un usuari del sistema. Hereda d'Account i té un camp password que és un hash de la contrasenya.
```python
class SystemAccount(Account):
    password: str
```


Per provar aquesta dependència, afegiu-la com a dependència en l'endpoint `@app.get('/account')` i proveu-lo:

```
from app.deps import get_current_user

@app.get('/account', summary='Get details of currently logged in user', response_model=SytemAccount
async def get_me(user: SystemAccount = Depends(get_current_user)):
    return user
```
Comproveu en la documentació com ara /account requereix un token per a ser accedit.

Ara que ja tenim feta la dependència, podem fer que els endpoints siguin accessibles només per a usuaris registrats.

### Exercici 2
- Protegiu tots els endpoints perquè només puguin ser accedits per usuaris registrats. Deixeu només els gets que no tinguin informació sensible (p.ex compte d'usuari, comandes, etc) com a públics.
- Modifiqueu la dependència per tal que si un usuari és admin pugui accedir a tots els endpoints.


 Frontend
--------------

Comencem pel disseny del component. En primer lloc, creeu-ne un de nou component anomenat `Login.vue`. A més, aneu al fitxer `index.js` i afegiu la ruta:

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
Si parem atenció a cada botó:

Botó Log In
-------

Comprova si el nostre nom d’usuari i contrasenya ja estan registrats. Per comprovar això, primer hem de desar el nom d'usuari i la contrasenya escrits per l'usuari. Per fer-ho, podem utilitzar `vmodel=”username”` i `vmodel=”password”`:

```html
<div class="form-label-group">
  <label for="inputEmail">Username</label>
  <input type="username" id="inputUsername" class="form-control"
  placeholder="Username" required autofocus v-model="username">
</div>
<div class="form-label-group">
  <br>
  <label for="inputPassword">Password</label>
  <input type="password" id="inputPassword" class="form-control"
  placeholder="Password" required v-model="password">
</div>
```

On el nom d'usuari i la contrasenya són variables creades a `data()-> return`.

```javascript
data () {
  return {
    logged: false,
    username: null,
    password: null,
    token: null
  }
```
          
Per comprovar l’usuari, hem de fer POST a /login per obtenir el token que utilitzarem més endavant:

```javascript
checkLogin () {
  const parameters = 'username=' + this.username + '&password=' + this.password
  const config = {
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  const path = 'http://localhost:8000/login'
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
},
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
        
### Exercici 3:
 
Creeu un mètode getAccount() a `Matches` que faci un GET al backend per mirar si l'usuari, que ha iniciat sessió, és administrador o no i deseu-lo en una variable anomenada `is_admin`.

Botó Create Account
--------------

![image](figures/sessio-5_create-account.png)

Aquest botó és un formulari on l'usuari introduirà les seves dades i les enviarà. El botó Envia crida a un mètode POST on les dades es guarden a la taula de comptes.
Abans d’enviar POST hauríem de plantejar-nos com obtenir les dades a enviar. Per obtenir aquestes dades, podem utilitzar Forms i emmagatzemar-los en un objecte.
En el nostre formulari, hem de recopilar la informació necessària per enviar-la al BE per POST o PUT.
Primer de tot, hem de crear un objecte per emmagatzemar les dades:

```javascript
  creatingAccount: false,
  addUserForm: {
    username: null,
    password: null
  }
```
A continuació, creeu un mètode d'objecte inicialitzador:

```javascript
initCreateForm () {
  this.creatingAccount = true
  this.addUserForm.username = null
  this.addUserForm.password = null
},
```
Després de definir l’objecte, s’ha de completar mitjançant un formulari (<https://bootstrap-vue.org/docs/components/form>).
Després, mitjançant el mètode onSubmit hauríem de cridar el mètode a POST. Finalment, crideu al mètode initForm per reiniciar els paràmetres.

### Exercici 4:

1. Creeu un formulari per desar les dades
2.  Creeu un mètode POST per enviar les noves dades d'usuari mitjançant path i paràmetres:

```javascript
const path = 'http://localhost:8000/account'
```

```javascript
const parameters = {
  username: this.addUserForm.username,
  password: this.addUserForm.password
} 
```

3. Alerta a l'usuari si el compte s'ha creat o ja existeix

Botó Back To Matches
--------------

Torna a la pàgina dels partits, però envia informació diferent amb la query, tal com ho fa el botó SIGN IN. En aquest cas, només cal enviar:

```javascript
this.$router.push({ path: '/' })
```
### Exercici 5:

1. Creeu un mètode per substituir la ruta i enllaceu el botó per tornar als partits.


Comprar amb seguretat
---------------------

Per proporcionar seguretat per a les compres de cada usuari, hauríem d’utilitzar el token obtingut quan l’usuari prem INICIAR SESSIÓ. Per fer-ho, a l’addPurchase (paràmetres) que ja heu creat a l’última sessió, canvieu-lo per:

```javascript
addPurchase (parameters) {
  const path = 'http://localhost:8000/order/${this.username}'
  axios.post(path, parameters, {
    auth: {username: this.token}
  })
    .then(() => {
      ...
    })
    .catch((error) => {
      ...
    })
},
```

Fixeu-vos que estem utilitzant paràmetres d'autenticació amb token com a nom d'usuari.

### Informació d’usuaris i partits

Per deixar clara la interacció de l'usuari, hauríem de mostrar-li la informació següent:

### Exercici 6:

1. A la visualització de partits, mostreu els diners disponibles i les entrades afegides a la cistella. Recordeu que els diners disponibles sempre són controlats per la base de dades i no ha de canviar amb la interacció de VUE, només cal canviar després de comprar entrades o actualitzar la pàgina.

    ![image](figures/sessio-5_user-status.png)
    
2. A la visualització de partits, mostreu les entrades disponibles per a cada partit. Com que els diners disponibles sempre són controlats per la base de dades i no han de canviar amb la interacció de VUE, només canvieu-ho després de comprar entrades o actualitzar la pàgina.
