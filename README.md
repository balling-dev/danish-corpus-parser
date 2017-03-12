Danish Corpus Parser
====================

This parser works on [corpus resources](http://korpus.dsl.dk/resources.html) (Korpus 90, 2000, and 2010) made available by the [Society for Danish Language and Literature (DSL)](www.dsl.dk).

Corpus resources are available for download in password-protected .zip files:

- [Korpus 90](http://korpus.dsl.dk/resources/corpora/KDK-1990.scrambled.zip) (244 MB, 22.1 million words)
- [korpus 2000](http://korpus.dsl.dk/resources/corpora/KDK-2000.scrambled.zip) (229 MB, 21.6 million words)
- [Korpus 2010](http://korpus.dsl.dk/resources/corpora/KDK-2010.scrambled.zip) (323 MB, 42.4 million words)

To obtain the password from DSL, please send a mail to korpus@dsl.dk with a brief description of the purpose(s) you intend to use the resources for. The conditions for using the corpora are [here](http://korpus.dsl.dk/conditions.html).

The parser is intended for use with DSL's corpus files, but **NOT** in any other way affiliated with, funded by, or associated with the [Society for Danish Language and Literature (DSL)](www.dsl.dk)

Requirements
------------

The contents of the password protected .zip files from DSL must be extracted to a folder (e.g. ```dsl```) with the following stucture:

- **Korpus 90**: text files (```0000.txt ``` etc) go into the folder ```dsl/1990/```
- **Korpus 2000**: text files (```0000.txt ``` etc) go into the folder ```dsl/2000/```
- **Korpus 2010**: folders (```01``` etc) go into the folder ```dsl/2010/```

The parser has the following requirements:

- Bourne Again Shell (**bash**): any recent version of bash is expected to work. Parser tested with bash version 4.3.46(1) (Ubuntu 16.04)
- GNU Awk (**gawk**): a recent version of gawk is required (awk and other implementations are not expected to work). Parser tested with gawk version 4.1.3 (Ubuntu 16.04)
- GNU Parallel (**parallel**): any recent version of parallel is expected to work. Parser tested with parallel version 20141022 (Ubuntu 16.04)
- Memory: Minimum 2GB available memory.
- Disk space: 10 GB (4 GB for DSL corpora, 4 GB for output files, 2 GB for temporary files)

Usage
-------

The parser counts the incidence of words, lemmas, part-of-speech (POS) tags, and [ePOS](http://korpus.dsl.dk/clarin/corpus-doc/pos-design.pdf) tags, as well as the incidence of unique combinations of the above.

To run all parser functions put the extracted DSL corpora in a dsl sub-folder and run:

```./dcp all -v```

Running this using 2 CPU cores (default) on a E5-2620v2 (2.1 GHz) system with 2 GB RAM and fast disks (NVME) takes approximately 31 minutes (in total, all three corpora), so grab a cup of coffee while it is running. Parsing only needs to be done once.

#### Output Files

The output from the parser are the following files (tab separated values):

- **words.tsv:** table of unique words (with id) and their incidence
- **lemmas.tsv:** table of unique lemmas (with id) and their incidence
- **pos.tsv:**  table of unique POS tags (with id) and their incidence
- **tags.tsv:**  table of unique ePOS tags (with id) and their incidence
- **wlpt.tsv:**  table of unique word, lemma, POS and ePOS combinations (with id)
  with words, lemmas, POS and ePOS being referred by their id. Incidence of 
  combination is also counted.
- **wpis.tsv:** table with word positions in sentences, words referred by their
  id in the wlpt.tsv table.
- **lbigrams.tsv:** table of letter bigram incidences.
- **sentences.txt:** assembled sentences from the corpora, one sentence per line
  without puntuations (commas, periods, hyphens) and all lower case.

The output file format and structure are made for loading the corpora into a relational database such as [PostgreSQL](https://www.postgresql.org/).

Command Line Options
--------------------

```
Danish Corpus Parser v0.0.1.0
Usage: dcp [ACTION] [OPTIONS]

where [ACTION] is:
  all,      make all tables
  base,     make base tables by assigning unique identifiers and summarising
            incidences of words, lemmas, POS and tags (from wlptc.tsv file)
  deriv,    make derived tables (letter bigrams, from words.tsv file)
  extract,  extract unique word, lemma, POS, tag combinations from corpus files
  sentence, output assembled sentences (one per line) (from corpus files).
  wpis,     make word position in sentence table (from corpus files and
            wlpt.tsv file)

Directory options:
  -i,  input (corpus) directory (default: ./dsl)
  -o,  output (tables) directory (.)
  -t,  temporary directory (default: /tmp)

Filename options:
  -A,  tags filename (default: tags.tsv)
  -B,  letter bigram filename (default: lbigrams.tsv)
  -I,  word positions in sentences filename (default: wpis.tsv)
  -L,  lemma filename (default: lemmas.tsv)
  -P,  part-of-speech (POS) filename (default: pos.tsv)
  -S,  sentences filename (default: sentences.txt)
  -W,  words filename (default: words.tsv)
  -X,  character words, lemmas, POS and tags filename (default: wlptc.tsv)
  -Y,  identifier words, lemmas, POS and tags filename (default: wlpt.tsv)

General options:
  -D,  debug
  -F,  force words, lemmas, POS and tags creation
  -h,  show this help text
  -v,  verbose output

Performance options:
  -C,  number of CPUs to use for prediction (default: 2)
  -U,  sort buffer size (default: 100M)
```


Notes on the ePOS structure
-------

Adapted from [Jørg Asmussen: Design of the ePOS tagger](http://korpus.dsl.dk/clarin/corpus-doc/pos-design.pdf).

The basic structure of an ePOS tag is:

```
CLASS:nominal:verbal:additional
```

where ```CLASS``` is a two-character POS classifier comprising a POS indicator (first character) and a sub-classifier. The first colon indicates the boundary between the ```CLASS``` part and the inflectional part of the tag. Here, ```nominal``` and ```verbal``` are strings of markers concerning nominal and verbal morphological information respectively. The ```additional``` string carries further markers relevant to adjectives, some adverbs, and pronouns. Marker strings have fixed lengths – ```nominal``` 4, ```verbal``` 2, and ```additional``` 4. Each marker in such a string is represented by one
character. The three groups of morphological markers are separated by colons (:).

The following structure shows the positions of the tags used in ePOS:

![](https://github.com/balling-cc/danish-corpus-parser/raw/master/inflectional-markers.png)

## Nominal markers

The string of ```nominal``` markers is of length four, it carries the following markers:

1. **Number** (NUM): *singular* (**s**) or *plural* (**p**)
2. **Definiteness** (DEF): *indefinite* (**i**) or *definite* (**d**)
3. **Case**   (CAS): *unmarked* (**u**), *genitive* (**g**), or *fossilized* (**f**), and – for personal
pronouns only – *nominative* (**n**) (*accusative* is identical with *unmarked* in
these cases and tagged with **u**)
4. **Gender** (GEN): *common* (**c**) or *neuter* (**n**)

## Verbal markers

The string of ```verbal ``` markers comprises two marker positions:

1. **Tense** (TMP): *present* (**s**), *past* (**t**)
2. **Voice** (VOC): *active* (**a**), *passive* (**p**)

## Additional markers

The string of ```additional``` markers constitute a heterogeneous group of the following four
markers:

1. **Degree** (DEG, adjectives and some adverbs): *positive* (**p**), *comparative* (**c**),
*superlative* (**s**), *absolute superlative* (**a**)
2. **Person** (PER, personal and possessive pronouns): *first* (**1**), *second* (**2**),
*third* (**3**)
3. **Reflexiveness** (RFL, personal and possessive pronouns): *yes* (**y**) or *no* (**n**)
4. **Possessor** (POS, possessive pronouns): *singular* (**s**) or *plural* (**p**)

## POS markers and subclassifiers

![](https://github.com/balling-cc/danish-corpus-parser/raw/master/pos-markers-and-subclassifiers.png)

License
-------

[Mozilla Public License Version 2.0](http://mozilla.org/MPL/2.0/)

Author Information
------------------

Kristoffer Winther Balling (kwballing@gmail.com)
Laura Winther Balling (laura.balling@gmail.com)
