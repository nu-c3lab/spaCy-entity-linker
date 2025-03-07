# Spacy Canonicalizer

## Introduction

Spacy Canonicalizer is a pipeline for spaCy that performs Linked Entity Extraction with Wikidata on a given Document.
The Entity Linking System operates by matching potential candidates from each sentence
(subject, object, prepositional phrase, compounds, etc.) to aliases from Wikidata. The package allows to easily find the
category behind each entity (e.g. "banana" is type "food" OR "Microsoft" is type "company"). It can is therefore useful
for information extraction tasks and labeling tasks.

The package was written before a working Linked Entity Solution existed inside spaCy. In comparison to spaCy's linked
entity system, it has the following advantages:

- no extensive training required (entity-matching via database)
- knowledge base can be dynamically updated without retraining
- entity categories can be easily resolved
- grouping entities by category

It also comes along with a number of disadvantages:

- it is slower than the spaCy implementation due to the use of a database for finding entities
- no context sensitivity due to the implementation of the "max-prior method" for entitiy disambiguation (an improved
  method for this is in progress)

## Use

```python
import spacy  # version 3.0.6'

# initialize language model
nlp = spacy.load("en_core_web_md")

# add pipeline (declared through entry_points in setup.py)
nlp.add_pipe("entityLinker", last=True)

doc = nlp("I watched the Pirates of the Caribbean last silvester")

# returns all entities in the whole document
all_linked_entities = doc._.linkedEntities
# iterates over sentences and prints linked entities
for sent in doc.sents:
    sent._.linkedEntities.pretty_print()

# OUTPUT:
# https://www.wikidata.org/wiki/Q194318     Pirates of the Caribbean        Series of fantasy adventure films                                                                   
# https://www.wikidata.org/wiki/Q12525597   Silvester                       the day celebrated on 31 December (Roman Catholic Church) or 2 January (Eastern Orthodox Churches)  

```

### EntityCollection

contains an array of entity elements. It can be accessed like an array but also implements the following helper
functions:

- <code>pretty_print()</code> prints out information about all contained entities
- <code>print_super_classes()</code> groups and prints all entites by their super class

```python
doc = nlp("Elon Musk was born in South Africa. Bill Gates and Steve Jobs come from the United States")
doc._.linkedEntities.print_super_entities()
# OUTPUT:
# human (3) : Elon Musk,Bill Gates,Steve Jobs
# country (2) : South Africa,United States of America
# sovereign state (2) : South Africa,United States of America
# federal state (1) : United States of America
# constitutional republic (1) : United States of America
# democratic republic (1) : United States of America
```

### EntityElement

each linked Entity is an object of type <code>EntityElement</code>. Each entity contains the methods

- <code>get_description()</code> returns description from Wikidata
- <code>get_id()</code> returns Wikidata ID
- <code>get_label()</code> returns Wikidata label
- <code>get_span()</code> returns the span from the spacy document that contains the linked entity
- <code>get_url()</code> returns the url to the corresponding Wikidata item
- <code>pretty_print()</code> prints out information about the entity element
- <code>get_sub_entities(limit=10)</code> returns EntityCollection of all entities that derive from the current
  entityElement (e.g. fruit -> apple, banana, etc.)
- <code>get_super_entities(limit=10)</code> returns EntityCollection of all entities that the current entityElement
  derives from (e.g. New England Patriots -> Football Team))

## Example

In the following example we will use SpacyEntityLinker to find find the mentioned Football Team in our text and explore
other football teams of the same type

```python

doc = nlp("I follow the New England Patriots")

patriots_entity = doc._.linkedEntities[0]
patriots_entity.pretty_print()
# OUTPUT:
# https://www.wikidata.org/wiki/Q193390     
# New England Patriots            
# National Football League franchise in Foxborough, Massachusetts                    

football_team_entity = patriots_entity.get_super_entities()[0]
football_team_entity.pretty_print()
# OUTPUT:
# https://www.wikidata.org/wiki/Q17156793 
# American football team          
# organization, in which a group of players are organized to compete as a team in American football   


for child in football_team_entity.get_sub_entities(limit=32):
    print(child)
    # OUTPUT:
    # New Orleans Saints
    # New York Giants
    # Pittsburgh Steelers
    # New England Patriots
    # Indianapolis Colts
    # Miami Seahawks
    # Dallas Cowboys
    # Chicago Bears
    # Washington Redskins
    # Green Bay Packers
    # ...
```

### Entity Linking Policy

Currently the only method for choosing an entity given different possible matches (e.g. Paris - city vs Paris -
firstname) is max-prior. This method achieves around 70% accuracy on predicting the correct entities behind link
descriptions on wikipedia.

## Note

The Entity Linker at the current state is still experimental and should not be used in production mode.

## Performance

The current implementation supports only Sqlite. This is advantageous for development because it does not requirement
any special setup and configuration. However, for more performance critical usecases, a different database with
in-memory access (e.g. Redis) should be used. This may be implemented in the future.

## Installation

To install the package run: <code>pip install spacy-canonicalizer</code>

## Versions:

- <code>spacy_entity_linker>=0.0</code> (requires <code>spacy>=2.2,<3.0</code>)
- <code>spacy_entity_linker>=1.0</code> (requires <code>spacy>=3.0</code>)

## TODO

- [ ] implement Entity Classifier based on sentence embeddings for improved accuracy
- [ ] implement get_picture_urls() on EntityElement
- [ ] retrieve statements for each EntityElement (inlinks + outlinks)
