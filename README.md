# bangla-text-processing-kit

A python tool kit for processing Bangla texts.

- [bangla-text-processing-kit](#bangla-text-processing-kit)
  - [How to use](#how-to-use)
    - [Installing](#installing)
    - [Checking text](#checking-text)
    - [Transforming text](#transforming-text)
      - [Normalizer](#normalizer)
      - [Character Normalization](#character-normalization)
      - [Punctuation space normalization](#punctuation-space-normalization)
      - [Zero width characters normalization](#zero-width-characters-normalization)
      - [Halant (হসন্ত) normalization](#halant-হসন্ত-normalization)
      - [Kar ambiguity](#kar-ambiguity)
      - [Clean text](#clean-text)
      - [Clean punctuations](#clean-punctuations)
      - [Clean digits](#clean-digits)
      - [Multiple spaces](#multiple-spaces)
      - [URLs](#urls)
      - [Emojis](#emojis)
      - [HTML tags](#html-tags)
      - [Multiple punctuations](#multiple-punctuations)
      - [Special characters](#special-characters)
      - [Non Bangla characters](#non-bangla-characters)
    - [Lemmatization](#lemmatization)
      - [Lemmatize text](#lemmatize-text)
      - [Lemmatize word](#lemmatize-word)
    - [Tokenization](#tokenization)
      - [Word tokenization](#word-tokenization)
      - [Word and Punctuation tokenization](#word-and-punctuation-tokenization)
      - [Sentence tokenization](#sentence-tokenization)
    - [Named Entity Recognition (NER)](#named-entity-recognition-ner)
    - [Parts of Speech (PoS) tagging](#parts-of-speech-pos-tagging)
    - [Shallow Parsing (Constituency Parsing)](#shallow-parsing-constituency-parsing)

## How to use

### Installing

Installing via pip:

```bash
pip install git+https://github.com/giga-tech/bangla-text-processing-kit.git
```

### Checking text

- `bkit.utils.is_bangla(text) -> bool`: Checks if text contains only Bangla characters, digits, spaces, punctuations and some symbols. Returns true if so, else return false.
- `bkit.utils.is_digit(text) -> bool`: Checks if text contains only **Bangla digit** characters. Returns true if so, else return false.
- `bkit.utils.contains_digit(text, check_english_digits) -> bool`: Checks if text contains **any digits**. By default checks only Bangla digits. Returns true if so, else return false.
- `bkit.utils.contains_bangla(text) -> bool`: Checks if text contains **any Bangla character**. Returns true if so, else return false.

### Transforming text

Text transformation includes the normalization and cleaning procedures. To transform text, use the `bkit.transform` module. Supported functionalities are:

#### Normalizer

This module normalize Bangla text using the following steps:

<!-- no toc -->
1. [Character normalization](#character-normalization)
2. [Zero width character normalization](#zero-width-characters-normalization)
3. [Halant normalization](#halant-হসন্ত-normalization)
4. [Vowel-kar normalization](#kar-ambiguity)
5. [Punctuation space normalization](#punctuation-space-normalization)

```python
import bkit

text = 'অাামাব় । '
print(list(text))
# >>> ['অ', 'া', 'া', 'ম', 'া', 'ব', '়', ' ', '।', ' ']

normalizer = bkit.transform.Normalizer(
    normalize_characters=True,
    normalize_zw_characters=True,
    normalize_halant=True,
    normalize_vowel_kar=True,
    normalize_punctuation_spaces=True
)

clean_text = normalizer(text)
print(clean_text, list(clean_text))
# >>> আমার। ['আ', 'ম', 'া', 'র', '।']
```

#### Character Normalization

This module performs character normalization in Bangla text. It performs nukta normalization, Assamese normalization, Kar normalization, legacy character normalization and Punctuation normalization sequentially.

```python
import bkit

text = 'আমাব়'
print(list(text))
# >>> ['আ', 'ম', 'া', 'ব', '়']

text = bkit.transform.normalize_characters(text)

print(list(text))
# >>> ['আ', 'ম', 'া', 'র']
```

#### Punctuation space normalization

Normalizes punctuation spaces i.e. adds necessary spaces before or after specific punctuations, also removes if necessary.

```python
import bkit

text = 'রহিম(২৩)এ কথা বলেন   ।তিনি (    রহিম ) আরও জানান, ১,২৪,৩৫,৬৫৪.৩২৩ কোটি টাকা ব্যায়ে...'

clean_text = bkit.transform.normalize_punctuation_spaces(text)
print(clean_text)
# >>> রহিম (২৩) এ কথা বলেন। তিনি (রহিম) আরও জানান, ১,২৪,৩৫,৬৫৪.৩২৩ কোটি টাকা ব্যায়ে...
```

#### Zero width characters normalization

There are two zero-width characters. These are Zero Width Joiner (ZWJ) and Zero Width Non Joiner (ZWNJ) characters. Generally ZWNJ is not used with Bangla texts and ZWJ joiner is used with `র` only. So, these characters are normalized based on these intuitions.

```python
import bkit

text = 'র‍্য‌াকেট'
print(f"text: {text} \t Characters: {list(text)}")
# >>> text: র‍্য‌াকেট     Characters: ['র', '\u200d', '্', 'য', '\u200c', 'া', 'ক', 'ে', 'ট']

clean_text = bkit.transform.normalize_zero_width_chars(text)
print(f"text: {clean_text} \t Characters: {list(clean_text)}")
# >>> text: র‍্যাকেট     Characters: ['র', '\u200d', '্', 'য', 'া', 'ক', 'ে', 'ট']
```

#### Halant (হসন্ত) normalization

This function normalizes halant (হসন্ত) [`0x09CD`] in Bangla text. While using this function, it is recommended to normalize the zero width characters at first, e.g. using the `bkit.transform.normalize_zero_width_chars()` function.

During the normalization it also handles the `ত্ -> ৎ` conversion. For a valid conjunct letter (যুক্তবর্ণ) where 'ত' is the former character, can take one of 'ত', 'থ', 'ন', 'ব', 'ম', 'য', and 'র' as the next character. The conversion is perform based on this intuition.

During the halant normalization, the following cases are handled.

- Remove any leading and tailing halant of a word and/or text.
- Replace two or more consecutive occurrences of halant by a single halant.
- Remove halant between any characters that do not follow or precede a halant character. Like a halant that follows or precedes a vowel, kar, য়, etc will be removed.
- Remove multiple fola (multiple ref, ro-fola and jo-fola)

```python
import bkit

text = 'আসন্্্ন আসফাকুল্লাহ্‌ আলবত্‍ আলবত্ র‍্যাব ই্সি'
print(list(text))
# >>> ['আ', 'স', 'ন', '্', '্', '্', 'ন', ' ', 'আ', 'স', 'ফ', 'া', 'ক', 'ু', 'ল', '্', 'ল', 'া', 'হ', '্', '\u200c', ' ', 'আ', 'ল', 'ব', 'ত', '্', '\u200d', ' ', 'আ', 'ল', 'ব', 'ত', '্', ' ', 'র', '\u200d', '্', 'য', 'া', 'ব', ' ', 'ই', '্', 'স', 'ি']

clean_text = bkit.transform.normalize_zero_width_chars(text)
clean_text = bkit.transform.normalize_halant(clean_text)
print(clean_text, list(clean_text))
# >>> আসন্ন আসফাকুল্লাহ আলবৎ আলবৎ র‍্যাব ইসি ['আ', 'স', 'ন', '্', 'ন', ' ', 'আ', 'স', 'ফ', 'া', 'ক', 'ু', 'ল', '্', 'ল', 'া', 'হ', ' ', 'আ', 'ল', 'ব', 'ৎ', ' ', 'আ', 'ল', 'ব', 'ৎ', ' ', 'র', '\u200d', '্', 'য', 'া', 'ব', ' ', 'ই', 'স', 'ি']
```

#### Kar ambiguity

Normalizes kar ambiguity with vowels, ঁ, ং, and ঃ. It removes any kar that is preceded by a vowel or consonant diacritics like: `আা` will be normalized to `আ`. In case of consecutive occurrence of kars like: `কাাাী`, only the first kar will be kept like: `কা`.

```python
import bkit

text = 'অংশইে অংশগ্রহণইে আাারো এখনওো আলবার্তোে সাধুু কাাাী'
print(list(text))
# >>> ['অ', 'ং', 'শ', 'ই', 'ে', ' ', 'অ', 'ং', 'শ', 'গ', '্', 'র', 'হ', 'ণ', 'ই', 'ে', ' ', 'আ', 'া', 'া', 'র', 'ো', ' ', 'এ', 'খ', 'ন', 'ও', 'ো', ' ', 'আ', 'ল', 'ব', 'া', 'র', '্', 'ত', 'ো', 'ে', ' ', 'স', 'া', 'ধ', 'ু', 'ু', ' ', 'ক', 'া', 'া', 'া', 'ী']

clean_text = bkit.transform.normalize_kar_ambiguity(text)
print(clean_text, list(clean_text))
# >>> অংশই অংশগ্রহণই আরো এখনও আলবার্তো সাধু কা ['অ', 'ং', 'শ', 'ই', ' ', 'অ', 'ং', 'শ', 'গ', '্', 'র', 'হ', 'ণ', 'ই', ' ', 'আ', 'র', 'ো', ' ', 'এ', 'খ', 'ন', 'ও', ' ', 'আ', 'ল', 'ব', 'া', 'র', '্', 'ত', 'ো', ' ', 'স', 'া', 'ধ', 'ু', ' ', 'ক', 'া']
```

#### Clean text

Clean text using the following steps sequentially:

<!-- no toc -->
1. [Removes all HTML tags](#html-tags)
2. [Removes all URLs](#urls)
3. [Removes all emojis (optional)](#emojis)
4. [Removes all digits (optional)](#clean-digits)
5. [Removes all punctuations (optional)](#clean-punctuations)
6. [Removes all extra spaces](#multiple-spaces)
7. [Removes all non bangla characters](#non-bangla-characters)

```python
import bkit

text = '<a href=some_URL>বাংলাদেশ</a>\nবাংলাদেশের   আয়তন ১.৪৭ লক্ষ কিলোমিটার!!!'

clean_text = bkit.transform.clean_text(text)
print(clean_text)
# >>> বাংলাদেশ বাংলাদেশের আয়তন লক্ষ কিলোমিটার
```

#### Clean punctuations

Remove punctuations with the given `replace_with` character/string.

```python
import bkit

text = 'আমরা মাঠে ফুটবল খেলতে পছন্দ করি!'

clean_text = bkit.transform.clean_punctuations(text)
print(clean_text)
# >>> আমরা মাঠে ফুটবল খেলতে পছন্দ করি

clean_text = bkit.transform.clean_punctuations(text, replace_with=' PUNC ')
print(clean_text)
# >>> আমরা মাঠে ফুটবল খেলতে পছন্দ করি PUNC
```

#### Clean digits

Remove any bangla digit from text by replacing with the given `replace_with` character/string.

```python
import bkit

text = 'তার বাসা ৭৯ নাম্বার রোডে।'

clean_text = bkit.transform.clean_digits(text)
print(clean_text)
# >>> তার বাসা    নাম্বার রোডে।

clean_text = bkit.transform.clean_digits(text, replace_with='#')
print(clean_text)
# >>> তার বাসা ## নাম্বার রোডে।
```

#### Multiple spaces

Clean multiple consecutive whitespace characters including space, newlines, tabs, vertical tabs, etc. It also removes leading and trailing whitespace characters.

```python
import bkit

text = 'তার বাসা ৭৯   \t\t নাম্বার   রোডে।\nসে খুব \v ভালো ছেলে।'

clean_text = bkit.transform.clean_multiple_spaces(text)
print(clean_text)
# >>> তার বাসা ৭৯ নাম্বার রোডে। সে খুব ভালো ছেলে।

clean_text = bkit.transform.clean_multiple_spaces(text, keep_new_line=True)
print(clean_text)
# >>> তার বাসা ৭৯ নাম্বার রোডে।\nসে খুব \n ভালো ছেলে।
```

#### URLs

Clean URLs from text and replace the URLs with any given string.

```python
import bkit

text = 'আমি https://xyz.abc সাইটে ব্লগ লিখি। এই ftp://10.17.5.23/books সার্ভার থেকে আমার বইগুলো পাবে। এই https://bn.wikipedia.org/wiki/%E0%A6%A7%E0%A6%BE%E0%A6%A4%E0%A7%81_(%E0%A6%AC%E0%A6%BE%E0%A6%82%E0%A6%B2%E0%A6%BE_%E0%A6%AC%E0%A7%8D%E0%A6%AF%E0%A6%BE%E0%A6%95%E0%A6%B0%E0%A6%A3) লিঙ্কটিতে ভালো তথ্য আছে।'

clean_text = bkit.transform.clean_urls(text)
print(clean_text)
# >>> আমি   সাইটে ব্লগ লিখি। এই   সার্ভার থেকে আমার বইগুলো পাবে। এই   লিঙ্কটিতে ভালো তথ্য আছে।

clean_text = bkit.transform.clean_urls(text, replace_with='URL')
print(clean_text)
# >>> আমি URL সাইটে ব্লগ লিখি। এই URL সার্ভার থেকে আমার বইগুলো পাবে। এই URL লিঙ্কটিতে ভালো তথ্য আছে।
```

#### Emojis

Clean emoji and emoticons from text and replace those with any given string.

```python
import bkit

text = 'কিছু ইমোজি হল: 😀🫅🏾🫅🏿🫃🏼🫃🏽🫃🏾🫃🏿🫄🫄🏻🫄🏼🫄🏽🫄🏾🫄🏿🧌🪸🪷🪹🪺🫘🫗🫙🛝🛞🛟🪬🪩🪫🩼🩻🫧🪪🟰'

clean_text = bkit.transform.clean_emojis(text, replace_with='<EMOJI>')
print(clean_text)
# >>> কিছু ইমোজি হল: <EMOJI>
```

#### HTML tags

Clean HTML tags from text and replace those with any given string.

```python
import bkit

text = '<a href=some_URL>বাংলাদেশ</a>'

clean_text = bkit.transform.clean_html(text)
print(clean_text)
# >>> বাংলাদেশ
```

#### Multiple punctuations

Remove multiple consecutive punctuations and keep the first punctuation only.

```python
import bkit

text = 'কি আনন্দ!!!!!'

clean_text = bkit.transform.clean_multiple_punctuations(text)
print(clean_text)
# >>> কি আনন্দ!
```

#### Special characters

Remove special characters like `$`, `#`, `@`, etc and replace them with the given string. If no character list is passed, `[$, #,  &, %, @]` are removed by default.

```python
import bkit

text = '#বাংলাদেশ$'

clean_text = bkit.transform.clean_special_characters(text, characters=['#', '$'], replace_with='')
print(clean_text)
# >>> বাংলাদেশ
```

#### Non Bangla characters

Non Bangla characters include characters and punctuation not used in Bangla like english or other language's alphabets  and replace them with the given string.

```python
import bkit

text = 'এই শূককীট হাতিশুঁড় Heliotropium indicum, অতসী, আকন্দ Calotropis gigantea গাছের পাতার রসালো অংশ আহার করে।'

clean_text = bkit.transform.clean_non_bangla(text, replace_with='')
print(clean_text)
# >>> এই শূককীট হাতিশুঁড়  , অতসী, আকন্দ  গাছের পাতার রসালো অংশ আহার করে
```

### Lemmatization

#### Lemmatize text

Lemmatize a given text. Generally expects the text to be a sentence.

```python
import bkit

text = 'পৃথিবীর জনসংখ্যা ৮ বিলিয়নের কিছু কম'

lemmatized = bkit.lemmatizer.lemmatize(text)

print(lemmatized)
# >>> পৃথিবী জনসংখ্যা ৮ বিলিয়ন কিছু কম
```

#### Lemmatize word

Lemmatize a word given the PoS information.

```python
import bkit

text = 'পৃথিবীর'

lemmatized = bkit.lemmatizer.lemmatize_word(text, 'noun')

print(lemmatized)
# >>> পৃথিবী
```

### Tokenization

Tokenize a given text. The `bkit.tokenizer` module is used to tokenizer text into tokens. It supports three types of tokenization.

#### Word tokenization

Tokenize text into words. Also separates some punctuations including comma, danda (।), question mark, etc.

```python
import bkit

text = 'তুমি কোথায় থাক? ঢাকা বাংলাদেশের রাজধানী। কি অবস্থা তার! ১২/০৩/২০২২ তারিখে সে ৪/ক ঠিকানায় গিয়ে ১২,৩৪৫ টাকা দিয়েছিল।'

tokens = bkit.tokenizer.tokenize(text)

print(tokens)
# >>> ['তুমি', 'কোথায়', 'থাক', '?', 'ঢাকা', 'বাংলাদেশের', 'রাজধানী', '।', 'কি', 'অবস্থা', 'তার', '!', '১২/০৩/২০২২', 'তারিখে', 'সে', '৪/ক', 'ঠিকানায়', 'গিয়ে', '১২,৩৪৫', 'টাকা', 'দিয়েছিল', '।']
```

#### Word and Punctuation tokenization

Tokenize text into words and any punctuation.

```python
import bkit

text = 'তুমি কোথায় থাক? ঢাকা বাংলাদেশের রাজধানী। কি অবস্থা তার! ১২/০৩/২০২২ তারিখে সে ৪/ক ঠিকানায় গিয়ে ১২,৩৪৫ টাকা দিয়েছিল।'

tokens = bkit.tokenizer.tokenize_word_punctuation(text)

print(tokens)
# >>> ['তুমি', 'কোথায়', 'থাক', '?', 'ঢাকা', 'বাংলাদেশের', 'রাজধানী', '।', 'কি', 'অবস্থা', 'তার', '!', '১২', '/', '০৩', '/', '২০২২', 'তারিখে', 'সে', '৪', '/', 'ক', 'ঠিকানায়', 'গিয়ে', '১২', ',', '৩৪৫', 'টাকা', 'দিয়েছিল', '।']
```

#### Sentence tokenization

Tokenize text into sentences.

```python
import bkit

text = 'তুমি কোথায় থাক? ঢাকা বাংলাদেশের রাজধানী। কি অবস্থা তার! ১২/০৩/২০২২ তারিখে সে ৪/ক ঠিকানায় গিয়ে ১২,৩৪৫ টাকা দিয়েছিল।'

okens = bkit.tokenizer.tokenize_sentence(text)

print(tokens)
# >>> ['তুমি কোথায় থাক?', 'ঢাকা বাংলাদেশের রাজধানী।', 'কি অবস্থা তার!', '১২/০৩/২০২২ তারিখে সে ৪/ক ঠিকানায় গিয়ে ১২,৩৪৫ টাকা দিয়েছিল।']
```

### Named Entity Recognition (NER)

Predicts the tags of the Named Entities of a given text.

```python
import bkit

text = 'তুমি কোথায় থাক? ঢাকা বাংলাদেশের রাজধানী। কি অবস্থা তার! ১২/০৩/২০২২ তারিখে সে ৪/ক ঠিকানায় গিয়ে ১২,৩৪৫.২৩ টাকা দিয়েছিল।'

ner = bkit.ner.Infer('ner-noisy-label')
predictions = ner(text)

print(predictions)
# >>> [('তুমি', 'O', 0.9998692), ('কোথায়', 'O', 0.99988306), ('থাক?', 'O', 0.99983954), ('ঢাকা', 'B-GPE', 0.99891424), ('বাংলাদেশের', 'B-GPE', 0.99710876), ('রাজধানী।', 'O', 0.9995414), ('কি', 'O', 0.99989176), ('অবস্থা', 'O', 0.99980336), ('তার!', 'O', 0.99983263), ('১২/০৩/২০২২', 'B-D&T', 0.97921854), ('তারিখে', 'O', 0.9271435), ('সে', 'O', 0.99934834), ('৪/ক', 'B-NUM', 0.8297553), ('ঠিকানায়', 'O', 0.99728775), ('গিয়ে', 'O', 0.9994825), ('১২,৩৪৫.২৩', 'B-NUM', 0.99740463), ('টাকা', 'B-UNIT', 0.99914896), ('দিয়েছিল।', 'O', 0.9998908)]
```

### Parts of Speech (PoS) tagging

Predicts the tags of the parts of speech of a given text.

```python
import bkit

text = 'তুমি কোথায় থাক? 5.33 ঢাকা বাংলাদেশের রাজধানী। কি অবস্থা তার! ১২/০৩/২০২২ তারিখে সে ৪/ক ঠিকানায় গিয়ে ১২,৩৪৫.২৩ টাকা দিয়েছিল।'

pos = bkit.pos.Infer('pos-noisy-label')
predictions = pos(text)

print(predictions)
# >>> [('তুমি', 'PRO', 0.99638426), ('কোথায়', 'ADV', 0.91948754), ('থাক?', 'VF', 0.91336894), ('ঢাকা', 'NNP', 0.99534553), ('বাংলাদেশের', 'NNP', 0.990851), ('রাজধানী।', 'NNC', 0.9912845), ('কি', 'ADV', 0.55443066), ('অবস্থা', 'NNC', 0.9948944), ('তার!', 'PRO', 0.996412), ('১২/০৩/২০২২', 'QF', 0.98805654), ('তারিখে', 'NNC', 0.99609643), ('সে', 'PRO', 0.9955552), ('৪/ক', 'QF', 0.9736483), ('ঠিকানায়', 'NNC', 0.9975303), ('গিয়ে', 'VNF', 0.92555565), ('১২,৩৪৫.২৩', 'QF', 0.9918138), ('টাকা', 'NNC', 0.9986051), ('দিয়েছিল।', 'VF', 0.9942147)]
```

### Shallow Parsing (Constituency Parsing)

Predicts the shallow parsing tags of a given text.

```python
import bkit

text = 'তুমি কোথায় থাক? ঢাকা বাংলাদেশের রাজধানী। কি অবস্থা তার! ১২/০৩/২০২২ তারিখে সে ৪/ক ঠিকানায় গিয়ে ১২,৩৪৫.২৩ টাকা দিয়েছিল।'

shallow = bkit.shallow.Infer(pos_model='pos-noisy-label')
predictions = shallow(text)

print(predictions)
# >>> (S (VP (NP (PRO তুমি)) (VP (ADVP (ADV কোথায়)) (VF থাক))) (NP (NNP ?) (NNP ঢাকা) (NNC বাংলাদেশের)) (ADVP (ADV রাজধানী)) (NP (NP (NP (NNC ।)) (NP (PRO কি))) (NP (QF অবস্থা) (NNC তার)) (NP (PRO !))) (NP (NP (QF ১২/০৩/২০২২) (NNC তারিখে)) (VNF সে) (NP (QF ৪/ক) (NNC ঠিকানায়))) (VF গিয়ে))
```
