# Connectant Vue i FastAPI


Ara hem de connectar el Framework de Frontend de VUE (node.js) amb el Framework de Backend de FastAPI (python). Però primer haurem de permetre que es puguin fer "requests" des del client (javascript del navegador) al servidor web. Per això haurem de permetre-ho en el nostre backend de FAST-API.

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
```

D'aquesta forma esteu concedint accés a l’aplicació FastAPI des de l’exterior.

### Configurant els directoris static de FastAPI.

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
