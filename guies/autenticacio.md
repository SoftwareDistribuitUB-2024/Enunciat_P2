# Autenticació

L'autenticació és el procés pel qual una persona (o un sistema) garanteix que és qui diu ser. Tradicionalment, els sistemes
més utilitzats han estat les contrasenyes (o altres tipus de claus), tot i que en l'actualitat els sistemes
biomètrics estan canviant aquesta tendència.

La informació de qui som s'haurà de proporcionar cada cop que interactuem amb un sistema, per exemple cada cop que fem
una petició a l'API. Estar enviant constantment la nostra clau per garantir la nostra identitat és una exposició d'aquesta
informació que posa en risc la seguretat del sistema. Per evitar-ho, el que es fa és generar "claus" temporals. Un dels
més utilitzats actualment són els **JSON Web Tokens**.

La idea és que inicialment ens autenticarem amb la nostra informació secreta (o biomètrica) per obtenir una clau temporal (**access-token**)
que ens identificarà durant un petit període de temps (generalment uns minuts). D'aquesta manera si algú intercepta aquesta
clau només ens podrà suplantar durant poc temps i amb un abast limitat.

Un cop caduca un token d'accés (**access-token**), caldrà tornar a autenticar-nos per obtenir-ne un de nou. Existeix l'opció
de definir un segon tipus de token anomenat token de refresc (**refresh token**), que té una caducitat més gran (hores o dies) i
que només serveix per renovar un token d'accés caducat.

## Crear i assignar JWT (JSON Web Token)

Un JWT és un estàndard obert ([RFC 7519](https://datatracker.ietf.org/doc/html/rfc7519)) que defineix una forma compacta i autònoma per a la transmissió segura d'informació entre parts com a un objecte JSON. Aquesta informació pot ser verificada i confiada perquè està signada digitalment. Els JWT es poden signar mitjançant un algoritme HMAC (clau secreta) o un parell de claus RSA / EC (clau pública i privada).
Usarem la llibreria jose, instal·leu-la fent:

```bash
poetry add jose
```

A continuació teniu un exemple de codi on podeu veure els paràmetres típics de configuració (settings) i 
com crear els tokens. Fixeu-vos en els camps utilitzats dins del token.

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
L'única diferència entre les dues funcions és que el temps de caducitat per refrescar els tokens és més llarg que el temps de caducitat dels tokens d'accés. Això és perquè els tokens d'accés s'han de renovar amb més freqüència que els tokens de refresc.
Aquestes funcions reben un subjecte que pot ser qualsevol cosa (en el nostre cas serà un nom d'usuari) i opcionalment un temps de caducitat. Si no s'especifica el temps de caducitat, s'utilitza el temps de caducitat predeterminat que hem definit a la classe Settings.

## Integració amb FastAPI

FastAPI té una manera de validar els camps de les peticions per complir amb els estàndards d'OpenAPI. Per això, heu de definir un esquema que tingui els camps que voleu validar i afegir-hi les restriccions que voleu. 
Afegiu el següent endpoint a main.py i feu la lògica demanada.

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
Aquesta dependència ens permet extraure les dades de la petició amb autentificació bàsica HTTP. 
En aquest exemple es fa servir la llibreria `python-multipart`, que permet utilitzar informació codificada com a formularis. Si no 
la teniu instal·lada la podeu afegir amb:

```bash
poetry add python-multipart
```

A la documentació de l'API podreu veure els inputs necessaris per a username i password. L'Schema de TokenSchema és el següent:

```python
from pydantic import BaseModel
class TokenSchema(BaseModel):
    access_token: str
    refresh_token: str
```

# Autorització

Un cop tenim garantida la identitat d'una persona o sistema, entra en joc l'autorització, que és definir quines accions
es poden realitzar o a quina informació es pot accedir. Segons l'aplicació aquest procés pot ser molt simple o molt complex,
però en general trobarem els següents casos:

* **Autenticat/Anònim:** Un filtre molt habitual és diferenciar aquelles parts que són accessibles a tothom o aquelles que 
requereixen estar autenticat per accedir. Aquí és important tenir en compte que alguns endpoints, com el d'autenticació
haurà d'estar obert a tothom.

* **Categories d'usuaris:** Podem assignar permisos a categories d'usuaris, per exemple els administradors podran realitzar
certes tasques que els usuaris normals no podràn.

* **Contextuals:** L'accés a les dades o les accions permeses depenen de les dades mateixes. Per exemple, un usuari pot canviar la seva informació, 
però no la d'un altre usuari.

## Protegir els endpoints

Ara que ja tenim suport per a l'autentificació, podem posar endpoints protegits per tal que només usuaris amb privilegis com l'administrador o usuaris registrats
puguin accedir-hi. Per fer-ho crearem dependències que ens permetran verificar si l'usuari té accés a l'endpoint.
Aquestes dependències es poden afegir com a paràmetres a les funcions dels endpoints, s'executen abans que la funció i poden retornar un valor que s'utilitzarà com a paràmetre per a la funció.

Fixeu-vos en el codi d'exemple següent:
```python
from typing import Union, Any
from datetime import datetime
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
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

async def get_current_user(token: str = Depends(reuseable_oauth), settings: Annotated[utils.Settings, Depends(get_settings)]) -> SystemAccount:
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
Té dos paràmetres obligatoris, `tokenUrl` que és l'URL en la teva aplicació que s'encarrega de l'autenticació i retorna Tokens i
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
Podem definir altres dependències perquè només els administradors puguin accedir, o només certs usuaris en concret.
