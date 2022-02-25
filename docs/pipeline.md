# Pipeline 

## INIT

from rules.csv populate DB
- import_rules
- import_reference

## CREATE FASTAPI

generate_api:
- generate_models

from rules create model with Jinja2 templating

- generate_routers
use a standard template into data/template/routers.tpl
https://fastapi-crudrouter.awtkns.com/
import fastApiCrud router but no way to use a Mongo connector
except maybe using beanie
Open endpoints but not connected to DB

So rewrite the routers for datasets organizations but can be generated

- [ ] open search and filter methods in datasets
- [ ] test every endpoints
- [ ] recreate index


## POPULATE DB

import organizations_fr
import datasets_fr why are missing 2 datasets?


## CREATE INDEX

- use rules to create index
- use rules to create filters
- remove log level set to debug:
```
curl -X PUT "localhost:9200/_cluster/settings?pretty" -H 'Content-Type: application/json' -d'
{
  "persistent": {
    "logger.org.elasticsearch.discovery": "DEBUG"
  }
}
'
```

## CREATE COMMENTS


------
NOTES

## init_db: from csv to mongodb table

- create rules DB
- 
Option: see Beanie as ODM

- create a document model using rules


## create_model: from mongodb table to pydantic

1. Create json-schema from example

Option 1: json schema example see Pydantic Doc Code Generation:


  - [x] add example value in rules.csv => init_db
  - [x] add validator using rules => enum OK
  - [x] declare external ref as ref
  - [x] add external_validor and send them to create_model
```
validators = {
    'username_validator':
    validator('username')(username_alphanumeric)
}

UserModel = create_model(
    'UserModel',
    username=(str, ...),
    __validators__=validators
    __config__ = Config
)
```

BUT: no way to generate from model only import

Option 2: create json-schema with code using rules


2. Create pymodel.py

Option 1. use datamodel-code-generator
alternative in CLI datamodel code generator takes a jsonschema and create model.py
> https://github.com/koxudaxi/datamodel-code-generator

import models into back.apps
Not working 
Option2. convert jsonschemas to openapi

## populate initial data using rules table to control insertion


## Create API CRUD endpoint from Models




## Create index

-
- See esengine is an ODM (Object Document Mapper) it maps Python classes in to Elasticsearch index/doc_type and object instances() in to Elasticsearch documents.


