Introducció a l'ús de bases de dades
=========

En aquesta guia sessió, afegirem la persistència de dades mitjançant un sistema de base de dades definit, modificarem els nostres mètodes GET, POST, PUT i DELETE per gestionar partits, estadis i equips amb persistència de dades i crearem nous mètodes per gestionar els `Teams` dins d `Matches`. Finalment, també afegirem nous endpoints relacionats amb aquests nous mètodes.

SQLAlchemy: definició de l'estructura de dades mitjançant models
------------------------------------------------

Els models de dades s’utilitzen en diferents aplicacions que gestionen l’emmagatzematge de dades i determinen com es poden organitzar i manipular les dades. Els més populars són els models relacionals, que fan servir un format basat en taules.

### Configuració SQLAlchemy

Per tal de crear models a la part superior de la nostra base de dades (ORM), utilitzarem un paquet Python anomenat SQLAlchemy. SQLAlchemy ens permet emmagatzemar i recuperar dades mitjançant objectes orientats a objectes amb la seva classe Model. Aquesta classe converteix les dades en objectes Python obtenint un codi compatible amb diferents bases de dades relacionals.
Instal·larem el sqlalchemy amb pip:


    pip install sqlalchemy


I afegirem nous fitxers en el nostre projecte:

- Creeu un fitxer buit anomenat `__ìnit__.py` en el directori principal del projecte (mateixa carpeta que `main.py`). Aquest fitxer és necessari perquè Python reconegui el directori com a un mòdul.
- Creeu un fitxer `database.py` en el projecte principal (mateixa carpeta que `main.py`)
amb aquest contingut:

```python
  
    from sqlalchemy import create_engine
    from sqlalchemy.ext.declarative import declarative_base
    from sqlalchemy.orm import sessionmaker

    SQLALCHEMY_DATABASE_URL = "sqlite:///./data.db"
    # SQLALCHEMY_DATABASE_URL = "postgresql://user:password@postgresserver/db"

    engine = create_engine(
        SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False} ) 
    #check_same_thread...is needed only for SQLite. It's not needed for other databases. 
    SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

    Base = declarative_base()
```
En aquest fitxer estem definint una base dades de tipus SQLite. SQLite és una base de dades relacional
però guardada en un sol fitxer, `data.db`en el nostre cas. 
A continuació definim l'engine d'SQLAlchemy, que és el component que ens permet interactuar amb la base de dades.
Aquest engine es defineix amb la URL de la base de dades i els paràmetres de connexió. En aquest cas, estem utilitzant SQLite, però també podem utilitzar PostgreSQL, MySQL, Oracle, etc. Si voleu utilitzar PostgreSQL, descomenteu la línia 7 i comenteu la línia 6.
També creem la sessió de la base de dades, SessionLocal, que usarem per a cada petició que es faci a la nostra API. Finalment, creem la base declarativa, que és la classe base que ens permetrà definir els models.

### Definint els Models de la base de dades

SQLAlchemy utilitza el terme "model" per referir-se a aquestes classes i instàncies que interactuen amb la base de dades.  
Però Pydantic també utilitza el terme "model" per a referir-se a coses diferents, les classes i instàncies de validació de dades, conversió i documentació. Aquestes segones, sovint es coneixen com a esquemes de dades (schema).
Crearem els models de la nostra base de dades en un nou fitxer anomenat `models.py` en el mateix directori que `database.py` i `main.py`. Aquest fitxer contindrà la definició de les nostres taules i els seus camps.

A `models.py` usarem la classe `Base` creada anteriorment:

``` python
from sqlalchemy import Boolean, MetaData, Column, ForeignKey, Integer, String, Date, Float, Enum, UniqueConstraint, Table
from sqlalchemy.orm import relationship

from database import Base
```

I prenent com a referència l’estructura de dades de la sessió anterior, crearem una taula d’equips amb 3 variables de classe
(`id, name, country`) corresponent a les diferents columnes de la taula.
Definirem l'estructura "Teams" que defineix el nom de la taula, el nom, el tipus i el nombre de columnes, com ara:

```python

class Team(Base):
    __tablename__ = 'teams' #This is table name

    id = Column(Integer, primary_key=True)
    name = Column(String(30),unique=True, nullable=False, index=True)
    country = Column(String(30),nullable=False)
    description = Column(String(100))
```

Mireu aquesta estructura:

-   `id` es defineix com a  `Integer` i `primary_ key= True`. Això indica que aquest valor s'utilitzarà per enllaçar amb altres taules (en el nostre cas, un enllaç entre "equips" i "partits"). Aquest valor es defineix com a `unique` per defecte. En cas contrari, no estarem segurs de quin esdeveniment està relacionat amb quin equip.

-   `name`, `country` i `description`,  estan definits com a `String` amb una mida màxima de 30 i de 100. No és obligatori definir una longitud màxima però es recomana. A més a més, `name`és `unique` i no pot ser `null`, com si fos una clau primària. També és `index` per tal de poder fer consultes més ràpides. Pel que fa a `country` no pot ser null però `description` sí perquè no és un camp obligatori.


### Definint relacions entre models

Les relacions entre models a SQLAlchemy són enllaços entre dos o més models que permeten als models referenciar-se automàticament.

En el nostre model ER, tenim dos tipus de relacions:

* "Many-To-Many" o "molts a molts" entre equips i competicions, ja que un equip pot participar en una o més competicions i una competició està present en un o més equips. En SQLAlchemy es tradueix de la següent manera:

```python
teams_in_competitions = Table("teams_in_competitions",Base.metadata,
                                 Column("id", Integer, primary_key=True),
                                 Column("team_id", Integer, ForeignKey("teams.id")),
                                 Column("competition_id",Integer, ForeignKey("competitions.id")))

class Competition(Base):

    __tablename__ = 'competitions'  # This is table name
    __table_args__ = (UniqueConstraint('name', 'category', 'sport'),)

    id = Column(Integer, primary_key=True)
    name = Column(String(30), nullable=False)
    category = Column(Enum(*categories_list),nullable=False)
    sport = Column(Enum(*sports_list),nullable=False)
    teams = relationship("Team",secondary=teams_in_competitions,backref="competitions")

```

* "One-To-Many" o "d'1 a molts" entre un match i una competició, ja que un match sempre formarà part d'una sola competició, però aquesta competició pot tenir un o més matches. També tenim present aquesta relació entre Teams i Matches: un match tindrà sempre un equip local i un equip visitant (_Child class_), però un equip està present en un o més matches (_Parent class_). Aquest tipus de relació no necessita una taula secundària de suport com la de `teams_in_competitions_`. En SQLAlchemy es tradueix de la següent manera:

```python
class Match(Base):
    __tablename__ = 'matches' #This is table name
    __table_args__ = (UniqueConstraint('local_id', 'visitor_id', 'competition_id', 'date'),)

    id = Column(Integer, primary_key=True)
    date = Column(DateTime, nullable=False)
    price = Column(Float, nullable=False)
    competition_id = Column(Integer, ForeignKey("competitions.id"), nullable=False)
    competition = relationship("Competition",backref="matches")

    local_id = Column(Integer, ForeignKey("teams.id"), nullable=False)
    visitor_id = Column(Integer, ForeignKey("teams.id"), nullable=False)
    local = relationship("Team", foreign_keys=local_id)
    visitor = relationship("Team", foreign_keys=visitor_id)
```

De manera general, una relació "Many-to-one" segueix el següent esquema:

```python
class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)
    child_id = Column(Integer, ForeignKey('child.id'))
    child = relationship("Child")

class Child(Base):
    __tablename__ = 'child'
    id = Column(Integer, primary_key=True)
```

Refent els esquemes/models `Pydantic`
------------------------------------

Ara que ja tenim els models, podem definir els esquemes de `Pydantic` que ens permetran validar les dades que rebem del client.
Tenim els esquemes que vam definir a la sessió anterior a `schemas.py`, però ara els hem de refer perquè siguin compatibles amb els models que hem definit a `models.py`.
Creem els esquemes Base per a cada model, que tenen atributs en comú, tant per a crear com per a llegir dades:

```python
import enum
from models import sports_list, categories_list
from pydantic import BaseModel
from typing import Optional
from datetime import datetime

class TeamBase(BaseModel):
    name: str
    country: str
    description: Optional[str] = None

class TeamCreate(TeamBase):
    pass


class Team(TeamBase):
    id: int

    class Config:
        orm_mode = True


class CompetitionBase(BaseModel):
    name: str
    category: enum.Enum('category',dict(zip(categories_list,categories_list)))
    sport:  enum.Enum('sport',dict(zip(sports_list,sports_list)))
    
class CompetitionCreate(CompetitionBase):
    pass


class Competition(CompetitionBase):
    id: int
    teams: list[Team] = []

    class Config:
        orm_mode = True


class MatchBase(BaseModel):
    date: datetime
    price: float
    local: TeamBase
    visitor: TeamBase
    competition: CompetitionBase


class MatchCreate(MatchBase):
    pass


class Match(MatchBase):
    id: int
    local: Team
    visitor: Team
    competition: Competition

    class Config:
        orm_mode = True
```

Les classes per llegir dades de la base de dades (`Team`, `Competition` i `Match`) tenen un atribut `Config` que ens permet especificar que volem que els atributs de la classe s'assignin automàticament a partir de les dades de la base de dades. Això ens permetrà utilitzar els esquemes per a llegir dades sense haver de fer cap transformació. Els utilitzarem per a retornar les dades a la resposta de la API.
Les classes per crear dades (`TeamCreate`, `CompetitionCreate` i `MatchCreate`) no tenen aquest atribut i hereden directament de les classes Base (`TeamBase`, `CompetitionBase` i `MatchBase`). Aquestes classes només es fan servir per a validar les dades que rebem del client.

Fitxer `repository.py`
----------------------

En aquest fitxer posarem les utilitats que ens permetran interactuar amb la base de dades. Bàsicament posarem les utilitats
per poder crear, llegir, actualitzar i esborrar dades de la base de dades (CRUD, en anglès: Create, Read, Update, Delete)

Importarem la sessió de SQLAlchemy per interactuar amb la BD, els models i els esquemes de `Pydantic` que hem definit anteriorment:

```python
from sqlalchemy.orm import Session
import models, schemas

def get_team(db: Session, team_id: int):
    return db.query(models.Team).filter(models.Team.id == team_id).first()

def get_team_by_name(db: Session, name: str):
    return db.query(models.Team).filter(models.Team.name == name).first()

def get_teams(db: Session, skip: int = 0, limit: int = 100):
    return db.query(models.Team).offset(skip).limit(limit).all()

def create_team(db: Session, team: schemas.TeamCreate):
    db_team = models.Team(name=team.name, country=team.country, description=team.description)
    db.add(db_team)
    db.commit()
    db.refresh(db_team)
    return db_team
```
