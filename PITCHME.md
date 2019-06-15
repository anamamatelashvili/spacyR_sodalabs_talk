
# Natural Language Processing with spaCy in R
@snap[east span-100]
## Ana Mamatelashvili 
@snapend

@snap[south-east span-30]
18 June, 2019
@snapend
---
@snap[midpoint span-50]
# A bit about me 
@snapend
---

# Why do NLP?
@ul
- @fa[robot](Chatbots)
- speech to text
- translation
- summarisation 
@ulend

will focus on text processing 

---

# Challenges of text processing 

@ul
- 
@ulend

 
---
# spaCy

multithreading 

---
# spacyr 

spacyr
will install miniconda and python and spacy and run them in the background 

---
# Key components of spacyr

@ul
- preprocessing
- linguistic features 
- language models
- word embeddings 
@ulend

---
# Preprocessing 
## Tokenisation
```{r}
text <- "Ines Montani made spaCy for natural language processing 
         (2+ years-ago). It's a nice library."
tokenised <- spacy_tokenize(text, what = "word", remove_punct = TRUE,
               remove_url = FALSE, remove_numbers = TRUE,
               remove_separators = TRUE, remove_symbols = FALSE, 
               padding = TRUE, multithread = TRUE, output = "list")
tokenised
#$text1
# [1] "Ines"       "Montani"    "made"       "spaCy"      "for"       
# [6] "natural"    "language"   "processing" ""           ""          
#[11] "+"          "years"      ""           "ago"        ""          
#[16] ""           ""           "It"         "'s"         "a"         
#[21] "nice"       "library"    ""    
```
 
---
# Preprocessing 
## Lemmatisation

---
# preprocessing

stopwords
 
---
# linguistic features 

pos and tag 


---
# linguistic features 

dep


---
# Entities 

named

---
# Entities

extended

--- 
# word embeddings

--- 
# Semantic similarity 

--- 
# Other text processing 

coreNLP

entity linking
coreference resolution
open information extraction

---
# entity linking

---
# coreference resolution

---
# open information extraction

--- 
# summarise 
maybe have a picture of grakn result???

---

# Thank you! 
There are many natural language processing libraries out there and each has it's own advantages. One of Ana's favourites is spaCy because of it's speed and user-friendliness. In this talk she will discuss how to leverage spaCy's powerful features with the spacyr package and combine them with other packages such as tidytext, quanteda and coreNLP to do natural language processing in R. She will discuss use cases for tokenisation, lemmatisation, extraction of linguistic features and named entities as well as Coreference Resolution.
