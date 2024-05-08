Introducció a la seguretat de les dades
=========

No és una bona idea emmagatzemar contrasenyes o informació delicada (número de targeta de crèdit, etc.) sense habilitar
mecanismes de seguretat per aquesta informació. Trobarem dos grans casos segons les necessitats:
- **Necessitem la informació real:** Si la informació que guardem és necessària (i.e. dades targeta de credit, compte bancari, ...) 
necessitarem realitzar un **xifrat**, que permet desxifrar les dades quan les necessitem.
- **No necessitem la informació real:** Si no necessitem la informació, sinó només poder validar-la (i.e. contrasenyes o documents), el que
aplicarem serà un mecanisme de **hash** unidireccional, que no permet recuperar la informació original. En alguns casos,
el resultat es pot **xifrar**, com en el cas de les signatures electròniques.


### Guardar informació sensible. Xifratge d'informació.

Hi ha diferents situacions que poden requerir xifrar informació, tant per compartir-la a través de la xarxa, com 
per emmagatzemar-la de forma segura a la base de dades. A mode resum, podem trobar dues grans famílies de mètodes de xifrat:

**Xifrat simètric:** S'utilitza una mateixa clau per xifrar i per desxifrar. Aquests mètodes ens poden ajudar a guardar
informació de forma segura a la base de dades. Qui té la clau pot xifrar i desxifrar.

**Xifrat asimètric:** S'utilitza un clau direfent per xifrar i per desxifrar. Aquests mètodes s'utilitzen ampliament en
sistemes de clau pública, com per exemple la signatura digital de documents.

Una llibreria que us proporciona gran varietat de mètodes criptiogràfics és la llibreria [**cryptography**](https://cryptography.io/en/latest/), que podeu
instal·lar com:

```bash	
poetry add cryptography
```

A continuació teniu uns mètodes que exemplifiquen com utilitzar xifrat simètric per xifar/desxifrar una informació. 
Abans de guardar una informació confidencial a la base de dades (per exemple el número de targeta de crèdit) es xifraria 
aquesta informació, i quan es necessiti accedir, es llegirà de la base de dades i es desxifrarà.

```python
from cryptography.fernet import Fernet

def get_key():
   return Fernet.generate_key()

def encrypt(key, data):   
   f = Fernet(key)
   return f.encrypt(data)
   
def decrypt(key, data):
   f = Fernet(key)
   return f.decrypt(data)

```

La funció `get_key` genera una nova clau de xifrat. Aquesta clau s'ha de passar de forma segura a l'aplicació, ja que 
si es perd no es podrà accedir a la informació xifrada. Si ens roben la clau, podran accedir a tota la informació de la
base de dades.

La funció `encrypt` agafa una dada "en clar" i una clau, i xifra la dada utilitzant la clau. En el cas més simple,
aquesta dada pot ser un simple ```string```, però podem xifrar un objecte sencer, per exemple, utilitzant un JSON serialitzat.

La funció `decrypt` agafa una dada "xifrada" i una clau, i desxifra la dada utilitzant la clau.

### Desar la contrasenya d’usuari. Password Hashing.

En el cas de les contrasenyes, no xifrarem, sinó que el que farem és fer un **hash** de la contrasenya original. Per
fer-ho més segur, s'acostuma a afegir una informació secreta a la contrasenya (salt), i el procés de hash es repeteix
unes quantes vegades. D'aquesta manera es dificulta que es pugui realitzar atacs per força bruta. Podeu veure un exemple
d'aquest mecanisme en la majoria dels sistemes **Unix/Linux**, que guarden les contrasenyes dels usuaris en el
fitxer ```/etc/shadow``` utilitzant aquest sistema.


Podem fer servir la biblioteca python [**passlib**](https://passlib.readthedocs.io/en/stable/index.html) per a la comprovació de contrasenyes tradicionalment s'ha utilitzat
l'algorisme **SHA** (SHA256 o SHA512), tot i que actualment per aquesta tasca pren força el mètode **bcrypt**, que és
més lent i per tant dificulta encara més trencar la seguretat.

Primer instal·leu `passlib` fent:

```bash	
poetry add passlib
```

Per a la gestió de contrasenyes, fixeu-vos en els següents mètodes:

```python
from passlib.context import CryptContext
from passlib.pwd import genword

password_context = CryptContext(schemes=["bcrypt"], deprecated="auto")


def get_password_hash(password: str) -> str:
    return password_context.hash(password)


def verify_password(password: str, hashed_pass: str) -> bool:
    return password_context.verify(password, hashed_pass)


def new_password() -> str:
    return genword(entropy=52, charset="hex")
```

La funció `get_hashed_password` agafa una contrasenya en clar i retorna un hash que es pot guardar a la base de dades de forma segura.

La funció `verify_password` agafa una contrasenya en clar i un hash i comprova si la contrasenya en clar coincideix amb el hash.

La funció `new_password` permet generar una nova contrasenya amb un cert nivell de seguretat (entropia).

**Nota:** El context defineix les opcions a utilitzar per crear el hash ("bcrypt" en l'exemple anterior). Pot ser necessari 
instal·lar paquets addicionals segons l'opció. En aquest cas, pot requerir afegir el paquet **bcrypt**:

```bash	
poetry add bcrypt
```
