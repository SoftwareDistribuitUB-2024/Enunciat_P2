Sessió 5
=========

Afegint seguretat al nostre backend
--------

En aquesta secció modificarem i afegirem alguns models i funcions al nostre Accounts, per afegir seguretat bàsica i autenticació. En el backend haurem de fer els següents passos:
* Password Hashing 
* Crear i assignar JWT (JSON Web Token) 
* Creació d'Usuaris
* Validació de tokens en cada petició per assegurar l'autentificació.


### Desar la contrasenya d’usuari xifrada. Password Hashing

No és una bona idea emmagatzemar contrasenyes o informació delicada (número de targeta de crèdit, etc.) sense xifrar aquesta informació.
En aquest cas, farem servir la biblioteca python `passlib` per a la comprovació de contrasenyes (` Els valors predeterminats són SHA256-Crypt en sistemes de 32 bits, SHA512-Crypt en sistemes de 64 bits`).

Primer instal·leu `passlib` fent:
	
	    pip3 install passlib


I afegiu aquests dos mètodes a un nou fitxer `utils.py`:

```python
from passlib.context import CryptContext

password_context = CryptContext(schemes=["bcrypt"], deprecated="auto")


def get_hashed_password(password: str) -> str:
    return password_context.hash(password)


def verify_password(password: str, hashed_pass: str) -> bool:
    return password_context.verify(password, hashed_pass)
```

La funció `get_hashed_password` agafa un password en clar i retorna un hash que es pot guardar a la base de dades de forma segura.
La funció `verify_password` agafa un password en clar i un hash i comprova si el password en clar coincideix amb el hash.

### Crear i assignar JWT (JSON Web Token)

Un JWT és un estàndard obert (RFC 7519) que defineix una forma compacta i autònoma per a la transmissió segura d'informació entre parts com a un objecte JSON. Aquesta informació pot ser verificada i confiada perquè està signada digitalment. Els JWT es poden signar mitjançant un algoritme HMAC (clau secreta) o un parell de claus RSA / EC (clau pública i privada).
Usarem la llibreria jose, instal·leu-la fent:

        pip3 install jose

Dins de `utils.py` afegiu:

```python
from pydantic import BaseSettings
from datetime import datetime, timedelta
from typing import Union, Any
from jose import jwt

class Settings(BaseSettings):
    access_token_expire_minutes: int = 30  # 30 minutes
    refresh_token_expire_minutes: int = 60 * 24 * 7 # 7 days
    algorithm: str = "HS256"
    jwt_secret_key: str     # should be kept secret
    jwt_refresh_secret_key: str     # should be kept secret
    
    class Config:
        env_file = ".env"

def create_access_token(subject: Union[str, Any], expires_delta: int = None) -> str:
    if expires_delta is not None:
        expires_delta = datetime.utcnow() + expires_delta
    else:
        expires_delta = datetime.utcnow() + timedelta(minutes=settings.access_token_expire_minutes)
    
    to_encode = {"exp": expires_delta, "sub": str(subject)}
    encoded_jwt = jwt.encode(to_encode, settings.jwt_secret_key, settings.algorithm)
    return encoded_jwt

def create_refresh_token(subject: Union[str, Any], expires_delta: int = None) -> str:
    if expires_delta is not None:
        expires_delta = datetime.utcnow() + expires_delta
    else:
        expires_delta = datetime.utcnow() + timedelta(minutes=settings.refresh_token_expire_minutes)
    
    to_encode = {"exp": expires_delta, "sub": str(subject)}
    encoded_jwt = jwt.encode(to_encode, settings.jwt_refresh_secret_key, settings.algorithm)
    return encoded_jwt

settings = Settings()
```

Fem servir la classe Settings per definir les constants que farem servir per crear els tokens. Aquestes constants es poden canviar en el futur si cal.
Les variables jwt_secret_key i jwt_refresh_secret_key són les claus secretes que farem servir per signar els tokens. Aquestes claus s'han de mantenir secretes i no s'han de compartir amb ningú. En aquest cas, les definim com a variables d'entorn. Per això, heu d'afegir-les al fitxer `.env` (Aquest fitxer no s'ha de compartir amb ningú, per això està en el fitxer `.gitignore`).

```bash:
JWT_SECRET_KEY="Alguna cosa secreta"
JWT_REFRESH_SECRET_KEY="Alguna altra cosa secreta"
```
I instal·leu la llibreria `python-dotenv` fent:

        pip3 install python-dotenv

Quan executeu el servidor si no us troba el fitxer .env, heu d'especificar on es troba el fitxer `.env` fent:

        python3 -m uvicorn main:app --env-file path/fitxer/.env

Recordeu que el fitxer .env està en el gitignore, i per tant l'heu de crear en local cada cop que feu un git clone.

L'única diferència entre les dues funcions és que el temps de caducitat per refrescar els tokens és més llarg que el temps de caducitat dels tokens d'accés. Això és perquè els tokens d'accés s'han de renovar amb més freqüència que els tokens de refresc.
Aquestes funcions reben un subjecte que pot ser qualsevol cosa (en el nostre cas serà un nom d'usuari) i opcionalment un temps de caducitat. Si no s'especifica el temps de caducitat, s'utilitza el temps de caducitat predeterminat que hem definit a la classe Settings.

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

### Exercici 1:

1. Comproveu o modifiqueu Account `__init__` (suprimiu la inicialització de contrasenya falsa) i inicialitzeu-ho només amb valors de nom d'usuari, `available_money` i `is_admin`.
2. Comproveu o modifiqueu els punts finals relacionats amb Accounts que funcionin correctament (GET, POST, DELETE, PUT).:

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
