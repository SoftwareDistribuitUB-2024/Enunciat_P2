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

- Afegir usuari
  - Crides curl per llistar usuaris

```bash
curl -X "GET" "http://127.0.0.1:8000/api/v1/users/?skip=0&limit=100" -H "accept: application/json"
``` 

  - Crida curl per afegir usuari

```bash
curl -X "POST" "http://127.0.0.1:8000/api/v1/users/" -H "accept: application/json" -H "Content-Type: application/json" -d "{\"full_name\": \"John Smith\", \"password\": \"my_password\", \"email\": \"admin@p2.sd.ub.edu\"}"
```

- Comprovar que el password no és llegible
- Fer revisar què passa quan es crea un usuari i vincular amb guia seguretat dades

### Exercici 2: Obtenció token autenticació

- Fer revisar endpoint de login
- Fer crida curl per obtenir JWT

```bash
curl -X "POST" "http://127.0.0.1:8000/api/v1/login/access-token" -H "accept: application/json" -H "Content-Type: application/x-www-form-urlencoded" -d "username=admin@p2.sd.ub.edu&password=my_password"
```

Teniu un mètode a l'API per provar els tokens. Primer, crideu a aquest mètode sense pasar cap token, per tant, sense
estar autenticat:

```bash
curl -X "POST" "http://127.0.0.1:8000/api/v1/login/test-token" -H "accept: application/json"
```

Per fer una crida autenticada amb el token anterior, cal afegir-lo en forma 
de capçalera:

```bash
curl -X "POST" "http://127.0.0.1:8000/api/v1/login/test-token" -H "accept: application/json" -H "Authorization: Bearer **<token>**"
```
on **<token>** és el valor obtingut en la crida anterior.


### Exercici 3: Protecció endpoints

- Protegir endpoint creació usuaris
- Verificar que ara ja no funciona la crida de l'exercici 1
- Explicar com passar JWT
- Crida curl amb token per crear un nou usuari, i veure que si que ara si que funciona.

### Exercici 4: Implementació autenticació al frontend

- Mostrar captura pantalla amb formular autenticació
- Explicar com afegir JWT a axios

## Tasques fora del laboratori

### Tasca 1: Protegir gestió dades de l'aplicació
- Només els administradors poden crear competicions i equips
- Només els organitzadors d'una competició i els administradors poden crear partits
- Qualsevol usuari registrat ha de poder comprar entrades

### Tasca 2: Protegir dades
- Modifica la teva aplicació per tal que els diners disponibles a un compte es guardin de forma encriptada.

### Tasca 3: [Opcional] Transaccions a la base de dades
- Assegurar concurrència respecte nombre entrades i salari disponible. 

