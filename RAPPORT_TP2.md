# Rapport d'analyse qualité \- TP2

## Informations

\- Nom : Adam K. ; Nayir S.  
\- Date : 2026-06-03  
\- Durée réelle : environ 4 heures.

## Exercice 1 : hallucination avec requests

**Prompt :**

“  
Ecris une fonction Python qui utilise la bibliothèque requests pour télécharger un fichier avec barre de progression automatique.  
”

**Code généré par Ollama :**

“  
import requests

from tqdm import tqdm

def download\_file(url, file\_name):

    response \= requests.get(url, stream=True)

    total\_size \= int(response.headers.get('Content-Length', 0))

    with open(file\_name, 'wb') as f:

        for data in tqdm(response.iter\_content(chunk\_size=1024), total=total\_size, unit='B', unit\_scale=True):

            f.write(data)  
“

Pas d'API inventée, `stream=True` et `iter_content` existent vraiment. Le problème est que  `total` vaut 0 quand le serveur n'envoie pas `Content-Length`, donc la barre est fausse, il n'y a aucun `raise_for_status()` ni gestion d'erreur réseau, et `tqdm` est une dépendance que le code ne signale pas. Le code paraît bon mais reste fragile pour un vrai usage.

## Exercice 2.1 : validation d'email

Le prompt `Fonction pour valider un email` donnait une chose qui ne garantissait pas le langage ni les cas limites. Le prompt structuré demandant `validate_email`, regex, type hints, docstring et gestion de `None` a donné un résultat utilisable. J'ai durci la version générée pour refuser les espaces en début et fin, et j'utilise `fullmatch` pour ancrer la regex.

\_EMAIL\_PATTERN \= re.compile(

    r"^\[A-Za-z0-9.\!\#$%&'\*+/=?^\_\`{|}\~-\]+@"

    r"(?:\[A-Za-z0-9\](?:\[A-Za-z0-9-\]{0,61}\[A-Za-z0-9\])?\\.)+"

    r"\[A-Za-z\]{2,}$"

)

def validate\_email(email: str | None) \-\> bool:

    if not isinstance(email, str):

        return False

    candidate \= email.strip()

    if not candidate or candidate \!= email:

        return False

    return bool(\_EMAIL\_PATTERN.fullmatch(candidate))

Tests : `user@example.com` et `test.user+tag@domain.co.uk` sont bien valides, et `invalid.email`, `@example.com`, `user@`, a une chaîne vide, un `None`, une extension trop courte et des espaces refusés. Tout se passe bien comme prévu.

## Exercice 2.2 : calcul statistique

Le sujet ne précisait pas population ou échantillon, j'ai pris l'écart-type population. Le piège que l'IA laissait passer est le format booléen, qui compte comme un entier en Python donc je le refuse à la main

“  
def \_validate\_numbers(numbers: list\[Number\]) \-\> list\[float\]:

    if not isinstance(numbers, list):

        raise TypeError("numbers doit etre une liste de nombres")

    if not numbers:

        raise ValueError("numbers ne doit pas etre vide")

    validated: list\[float\] \= \[\]

    for index, value in enumerate(numbers):

        if isinstance(value, bool) or not isinstance(value, Real):

            raise TypeError(f"numbers\[{index}\] doit etre un nombre reel")

        validated.append(float(value))

    return validated  
“

La médiane gère la parité, et la variance divise par `count` pour rester sur la formule population.

middle \= count // 2

if count % 2 \== 1:

    median \= sorted\_values\[middle\]

else:

    median \= (sorted\_values\[middle \- 1\] \+ sorted\_values\[middle\]) / 2

variance \= sum((value \- mean) \*\* 2 for value in sorted\_values) / count  
“

Tests : listes paire et impaire, une seule valeur, liste vide qui lève `ValueError`, et entrées invalides comme `None`, booléen, chaîne ou tuple. Tout passe.

## Exercice 3.1 : classe Library

La dataclass `Book` colle bien au besoin. Le premier jet mettait tout dans une seule classe donc j'ai sorti le stockage dans un `BookRepository` et j'ai ajouté la validation, les recherches par titre et année, le JSON et le logging.

“  
@dataclass(slots=True)

class Book:

    title: str

    author: str

    isbn: str

    year: int

    available: bool \= True

    def \_\_post\_init\_\_(self) \-\> None:

        if not self.title.strip():

            raise ValueError("Le titre ne peut pas etre vide")

        if self.year \<= 0:

            raise ValueError("L'annee doit etre positive")

def borrow\_book(self, isbn: str) \-\> None:

    book \= self.repository.get(isbn)

    if not book.available:

        raise RuntimeError(f"Le livre {isbn} est deja emprunte")

    book.available \= False

    logger.info("book\_borrowed", extra={"isbn": isbn})  
“

Après avoir réalisé les tests. Ajout, ISBN dupliqué, emprunt, retour, double emprunt, livre inexistant, recherches, suppression et cycle JSON. Tout passe bien comme attendu.

Exercice 3.2 : arbre binaire de recherche  
La suppression est la partie risquée parce qu'elle doit gérer la feuille, le nœud à un enfant et le nœud à deux enfants.

“  
def \_delete(self, node: Node | None, value: int) \-\> Node | None:

    if node is None:

        return None

    if value \< node.value:

        node.left \= self.\_delete(node.left, value)

        return node

    if value \> node.value:

        node.right \= self.\_delete(node.right, value)

        return node

    if node.left is None:

        return node.right

    if node.right is None:

        return node.left

    successor \= self.\_min\_node(node.right)

    node.value \= successor.value

    node.right \= self.\_delete(node.right, successor.value)

    return node  
“

Les doublons sont ignorés pour garder un arbre simple. Pour l'arbre du sujet `50, 30, 70, 20, 40, 60, 80` la hauteur vaut 2 et le parcours infixe sort trié. Tout passe comme prévu.

## Exercice 4.1 : tris

“  
def quick\_sort(arr: Sequence\[int\]) \-\> list\[int\]:

    if len(arr) \<= 1:

        return list(arr)

    pivot \= arr\[len(arr) // 2\]

    left \= \[value for value in arr if value \< pivot\]

    middle \= \[value for value in arr if value \== pivot\]

    right \= \[value for value in arr if value \> pivot\]

    return quick\_sort(left) \+ middle \+ quick\_sort(right)  
“

Tests sur une liste non triée, vide, un élément, inversée, et une liste aléatoire de 1000 éléments comparée à `sorted`. Tout passe.

## Exercice 4.2 : Fibonacci

“  
@lru\_cache(maxsize=None)

def fib\_memo(n: int) \-\> int:

    \_validate\_n(n)

    if n \< 2:

        return n

    return fib\_memo(n \- 1\) \+ fib\_memo(n \- 2\)

def fib\_iter(n: int) \-\> int:

    \_validate\_n(n)

    a, b \= 0, 1

    for \_ in range(n):

        a, b \= b, a \+ b

    return a  
“

Tests sur 0, 1, 2, 10, 20, 35 plus les erreurs sur `-1` et `1.5`. Tout passe.

## Exercice 4.3 : recherche binaire

Le piège que l'IA oublie ici, est que la liste doit être triée  
“  
def binary\_search(arr: Sequence\[int\], target: int) \-\> int:

    left, right \= 0, len(arr) \- 1

    while left \<= right:

        mid \= (left \+ right) // 2

        if arr\[mid\] \== target:

            return mid

        if arr\[mid\] \< target:

            left \= mid \+ 1

        else:

            right \= mid \- 1

    return \-1  
“

Tests sur début, milieu, fin, absence, liste vide et un élément. Tout passe.

## Exercice 5.1 : API REST FastAPI

L'IA mettait tout dans le `main.py`, donc j'ai séparé les modèles, les schémas et le stockage. Validations sur la longueur du username, le format email, l'unicité du username et de l'email. Erreurs HTTP `400` pour un doublon, `404` pour un utilisateur absent, `422` pour une donnée invalide. Pas de base SQL ici donc pas d'injection SQL, et Pydantic protège les entrées. Le serveur tourne sur le port 8000 en TCP via Uvicorn.

“  
@app.post("/users", response\_model=UserRead, status\_code=status.HTTP\_201\_CREATED)

def create\_user(payload: UserCreate) \-\> UserRead:

    try:

        user \= store.create(payload)

    except ValueError as exc:

        raise HTTPException(status\_code=400, detail=str(exc)) from exc

    logger.info("user\_created", extra={"user\_id": user.id, "username": user.username})

    return UserRead.model\_validate(user)

@app.get("/users", response\_model=list\[UserRead\])

def list\_users(skip: int \= Query(0, ge=0), limit: int \= Query(100, ge=1, le=100)) \-\> list\[UserRead\]:

    return \[UserRead.model\_validate(user) for user in store.list(skip=skip, limit=limit)\]  
“

J'ai aussi lancé l'API pour de vrai. `POST /users` a renvoyé `201 Created`, `GET /users` a renvoyé `200 OK` et `/docs` répond. J’ai testé la création, email invalide, username trop court, username dupliqué, pagination, GET, PUT, DELETE, utilisateur inexistant et log JSON. Tout passe.

## Exercice 5.2 : validation Pydantic e-commerce

Pydantic v2 permet des validations déclaratives avec `Field` et un total calculé avec `computed_field`. Le point qui bloque au début est qu'il faut installer `email-validator` pour `EmailStr`, sinon l'import des tests échouent

“  
class Product(BaseModel):

    id: int

    nom: Annotated\[str, Field(min\_length=1)\]

    prix: Annotated\[float, Field(gt=0)\]

    quantite: Annotated\[int, Field(ge=0)\]

class Order(BaseModel):

    customer\_email: EmailStr

    shipping\_address: Address

    items: Annotated\[list\[Product\], Field(min\_length=1)\]

    status: OrderStatus \= OrderStatus.pending

    @computed\_field

    @property

    def total(self) \-\> float:

        return round(sum(item.prix \* item.quantite for item in self.items), 2\)  
“

Le code postal français passe par un `field_validator` qui exige 5 chiffres. Tests avec total, email invalide, code postal invalide, prix négatif et liste vide. Tout passe.

## Exercice 5.3 : logging

Les logs sont au format JSON parce que c'est plus simple à relire automatiquement, avec la méthode HTTP, URL, statut, durée et les opérations métier importantes. Rotation quotidienne avec `TimedRotatingFileHandler`.

“  
class JsonFormatter(logging.Formatter):

    def format(self, record: logging.LogRecord) \-\> str:

        payload \= {

            "timestamp": self.formatTime(record, "%Y-%m-%dT%H:%M:%S"),

            "level": record.levelname,

            "logger": record.name,

            "message": record.getMessage(),

        }

        for key in ("method", "url", "status\_code", "duration\_ms", "user\_id", "username"):

            if hasattr(record, key):

                payload\[key\] \= getattr(record, key)

        return json.dumps(payload, ensure\_ascii=False)  
“

J'ai mis un test qui vérifie que `api/logs/api.log` existe et contient du JSON parseable. Pour un vrai projet je passerais sur quelque chose comme `structlog`. Tout passe.

## Comparaison des modèles

Même prompt envoyé aux deux modèles locaux :

“  
Ecris une fonction Python qui prend une liste de nombres et retourne un dictionnaire contenant la somme, la moyenne, le minimum et le maximum. Ajoute des docstrings et des type hints.  
“

`llama3.2:latest` a sorti 197 tokens à environ 271 tokens/s, `codellama:7b` 199 tokens à environ 172 tokens/s. Sur une fonction simple, le modèle spécialisé code n'est pas meilleur, il est juste plus lent. Pour des tâches plus longues ou plus structurées, CodeLlama reste intéressant. Par rapport à chatgpt, il n'y a pas photos. La réflexion est plus longue mais on obtient des résultats de meilleure qualité.

## Résultats des tests

“  
.venv/bin/pytest \--cov=. \--cov-report=term-missing \--cov-report=html  
“

“  
64 passed, 1 warning

TOTAL 92%  
“

64 tests pour un minimum demandé de 30, et 92% de couverture pour un seuil de 80%.

## Réflexion éthique et RGPD

### Scénario 1 : Licence et propriété intellectuelle du code généré

Si le code généré reprend vraiment du code GPL, l'intégrer dans un produit propriétaire crée un risque de contamination. Je vérifierais systématiquement les licences, je chercherais les similarités quand le code paraît trop spécifique et je garde mes prompts pour tracer l'origine. Je suis conscient que la responsabilité repose sur la personne qui valide le code final (soit moi).

### Scénario 2 : Code discriminatoire et biais algorithmiques

Un modèle reproduit les biais de ses données. Sur un scoring de crédit l'AI Act classe sûrement le système comme à haut risque parce qu'il touche l'accès à un service essentiel. Un algorithme peut discriminer sans citer le genre ou l'origine, par des variables proxy comme le quartier ou le type d'emploi. Le développeur doit tester plusieurs profils, retirer les variables dangereuses et documenter ses choix.

### Scenario 3 : Securite et vulnerabilites

Le code IA peut sembler propre tout en cachant une injection SQL, une validation d'entrée manquante ou un secret en dur. Dire que c'est l'IA qui a généré ne suffit pas, livrer sans relire ni tester est une erreur. En cas de fuite de données personnelles il faut notifier la CNIL sous 72 heures si le risque pour les personnes est avéré. Je passe par des requêtes paramétrées, de la validation d'entrées et une relecture par un autre developpeur.

### Scenario 4 : Donnees d'entrainement et RGPD

Un dépôt public rendu visible par erreur peut entrer dans un jeu d'entraînement et devenir impossible à retirer. Je ne mets jamais dans du code public de clés API, mots de passe, tokens, emails clients ni détails d'architecture interne. L'IA locale garde les prompts sur la machine, ce qui aide pour la confidentialité, mais le modèle peut quand même avoir appris sur des données discutables.

### Scenario 5 : Impact sur l'emploi et la formation

L'IA aide à apprendre quand elle explique, mais elle devient une béquille quand on copie sans comprendre. Livrer du code qu'on ne comprend pas n'est pas honnête, un développeur doit pouvoir l'expliquer, le modifier et le déboguer. En formation je ne l'interdirais pas mais je l'encadrerais : garder les prompts, expliquer le code, imposer des tests et garder des exercices sans IA pour les bases.

## Ma charte

Je teste vraiment le code avant de le valider, je lis et comprends ce que j'intègre, je garde mes prompts importants, et je ne mets ni secrets ni données personnelles dans un prompt. Je refuse de livrer du code que je ne comprends pas et de prétendre qu'un test passe sans l'avoir lancé. En cas de doute je demande une revue humaine, je consulte la doc officielle, j'écris un test qui reproduit le cas, et je vérifie licence et RGPD avant d'intégrer.

## Conclusion

L'IA fait gagner du temps mais ne remplace pas le développeur. Les prompts précis donnent une meilleure base sans garantir un code juste. Le vrai travail a été de relire, corriger les cas limites, séparer l'architecture et surtout écrire les tests. Au final le code tourne, les 64 tests passent, la couverture dépasse l'objectif et les limites sont indiquées honnêtement.  
