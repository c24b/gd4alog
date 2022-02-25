# Elastic Search

## Stratégies de recherche

### Quelle pertinence?

> Le curseur doit etre positionné entre **généralité et spécificité**:
>
> - trop strict il ne retourne pas de resultats
>
> - trop large les resultats  ne sont pas pertinents
La paramétrage de la recherche influe sur le score de pertinence

### FAQ et Guide au paramétrage

Quelles questions générales pour guider les choix dans la stratégie de recherche 

et donc dans le paramêtrage:

- quelle **type de recherche**: recherche libre, recherche par termes ou recherche par facettes?
  - recherche plein text: la recherche est approximative, considérée dans son ensemble (full text queries, including fuzzy matching and phrase or proximity queries) **par défault** 
  - recherche par termes: la recherche examine chacun des mots et définit les conditions explicitement
  - recherche par facettes: la recherche prend en compte le type de la requete: date, entier, range ...

- quelle **degré de précision**: doit on prendre en compte les fautes de frappe? ouvrir les potentiels synonymes?
    - proximity search
    - sentence/word/regex
    - fuzzy search
    - terms_expansion

- une **recherche avec un seul terme** doit elle retourner un match exact?
  - query or terms, fuzzy search
  - proximity research 
  
- une **recherche avec plusieurs termes** doit elle préciser la recherche ou au contraire l'élargir?
    - boolean OR ou AND / query_string

- une **recherche avec plusieurs termes** doit elle absolument contenir tous les termes?
  - AND OR
  - must or should
  - minimum should match


- définir les **champs de recherche** : doit on chercher simplement dans la description et le nom ou doit-on élargir aux filtres thématiques?
    - champs de recherche
    - poids des différents champs de recherche
    - boost

- **exclusion**: doit on permettre l'exclusion de certains termes en utilisant l'opérateur `-`
  - NOT

- **recherche multilingue**: une recherche en français doit elle renvoyer des resultats correspondants en anglais (et vie et versa)
  - rechercher par défaut dans un index puis si pas de résultats dans un autre
  - deux indexes séparé

- **seuil de pertinence**: doit-on renvoyer tous les résultats ou supprimer les moins pertinenst:
    - définition d'un seuil minimal selon les score de pertinence

- la recherche  dans la barre doit elle se combiner avec les filtres selectionnés?
  
## Propositions

* **Barre de recherche**: 

recherche plein text par défault sur les champs titre, description et tous les champs de la section DATA

highlight sur la description et le titre (apparaissent seulement dans la liste des résultats en texte  contrairement aux filtres
affichage du score de pertinence *10

* **Filtres**: 

recherche multi termes booléenne avec un minimum_should_match à 90%: à discuter
pas de highlight: affiche des resultats ou pas
  


## Définition de Pertinence et calcul du score

La recherche par défault dans Elastic Search repose sur le [Practical Scoring Function](https://www.elastic.co/guide/en/elasticsearch/guide/current/practical-scoring-function.html#practical-scoring-function) du moteur Lucene et sa forme concrète du score de pertinence est principalement fondée sur une combinaison de plusieurs facteurs :

### le **modèle booléen**
  
Le modèle booléen applique simplement des règles de logique (booléenne): les conditions de filtre `OR` `AND` `NOT`, le score part du principe que "plus ca matche meilleur c'est". 

Dans le cas d'Elastic Search, la logique booléenne est étendue avec la clause "filter", des filtres qui s'appliquent après la recherche et l'obtention du score.


**OR**

Rechercher: chlordécone OU banane
```
{
  "query": {
    "bool": {
      "should": [
        {"term": { "text": "chlordécone" }},
        {"term": { "text": "banane"   }}
      ]
    }
  }
}
```


**AND**

Rechercher: chlordécone ET banane
```
{
  "query": {
    "bool": {
      "must": [
        {"term": { "text": "chlordécone" }},
        {"term": { "text": "banane"   }}
      ]
    }
  }
}
```

**NOT**


Rechercher: chlordécone SANS banane

{
  "query": {
    "bool": {
      "must": [
        {"term": { "text": "chlordécone" }},
      ],
      "must_not":
        {"term": { "text": "banane"   }}
      ]
    }
  }
}

**FILTER**


On va rechercher chlordécone ET banane dans la catégorie "Alimentation": 
on va obtenir le même score que **AND** mais on va filtrer les résultats obtenus a posteriori en choississant seulement ceux qui sont dans la categorie alimentation

Rechercher: chlordécone ET banane FILTRER categorie ALIMENTATION
{
  "query": {
    "bool": {
      "must": [
        {"term": { "text": "chlordécone" }},
        {"term": { "text": "banane"   }}
      ]
    },
    "filter": [
        {"category":"Alimentation}
    ]
  }
}


### le **modèle TF/IDF**

TF/IDF = `Term Frequency/Inverse Document Frequency (TF/IDF)`

qui prend 3 parametres en compte:

- La **fréquence du terme** recherché dans le document. Plus le terme est fréquent, plus son poids sera élevé.

Nombre d'apparitions du terme t dans le document / Nombre total de termes dans le document

`tf(t in d) = √frequency`

- La **fréquence inverse du terme** à travers tous les documents. Plus le terme est fréquent, moins il aura de poids.

logarithme (en base 10 ou en base 21) de l'inverse de la proportion de documents du corpus qui contiennent le terme
`idf(t) = 1 + log ( numDocs / (docFreq + 1))`

Le score TDF-IDF s'obtient en multipliant les 2 parametres.

- Dans le cas de la recherche plein texte, on pondère par la **mesure de longueur du champ**. 

Plus le champ est grand, plus le poids sera faible ; inversement, plus le champ est petit, plus le poids sera élevé.
La norme de longueur de champs est l'inverse de la racine carré du nombre de termes dans le champs.

`norm(d) = 1 / √numTerms`

Le score de ElasticSearch dans le cas d'une recherche plein texte peut être pondéré par ce calcul. 
Il est utile dans certains cas de le désactiver par exemple pour une recherche d'un composant chimique spécifique, en effet on cherche plutot l'occurence exacte que la longueur du terme.

##### Le **modèle espace vectoriel** (Space Model Vector)

Dans le cas de recherche multi-termes, le modèle d'espace vectoriel permet de comparer les différents termes à travers un document.
Pour se faire, on trasnforme le document et la recherche en vecteurs unidimensionnel.

Chaque nombre dans le vecteur consiste dans le poids du terme calculé avec TF/IDF.

On compare ensuite l'angle (la distance entre les deux axes) entre les vecteurs: 
- si l'angle est  large entre le vecteur du document considéré et celui de la requete alors la pertinence est faible 
- si l'angle est  resserré alors la pertinence est forte

cela prend la forme d'un score qui pondère lui aussi le score de pertinence


## Un coup de boost?

Quand on a plusieurs champs, il peut être intéressant de donner davantage de poids à un ou plusieurs champs spécifiques, c'est l'option `boost` proposé dans ElasticSearch.

> Dans notre cas précis on peut par exemple donner un poid plus fort au match (la correspondance ) sur le champs thématique que sur le titre par exemple

Il y a deux endoits ou l'on peut spécifier le boost

- dans le paramétrage de l'index

```
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "boost": 2 
      },
      "content": {
        "type": "text"
      }
    }
  }
}

```

- dans la requête


```
GET /_search
{
  "query": {
    "term": {
      "user.id": {
        "value": "kimchy",
        "boost": 1.0
      }
    }
  }
}
```

## Stratégies de 'highlights'

L'option highlight permet de renvoyer dans les résultats les endroits exacts  où la correspondance a été trouvée

### Choix du type d'algorithme

- [Unified](https://www.elastic.co/guide/en/elasticsearch/reference/current/highlighting.html#unified-highlighter) (par défault)
- [Plain](https://www.elastic.co/guide/en/elasticsearch/reference/current/highlighting.html#plain-highlighter
- [Fast vector](https://www.elastic.co/guide/en/elasticsearch/reference/current/highlighting.html#unified-highlighter)


### Définition de l'offset

Pour extraire l'élément de correspondance, il faut définir le carcactère de début et de fin pour chaque mot du texte original.

- `fields` : au paramétrage initial en définissant les champs concernés par le highlight à choisir quand les champs contiennent beaucoup de texte (meilleures perfs) > unified
- `fvh` use term_vector (plus rapide pour les * et prefix) > fast_vector
- `plain` fonctionnement par défaut > plain

### Paramétrages des limites

- boundary_chars: choisir plusieurs caractères pour stopper le scan.Defaults to .,!? \t\n.
- boundary_max_scan: choisir le nombre de caractères max retournés. defaults to 20
- boundary_scanner: chars, sentence, ou word seulement si le type fvh or plain est choisi


### Paramétrage de l'encodage
le highlight doit il etre du texte simple ou de l'HTML defaul or HTML

### Choix des champs à afficher

fields: lister les champs à afficher * est accepté


### Paramétrage du fragment:
- type de fragment `fragmenter`: simple (meme taille) ou span (conserve le texte autour)
- marge du fragment `fragment_offset`: +x caractères avant et après le match (seulement avec fvh)
- taille du fragment `fragment_size`: taille du fragment en nb de caractère: default: 100
- type de tag de début et de fin: pre_tag, post_tags
- ordre: dans l'ordre des champs annoncé et ou l'ordre de pertinence?
- nombre de fragment: number_of_fragments 
- quand pas de match no_match_size 
- matched_fields combiner les match dans un seul champ

Le paramétrage se fait dans la réquête on peut définir les règles générales et réecrire les cas particulier localement

Pour plus de détails : https://www.elastic.co/guide/en/elasticsearch/reference/6.8/search-request-highlighting.html#fast-vector-highlighter

