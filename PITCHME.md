
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

- @fa[robot](Chatbots)
- Knowledge graphs 
- Machine translation
- Automatic summarisation 
- Speech-to-text
- OCR  


### We will focus on text processing 

---

# Challenges of text processing 

- Large vocabulary
- Vast amounts of data
- Inconsistencies 
 
---
# spaCy

- Fast: Cython
- Wide variety of features 
- Several pretrained Enlish models
- Good docs

---
# spacyr 

- Runs spacy in the background
- Will install miniconda, python and spacy 

---
# Key components of spacyr


- Preprocessing
- Linguistic features 
- Language models
- Word embeddings 


---
# Preprocessing 
## Tokenisation
```r
text <- "The Radch Empire was created thousands of years ago. 
         It's leader is Annander Mianaai. 
         She is many-bodied and divided in at least 2 factions."
tokenised <- spacy_tokenize(text, what = "word", remove_punct = TRUE,
              remove_url = FALSE, remove_numbers = TRUE,
              remove_separators = TRUE, remove_symbols = FALSE, padding = TRUE,
              multithread = TRUE, output = "list")
tokenised
$text1
 [1] "The"       "Radch"     "Empire"    "was"       "created"   "thousands"
 [7] "of"        "years"     "ago"       ""          "It"        "'s"       
[13] "leader"    "is"        "Annander"  "Mianaai"   ""          "She"      
[19] "is"        "many"      ""          "bodied"    "and"       "divided"  
[25] "in"        "at"        "least"     ""          "factions"  ""       
```
 
---
# Preprocessing 
## Sentence segmentation 
```r
text <- "Explosion-AI made spaCy for natural language processing 
         (2+ years ago) It's faster than other NLP libraries."
sentences <- spacy_tokenize(text, what = "sentence", remove_punct = TRUE,
               remove_url = FALSE, remove_numbers = TRUE,
               remove_separators = TRUE, remove_symbols = FALSE, 
               padding = TRUE, multithread = TRUE, output = "list")
sentences
#$text1
#[1] "Explosion-AI made spaCy for natural language processing (2+ years ago)"
#[2] "It's faster than other NLP libraries."    
``` 
---
# Preprocessing 
## Lemmatisation
```r
text <- "Explosion-AI made spaCy for natural language processing 
        (2+ years ago). It's faster than other NLP libraries."
lemmatised <- spacy_parse(text, pos = FALSE, tag = FALSE, lemma = TRUE,
            entity = FALSE, dependency = FALSE, nounphrase = FALSE,
            multithread = TRUE)
lemmatised %>% filter(token != lemma)
#  doc_id sentence_id token_id     token     lemma
#1  text1           1        1 Explosion explosion
#2  text1           1        4      made      make
#3  text1           1        5     spaCy     spacy
#4  text1           1       13     years      year
#5  text1           2        1        It    -PRON-
#6  text1           2        2        's        be
#7  text1           2        3    faster      fast
#8  text1           2        7 libraries   library
```

---
# Preprocessing
## Stopwords
```r
text <- "Explosion-AI made spaCy for natural language processing 
        (2+ years ago). It's faster than other NLP libraries."
         
unnest_tokens(lemmatised, word, token, to_lower = TRUE) %>%
  anti_join(stop_words) %>% `[[`('word')
#Joining, by = "word"
# [1] "explosion"  "ai"         "spacy"      "natural"    "language"  
# [6] "processing" "2"          "ago"        "faster"     "nlp"       
#[11] "libraries"

lemmatised <- spacy_parse(text, pos = FALSE, tag = FALSE, lemma = TRUE,
              entity = FALSE, dependency = FALSE, nounphrase = FALSE,
              multithread = TRUE, additional_attributes = 'is_stop')
lemmatised %>% filter(is_stop != TRUE) %>% `[[`('token')
# [1] "Explosion"  "-"          "AI"         "spaCy"      "natural"   
# [6] "language"   "processing" "("          "2"          "+"         
#[11] "years"      "ago"        ")"          "."          "faster"    
#[16] "NLP"        "libraries"  "."                    
``` 
---
# Linguistic features
## Parts of speech 
```r
text <- "Explosion-AI made spaCy for natural language processing 
        (2+ years ago). It's faster than other NLP libraries."
pos <- spacy_parse(text, pos = TRUE, tag = TRUE, lemma = FALSE,
                   entity = FALSE, dependency = FALSE, nounphrase = FALSE,
                   multithread = TRUE)
pos %>% filter(pos == 'ADJ' | pos == 'VERB')
#  doc_id sentence_id token_id   token  pos tag
#1  text1           1        4    made VERB VBD
#2  text1           1        5   spaCy  ADJ  JJ
#3  text1           1        7 natural  ADJ  JJ
#4  text1           2        2      's VERB VBZ
#5  text1           2        3  faster  ADJ JJR
#6  text1           2        5   other  ADJ  JJ
```

See [annotation specifications](https://spacy.io/api/annotation) for the full tag list. 

---
# Linguistic features
## Dependencies

```r
text <- "Explosion-AI made spaCy for natural language processing 
        (2+ years ago). It's faster than other NLP libraries."
dep <- spacy_parse(text, pos = FALSE, tag = FALSE, lemma = FALSE,
                   entity = FALSE, dependency = TRUE, nounphrase = FALSE,
                   multithread = TRUE)
dep %>% filter(sentence_id == 2)
#  doc_id sentence_id token_id     token head_token_id  dep_rel
#1  text1           2        1        It             2    nsubj
#2  text1           2        2        's             2     ROOT
#3  text1           2        3    faster             2    acomp
#4  text1           2        4      than             3     prep
#5  text1           2        5     other             7     amod
#6  text1           2        6       NLP             7 compound
#7  text1           2        7 libraries             4     pobj
#8  text1           2        8         .             2    punct
```

---
@snap[midpoint span-100]
![](dep.png)
@snapend

---
# Noun phrases 
```r
text <- "Explosion-AI made SPACY for natural language processing 
         (2+ years ago). It's faster than other NLP libraries."
nounphrases <- spacy_parse(text, pos = FALSE, tag = FALSE, lemma = FALSE,
                   entity = FALSE, dependency = FALSE, nounphrase = TRUE,
                   multithread = TRUE)
nounphrase_extract(nounphrases, concatenator = "_")
#  doc_id sentence_id                  nounphrase
#1  text1           1                Explosion-AI
#2  text1           1                       SPACY
#3  text1           1 natural_language_processing
#4  text1           2                          It
#5  text1           2         other_NLP_libraries
```


---
# Named entities
```r
text <- "Explosion-AI made SPACY for natural language processing 
         (2+ years ago). It's faster than other NLP libraries."
entities <- spacy_parse(text, pos = FALSE, tag = FALSE, lemma = FALSE,
                           entity = TRUE, dependency = FALSE, nounphrase = FALSE,
                           multithread = TRUE)
entity_extract(entities, type = 'all', concatenator = "_")
#  doc_id sentence_id         entity entity_type
#1  text1           1 Explosion_-_AI         ORG
#2  text1           1          SPACY         ORG
#3  text1           1  2_+_years_ago        DATE
#4  text1           2            NLP         ORG
```
--- 
# Word embeddings
[Global Vectors for word representation](https://nlp.stanford.edu/projects/glove/)
```r
text <- "apple orange chair rumpelstiltskin"
vectors <- spacy_parse(text, pos = FALSE, tag = FALSE, lemma = FALSE,
                       entity = FALSE, dependency = FALSE, nounphrase = FALSE,
                       multithread = TRUE, 
                       additional_attributes = c('has_vector', 'vector_norm', 'vector'))
vectors[1:2,] %>% select(token, has_vector, vector_norm) 
#            token has_vector vector_norm
#1           apple       TRUE    7.134685
#2          orange       TRUE    6.542022
```
---
# Semantic similarity 

Cosine similarity scores between:

- apple and orange: 0.5618917
- apple and chair: 0.1714211
- apple and rumpelstiltskin: -0.1144477


--- 
@transition[none]
# Additional attributes  

Check spaCy doc for [full list of token attributes](https://spacy.io/api/token). 

- lower_, is_lower,
- shape_,
- is_alpha, is_digit, is_ascii, like_num, is_currency, 
- is_oov,
- is_space, is_bracket, is_quote, etc.


```r
spacy_finalize()
```


--- 
# Other text processing tasks

- Sentiment scores 
- Coreference resolution
- Entity linking 
- Open information extraction 

## Stanford [coreNLP](https://stanfordnlp.github.io/CoreNLP/)

---
# Open information extraction

```r
downloadCoreNLP()
initCoreNLP(type='english_all')
text <- "Explosion-AI made SPACY for natural language processing 
         (2+ years ago). It's faster than other NLP libraries."
annObj <- annotateString(text)
getOpenIE(annObj) %>% select(subject, relation, object)
#       subject       relation                                object
#1 Explosion-AI           made                                 SPACY
#2 Explosion-AI           made         SPACY for language processing
#3 Explosion-AI           made SPACY for natural language processing
#4           It 's faster than                   other NLP libraries
#5           It 's faster than                         NLP libraries
#6           It            has                                faster
```

--- 
# One step further


- Pretrain with domain-specific text
- Add custom entity types 
- Make domain-specific word embeddings 


---
@snap[west span-100]
# Thank you! 
@snapend

