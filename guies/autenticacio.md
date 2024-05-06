Autenticació
=========









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
