# SoMaJo #

  * [Introduction](#introduction)
  * [Installation](#installation)
  * [Usage](#usage)
      * [Using the somajo-tokenizer executable](#using-the-somajo-tokenizer-executable)
      * [Using the module](#using-the-module)
  * [Evaluation](#evaluation)
  * [References](#references)


## Introduction ##

SoMaJo is a state-of-the-art tokenizer and sentence splitter for
German web and social media texts that won the [EmpiriST 2015 shared
task](https://sites.google.com/site/empirist2015/) on automatic
linguistic annotation of computer-mediated communication / social
media. As such, it is particularly well-suited to perform tokenization
on all kinds of written discourse, for example chats, forums, wiki
talk pages, tweets, blog comments, social networks, SMS and WhatsApp
dialogues.

In addition to tokenizing the input text, SoMaJo can also output token
class information for each token, i.e. if it is a number, an emoticon,
an abbreviation, etc.:

    echo 'Wow, superTool!;)' | somajo-tokenizer -c -t -
    Wow	regular
    ,	symbol
    super	regular
    Tool	regular
    !	symbol
    ;)	emoticon

SoMaJo can also output additional information for each token that can
help to reconstruct the original untokenized text (to a certain
extent):

    echo 'der beste Betreuer? - >ProfSmith! : )' | somajo-tokenizer -c -e -
    der	
    beste	
    Betreuer	SpaceAfter=No
    ?	
    ->	SpaceAfter=No, OriginalSpelling="- >"
    Prof	SpaceAfter=No
    Smith	SpaceAfter=No
    !	
    :)	OriginalSpelling=": )"

The `-t` and `-e` options can also be used in combination, of course.

SoMaJo can split the input text into sentences using the
`--split_sentences` option.

SoMaJo has full XML support, i.e. it can perform sensible tokenization
and sentence splitting on well-formed XML files using the `--xml` and
`--tag` options.

The system is described in greater detail in [Proisl and Uhrig
(2016)](http://aclweb.org/anthology/W16-2607).

For part-of-speech tagging, we recommend
[SoMeWeTa](https://github.com/tsproisl/SoMeWeTa), a part-of-speech
tagger with state-of-the-art performance on German web and social
media texts:

    somajo-tokenizer --split_sentences <file> | somewe-tagger --tag <model> -


## Installation ##

SoMaJo can be easily installed using pip:

    pip install SoMaJo

Alternatively, you can download and decompress the [latest
release](https://github.com/tsproisl/SoMaJo/releases/latest) or clone
the git repository:

    git clone https://github.com/tsproisl/SoMaJo.git

In the new directory, run the following command:

    python3 setup.py install


## Usage ##

### Using the somajo-tokenizer executable

You can use the tokenizer as a standalone program from the command
line. General usage information is available via the `-h` option:

    somajo-tokenizer -h

To tokenize a text file according to the guidelines of the EmpiriST
2015 shared task, just call the tokenizer like this:

    somajo-tokenizer -c <file>

If you do not want to split camel-cased tokens, simply drop the `-c`
option:

    somajo-tokenizer <file>

The tokenizer can also output token class information for each token,
i.e. if it is a number, an emoticon, an abbreviation, etc.:

    somajo-tokenizer -t <file>

If you want to be able to reconstruct the untokenized input to a
certain extent, SoMaJo can also provide you with additional details
for each token, i.e. if the token was followed by whitespace or if it
contained internal whitespace (according to the tokenization
guidelines, things like “: )” get normalized to “:)”):

    somajo-tokenizer -e <file>

SoMaJo assumes that paragraphs are delimited by empty lines in the
input file. If your input file uses single newlines instead, you have
to tell that to the tokenizer via the `-s` or `--paragraph_separator`
option:

    somajo-tokenizer --paragraph_separator single_newlines <file>

To speed up tokenization, you can specify the number of worker
processes used via the `--processes` option:

    somajo-tokenizer --processes <number> <file>

SoMaJo can also split the input paragraphs into sentences:

    somajo-tokenizer --split_sentences <file>

SoMaJo can also process XML files. Use the `-x` or `--xml` option to
tell the tokenizer that your input is an XML file:

    somajo-tokenizer --xml <xml-file>

If you also want to do sentence splitting, you can use (multiple
instances of) the `--tag` option to specify XML tags that are always
sentence breaks, i.e. that can never occur in the middle of a
sentence. Per default, the sentence splitter uses the following list
of tags: title, h1, h2, h3, h4, h5, h6, p, br, div, ol, ul, dl and
table.

    somajo-tokenizer --xml --tag h1 --tag p --tag div <xml-file>


### Using the module ###

You can easily incorporate the tokenizer into your own Python
projects. All you have to do is import `somajo.Tokenizer`, create a
`Tokenizer` object and call its `tokenize` method:

    from somajo import Tokenizer

    # note that paragraphs are allowed to contain newlines
    paragraph = "der beste Betreuer?\n-- ProfSmith! : )"

    tokenizer = Tokenizer(split_camel_case=True, token_classes=False, extra_info=False)
    tokens = tokenizer.tokenize(paragraph)

    print("\n".join(tokens), "\n")

Sentence splitting operates on tokenized text:

    from somajo import SentenceSplitter
    
    # set is_tuple=True if token_classes=True or extra_info=True
    sentence_splitter = SentenceSplitter(is_tuple=False)
    sentences = sentence_splitter.split(tokens)
    
    for sentence in sentences:
        print("\n".join(sentence), "\n")

For processing XML data, use the `tokenize_xml` and `split_xml` methods:

    # you can read from an open file object
    tokens = tokenizer.tokenize_xml(file_object)
	# or you can pass a string with XML data
	tokens = tokenizer.tokenize_xml(xml_string, is_file=False)
	
	print("\n".join(tokens), "\n")
	
	eos_tags = set(["title", "h1", "p"])
	sentences = sentence_splitter.split_xml(tokens, eos_tags)
	
    for sentence in sentences:
        print("\n".join(sentence), "\n")
	

## Evaluation ##

SoMaJo was the system with the highest average F₁ score in the
EmpiriST 2015 shared task. The performance of the current version on
the two test sets is summarized in the following table (Training and
test sets are available from the [official
website](https://sites.google.com/site/empirist2015/home/gold)):

| Corpus | Precision | Recall | F₁    |
|--------|-----------|--------|-------|
| CMC    | 99.62     | 99.56  | 99.59 |
| Web    | 99.83     | 99.92  | 99.87 |


## References ##

  * Proisl, Thomas, Peter Uhrig (2016): “SoMaJo: State-of-the-art
    tokenization for German web and social media texts.” In:
    Proceedings of the 10th Web as Corpus Workshop (WAC-X) and the
    EmpiriST Shared Task. Berlin: Association for Computational
    Linguistics (ACL), 57–62.
    [PDF](http://aclweb.org/anthology/W16-2607).
