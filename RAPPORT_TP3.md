# TP3 : Tests et Qualite de Code avec IA \- Version Detaillee

## Informations

\- Nom : Adam K. ; Nayd S.  
\- Date : 2026-06-04  
\- Durée réelle : environ 5 heures.

## Tests unitaires de calculate\_discount

La fonction prend un prix, un pourcentage de réduction et un type de client parmi `regular`, `vip` et `employee`. Le bonus VIP et employé utilise le montant de la remise, pas le prix.

“  
base\_discount \= price \* (discount\_percent / 100\)

if customer\_type \== "vip":

    base\_discount \*= 1.1

elif customer\_type \== "employee":

    base\_discount \*= 1.2

elif customer\_type \!= "regular":

    raise ValueError(f"Invalid customer type: {customer\_type}")

return price \- base\_discount  
“

Chaque test sont prévu pour préparer ses propres données donc ils restent indépendants

Le cas métier intéressant est qu'une remise de 100% sur un client VIP ou employé donne un prix négatif parce que le bonus pousse la remise au-delà du prix

“  
@pytest.mark.parametrize("customer\_type,expected", \[("vip", \-10.0), ("employee", \-20.0)\])

def test\_full\_discount\_with\_bonus\_can\_make\_negative\_price(self, customer\_type, expected):

    result \= calculate\_discount(100.0, 100.0, customer\_type)

    assert result \== pytest.approx(expected, rel=1e-9)  
“  
Le code respecte la spec fournie mais côté métier il faudrait sûrement plafonner le prix final à 0

## Tests d'intégration de l'API Flask

L'API gère des utilisateurs avec `POST /users`, `GET /users/<id>`, `PUT /users/<id>` et `DELETE /users/<id>`, stockés dans un dictionnaire en mémoire. Je teste via le client de test Flask, donc sans ouvrir de vrai port réseau. Les tests sont groupés par endpoint plus un scénario CRUD et une fixture `reset_db` appelle `reset_database()` avant chaque test pour isoler la base.

“  
def reset\_database():

    global users\_db, next\_id

    users\_db \= {}

    next\_id \= 1  
“

Je vérifie les codes HTTP, la structure JSON, la lecture d'un utilisateur présent ou absent, la mise à jour partielle et la suppression. La validation d'email de l'API est faible parce qu'elle vérifie seulement la présence d'un `@` donc une vraie validation sur le `PUT` serait à ajouter

## Résultats des tests

PYTHONPATH=. pytest tests \-q \--cov=src \--cov-report=term-missing \--cov-report=html:coverage\_html

Name                                Stmts   Miss  Cover

src/\_\_init\_\_.py                         0      0   100%

src/api.py                             48      0   100%

src/calculator\_refactored.py           57      0   100%

src/discount.py                        13      0   100%

src/order\_processor\_refactored.py      76      0   100%

TOTAL                                 194      0   100%

62 passed in 0.13s

62 tests passent et la couverture est de 100% sur le code testé. Le rapport HTML est dans `coverage_html/`. Après, 100% de couverture ne prouve pas l'absence de bug par contre. Juste que les lignes ciblées ont été exécutées.

## Analyse qualité et refactoring de la calculatrice

`calculator_original.py` cumule les code smells. Du code dupliqué entre `calc` et `calc2`, trop de paramètres, des responsabilités mélangées entre calcul, affichage, écriture fichier et erreurs, des erreurs renvoyées sous forme de chaînes et des noms peu clairs. 

“  
src/calculator\_original.py

    F 4:0 calc \- B (9)

    F 25:0 calc2 \- B (7)

Average complexity: A (3.08)  
“

J'ai refactorisé en séparant les rôles. Un enum `Operation`, une exception `CalculationError`, un `ResultLogger` pour le logging et l'écriture fichier, et une classe `Calculator` pour la logique, le tout avec type hints et docstrings. Le code est plus long mais chaque bloc reste simple.

“  
Average complexity: A (2.32)  
“

La complexité moyenne baisse de 3.08 à 2.32 et la testabilité est meilleure.

## Revue de code de OrderProcessor

Le code original manipule des dictionnaires imbriqués fragiles et débite la carte en modifiant directement son solde, sans validation ni gestion d'erreur robuste.

“  
def charge\_card(self, card, amount):

    if card\['balance'\] \>= amount:

        card\['balance'\] \= card\['balance'\] \- amount

        return True

    return False  
“

Les problèmes majeurs sont le paiement non sécurisé, l'absence de validation des commandes, la violation de la responsabilité unique, un calcul de revenu basé sur un total mutable au lieu d'un calcul dynamique, l'annulation mal contrôlée et l'usage de `print` au lieu d'un logger. La version corrigée utilise des dataclasses `Order` et `OrderItem`, des services injectés pour le paiement, l'email et l'inventaire, une exception `OrderValidationError`, une validation explicite, des logs, un revenu calculé dynamiquement et des tests avec mocks pour les services externes.

## Réflexion éthique, légale et RGPD

### Fausse sécurité de la couverture

Le scénario qui me parle le plus est celui de la fausse confiance donnée par la couverture. J'ai 100 pour cent ici, mais ce chiffre dit seulement que les lignes sont exécutées, pas que le comportement métier est juste. Dans la santé, la finance ou le transport, un test peut afficher une couverture parfaite tout en oubliant une interaction médicamenteuse ou un seuil critique. Le cas du prix négatif avec remise 100 pour cent le montre bien, la couverture reste parfaite mais la règle métier est discutable. L'IA aide à générer une base de tests mais ne remplace pas l'analyse humaine.

### Données de test et RGPD

Utiliser des données réelles dans les tests peut violer le RGPD, surtout si l'environnement de test est moins protégé que la production. J'utilise des données synthétiques, pas de données clients, et je garde les environnements isolés. Une procédure de suppression des données de test resterait à formaliser en entreprise.

### Code et secrets envoyés à une IA cloud

Envoyer du code à un service cloud peut exposer des secrets ou de la propriété intellectuelle. Il faut scanner les secrets, exclure le code sensible, et préférer un modèle local quand la confidentialité l'exige. Je ne mets jamais de secret dans un fichier de test.

### Recommandations

Une revue humaine des tests générés doit être obligatoire pour éviter les tests superficiels qui vérifient juste que le programme ne plante pas. Une politique de données de test doit interdire la production non maîtrisée et imposer des données synthétiques ou anonymisées. Enfin une gouvernance IA doit définir quels outils sont autorisés, sur quel code, avec quelles garanties, et tenir un registre des usages pour les audits.

## Conclusion

Les tests automatisés générés par IA ne suffisent pas seuls pour des systèmes critiques. Ils accélèrent la production et détectent certains cas, mais ils ne comprennent pas toujours le métier et peuvent créer une fausse confiance. Le bon compromis est d'utiliser l'IA comme assistant puis d'imposer une validation humaine renforcée sur les parties sensibles. Au final le TP est livré avec 62 tests qui passent, 100 pour cent de couverture sur le code testé, une calculatrice refactorisée qui descend de 3.08 à 2.32 en complexité, une revue de OrderProcessor corrigée, de la documentation et une réflexion RGPD, le tout vérifié par exécution réelle de pytest, pytest-cov et radon.  
