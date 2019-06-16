
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
         Its leader is Annander Mianaai. 
         She's many-bodied and divided in at least 2 conflicting factions."
tokenised <- spacy_tokenize(text, what = "word", remove_punct = TRUE,
              remove_url = FALSE, remove_numbers = TRUE,
              remove_separators = TRUE, remove_symbols = FALSE, 
              padding = TRUE, multithread = TRUE, output = "list")
tokenised
$text1
 [1] "The"       "Radch"     "Empire"    "was"       "created"  
 [6] "thousands" "of"        "years"     "ago"       ""         
[11] "Its"       "leader"    "is"        "Annander"  "Mianaai"  
[16] ""          "She"       "'s"        "many"      ""         
[21] "bodied"    "and"       "divided"   "in"        "at"       
[26] "least"     ""          "factions"  ""     
```
 
---
# Preprocessing 
## Sentence segmentation 
```r
text <- "The Radch Empire was created thousands of years ago. 
         Its leader is Annander Mianaai 
         She's many-bodied and divided in at least 2 factions."
sentences <- spacy_tokenize(text, what = "sentence", remove_punct = TRUE,
               remove_url = FALSE, remove_numbers = TRUE,
               remove_separators = TRUE, remove_symbols = FALSE, 
               padding = TRUE, multithread = TRUE, output = "list")
sentences
$text1
[1] "The Radch Empire was created thousands of years ago." 
[2] "Its leader is Annander Mianaai"                       
[3] "She's many-bodied and divided in at least 2 factions."  
``` 
---
# Preprocessing 
## Lemmatisation
```r
text <- "The Radch Empire was created thousands of years ago. 
         Its leader is Annander Mianaai. 
         She's many-bodied and divided in at least 2 factions."
lemmatised <- spacy_parse(text, pos = FALSE, tag = FALSE, lemma = TRUE,
            entity = FALSE, dependency = FALSE, nounphrase = FALSE,
            multithread = TRUE)
lemmatised %>% filter(token != lemma)
   doc_id sentence_id token_id     token    lemma
1   text1           1        1       The      the
2   text1           1        4       was       be
3   text1           1        5   created   create
4   text1           1        6 thousands thousand
5   text1           1        8     years     year
6   text1           2        1       Its   -PRON-
7   text1           2        3        is       be
8   text1           3        1       She   -PRON-
9   text1           3        2        's       be
10  text1           3        7   divided   divide
11  text1           3       12  factions  faction
```

---
# Preprocessing
## Stopwords
```r
text <- "The Radch Empire was created thousands of years ago. 
         Its leader is Annander Mianaai. 
         She's many-bodied and divided in at least 2 factions."
         
unnest_tokens(lemmatised, word, token, to_lower = TRUE) %>%
  anti_join(stop_words) %>% `[[`('word')
Joining, by = "word"
 [1] "radch"     "empire"    "created"   "thousands" "ago"      
 [6] "leader"    "annander"  "mianaai"   "bodied"    "divided"  
[11] "2"         "factions"

lemmatised <- spacy_parse(text, pos = FALSE, tag = FALSE, lemma = TRUE,
              entity = FALSE, dependency = FALSE, nounphrase = FALSE,
              multithread = TRUE, additional_attributes = 'is_stop')
lemmatised %>% filter(is_stop != TRUE) %>% `[[`('token')
 [1] "Radch"     "Empire"    "created"   "thousands" "years"    
 [6] "ago"       "."         "leader"    "Annander"  "Mianaai"  
[11] "."         "-"         "bodied"    "divided"   "2"        
[16] "factions"  "."                     
``` 
---
# Linguistic features
## Parts of speech 
```r
text <- "The Radch Empire was created thousands of years ago. 
         Its leader is Annander Mianaai. 
         She's many-bodied and divided in at least 2 factions."
pos <- spacy_parse(text, pos = TRUE, tag = TRUE, lemma = FALSE,
                   entity = FALSE, dependency = FALSE, nounphrase = FALSE,
                   multithread = TRUE)
pos %>% filter(pos == 'ADJ' | pos == 'VERB')
  doc_id sentence_id token_id   token  pos tag
1  text1           1        4     was VERB VBD
2  text1           1        5 created VERB VBN
3  text1           2        3      is VERB VBZ
4  text1           3        2      's VERB VBZ
5  text1           3        3    many  ADJ  JJ
6  text1           3        5  bodied  ADJ  JJ
7  text1           3        7 divided VERB VBN
8  text1           3       10   least  ADJ JJS
```

See [annotation specifications](https://spacy.io/api/annotation) for the full tag list. 

---
# Linguistic features
## Dependencies

```r
text <- "The Radch Empire was created thousands of years ago. 
         Its leader is Annander Mianaai. 
         She's many-bodied and divided in at least 2 factions."
dep <- spacy_parse(text, pos = FALSE, tag = FALSE, lemma = FALSE,
                   entity = FALSE, dependency = TRUE, nounphrase = FALSE,
                   multithread = TRUE)
dep %>% filter(sentence_id == 2)
  doc_id sentence_id token_id    token head_token_id  dep_rel
1  text1           2        1      Its             2     poss
2  text1           2        2   leader             3    nsubj
3  text1           2        3       is             3     ROOT
4  text1           2        4 Annander             5 compound
5  text1           2        5  Mianaai             3     attr
6  text1           2        6        .             3    punct
```

---
@snap[midpoint span-100]
![](dep.png)
@snapend

---
# Noun phrases 
```r
text <- "The Radch Empire was created thousands of years ago. 
         Its leader is Annander Mianaai. 
         She's many-bodied and divided in at least 2 factions."
nounphrases <- spacy_parse(text, pos = FALSE, tag = FALSE, lemma = FALSE,
                   entity = FALSE, dependency = FALSE, nounphrase = TRUE,
                   multithread = TRUE)
nounphrase_extract(nounphrases, concatenator = "_")
  doc_id sentence_id          nounphrase
1  text1           1    The_Radch_Empire
2  text1           1               years
3  text1           2          Its_leader
4  text1           2    Annander_Mianaai
5  text1           3                 She
6  text1           3 at_least_2_factions
```


---
# Entities
```r
text <- "The Radch Empire was created thousands of years ago. 
         Its leader is Annander Mianaai. 
         She's many-bodied and divided in at least 2 factions."
entities <- spacy_parse(text, pos = FALSE, tag = FALSE, lemma = FALSE,
                           entity = TRUE, dependency = FALSE, nounphrase = FALSE,
                           multithread = TRUE)
entity_extract(entities, type = 'all', concatenator = "_")
  doc_id sentence_id                 entity entity_type
1  text1           1       The_Radch_Empire         GPE
2  text1           1 thousands_of_years_ago        DATE
3  text1           2       Annander_Mianaai      PERSON
4  text1           3             at_least_2    CARDINAL
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
- Open information extraction 

## Stanford [coreNLP](https://stanfordnlp.github.io/CoreNLP/)

---
# Open information extraction

```r
downloadCoreNLP()
initCoreNLP(type='english_all')
text <- "The Radch Empire was created thousands of years ago. 
         Its leader is Annander Mianaai. 
         She's many-bodied and divided in at least 2 factions."
annObj <- annotateString(text)
getOpenIE(annObj) %>% select(subject, relation, object)
       subject    relation                 object
1 Radch Empire was created     thousands of years
2 Radch Empire was created thousands of years ago
3 Radch Empire was created              thousands
4   Its leader          is       Annander Mianaai
5          She  divided in    at least 2 factions
6          She         has            many-bodied
```
---
# Coreferences and sentiment 
```r
getCoreference(annObj)
  corefId sentence start end head startIndex endIndex
1       1        1     1   4    3          1        3
2       1        2     1   2    1         11       11
3       2        2     4   6    5         14       15
4       2        2     1   3    2         11       12
5       2        3     1   2    1         17       17

getSentiment(annObj)
  id sentimentValue sentiment
1  1              2   Neutral
2  2              1  Negative
3  3              2   Neutral
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

