
<!-- README.md is generated from README.Rmd. Please edit that file -->

# Pipeline for counting terms in an annual report

In this guide, we will walk you through the step-by-step process to
perform a complete term count using the term list from the paper

**“The Standardization of Accounting Language”.**

The guide is designed to be accessible to both non-technical and
technical audiences, and will explain each step in detail, starting with
a general description followed by more technical details.

First, we load the required libraries and source the “functions.R” file:

``` r
library <- function(...) suppressPackageStartupMessages(base::library(...))

library(tidyverse); library(rTermCount); library(readtext); library(openxlsx); library(stringi)
library(qdapTools)

source("functions.R", encoding = "UTF-8")
```

## Step 1: Retrieving a document

We will start by obtaining an Annual Report from the website of BASF SE
for the year 2020. This document is already included in the package, so
there is no need to download it again. The document will be stored in a
folder within the project called “00_documents”.

*Technical details:* In this part of the process, the code makes sure
that the necessary folder exists and that the document is saved in the
correct location. The **`dir.exists()`** function checks if the
directory exists, and **`dir.create()`** creates the directory if it
doesn’t. The **`download.file()`** function is used to download and save
the file in the specified folder.

(Please note that downloading a file can differ from operating system to
operating system)

``` r
.dir_doc  <- "00_documents"
.url_basf <- "https://report.basf.com/2020/en/servicepages/downloads/files/basf-report-2020-basf-ar20.pdf"
.fil_doc  <- file.path(.dir_doc, "2020_BASF.pdf")

if (!dir.exists(.dir_doc)) dir.create(.dir_doc, FALSE, TRUE)
if (!file.exists(.fil_doc))  download.file(.url_basf, .fil_doc, mode = "wb")
```

## Step 2: Converting PDF

Next, we will read the downloaded PDF file and transform its contents
into a format that can be easily analyzed. We will use a package called
`readtext` to achieve this.

*Technical details:* The **`readtext::readtext()`** function reads the
PDF file and stores its content in a dataframe called **`doc_raw`**.

``` r
doc_raw <- readtext::readtext(.fil_doc) %>%
  dplyr::mutate(doc_id = gsub(".pdf$", "", doc_id))
```

After executing this step, we will have a table containing two columns:

1.  `doc_id`: The name of the document

2.  `text`: The entire text of the PDF stored in a single row

Once the PDF is read, we will use a function from the `rTermCount`
Package to create a table containing individual words (tokens) from the
document. We wrap this function in a custom function to replicate the
standardization procedure used in the paper

*Technical details:* The **`tokenize_corpus()`** function processes and
tokenizes the input text using a series of string operations and
adjustment rules. It starts by defining a pipeline of string operations,
such as standardizing quotes, hyphens, slashes, brackets, and more.
These operations are performed using the **`string_ops()`** function,
which takes an input string and a table of string operations. This table
is then used to iterate through each row, applying the corresponding
string operation to the input string. The **`replace_words()`** function
replaces words in the input string based on the replacement rules
specified in a separate table. The **`string_operations()`** function is
a pipeline of string operations that sequentially calls the
**`string_ops()`** function. The **`get_adjustment_lists()`** function
reads and processes multiple files to create a list of adjustment rules
for text preprocessing. The **`rTermCount::prep_document()`** function
is then called to prepare the document for term counting. In summary,
this script provides a comprehensive tokenization and preprocessing
workflow for text data, using the pipeline described in the Online
Appendix to the paper, in detail:

1.  Apply a set of Regular Expressions (RegEx) to abstract away from
    minor differences in the linguistic notation of terms

2.  Normalize different varieties of English by converting all British
    English spellings to American English

3.  Account for stylistic differences arising from different
    hyphenations and the inclusion/exclusion of blank characters

4.  Lemmatize our terms to account for different inflectional endings
    and plurals

5.  Remove all punctuation

Please refer to the Online Appendix accompanying the paper for more
information. Also, refer to the R script functions.R for a complete
description of every function used.

``` r
doc_mod <- tokenize_corpus(doc_raw)
```

Below the first six entries of this table:

<table class=" lightable-paper table" style="font-family: &quot;Arial Narrow&quot;, arial, helvetica, sans-serif; margin-left: auto; margin-right: auto; font-size: 10px; margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:left;">
doc_id
</th>
<th style="text-align:right;">
pag_id
</th>
<th style="text-align:right;">
par_id
</th>
<th style="text-align:right;">
sen_id
</th>
<th style="text-align:right;">
tok_id
</th>
<th style="text-align:left;">
token
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
2020_BASF
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
basf
</td>
</tr>
<tr>
<td style="text-align:left;">
2020_BASF
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:left;">
report
</td>
</tr>
<tr>
<td style="text-align:left;">
2020_BASF
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
2020
</td>
</tr>
<tr>
<td style="text-align:left;">
2020_BASF
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
4
</td>
<td style="text-align:left;">
economic
</td>
</tr>
<tr>
<td style="text-align:left;">
2020_BASF
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
5
</td>
<td style="text-align:left;">
environmental
</td>
</tr>
<tr>
<td style="text-align:left;">
2020_BASF
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
6
</td>
<td style="text-align:left;">
and
</td>
</tr>
</tbody>
</table>

## Step 3: Retrieving the Term List

Now, we will obtain the term list that was used in the paper. This term
list is located in the folder “01_termlist”. Since the term list is
already prepared, no further adjustments are needed.

*Technical details:* The **`openxlsx::read.xlsx()`** function is used to
read the term list from the Excel file, and the
**`rTermCount::prep_termlist()`** function processes the term list
without any further adjustments.

``` r
termlist <- openxlsx::read.xlsx("01_termlist/term_list.xlsx") %>%
  rTermCount::prep_termlist()
```

Below the first six entries of this table:

<table class=" lightable-paper table" style="font-family: &quot;Arial Narrow&quot;, arial, helvetica, sans-serif; margin-left: auto; margin-right: auto; font-size: 10px; margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:right;">
tid
</th>
<th style="text-align:right;">
ngram
</th>
<th style="text-align:left;">
term
</th>
<th style="text-align:left;">
term_orig
</th>
<th style="text-align:left;">
term_raw
</th>
<th style="text-align:left;">
cat1
</th>
<th style="text-align:left;">
cat2
</th>
<th style="text-align:left;">
cat3
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:left;">
10 k
</td>
<td style="text-align:left;">
10 k
</td>
<td style="text-align:left;">
10 - k
</td>
<td style="text-align:left;">
FinA
</td>
<td style="text-align:left;">
GAAP
</td>
<td style="text-align:left;">
CF
</td>
</tr>
<tr>
<td style="text-align:right;">
2
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:left;">
10 ksb
</td>
<td style="text-align:left;">
10 ksb
</td>
<td style="text-align:left;">
10 - ksb
</td>
<td style="text-align:left;">
BusA
</td>
<td style="text-align:left;">
NA
</td>
<td style="text-align:left;">
NA
</td>
</tr>
<tr>
<td style="text-align:right;">
3
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:left;">
10 q
</td>
<td style="text-align:left;">
10 q
</td>
<td style="text-align:left;">
10 - q
</td>
<td style="text-align:left;">
FinA
</td>
<td style="text-align:left;">
GAAP
</td>
<td style="text-align:left;">
NA
</td>
</tr>
<tr>
<td style="text-align:right;">
4
</td>
<td style="text-align:right;">
5
</td>
<td style="text-align:left;">
12 month expect credit loss
</td>
<td style="text-align:left;">
12 month expect credit loss
</td>
<td style="text-align:left;">
12 - month expected credit losses
</td>
<td style="text-align:left;">
FinA
</td>
<td style="text-align:left;">
GAAP
</td>
<td style="text-align:left;">
NA
</td>
</tr>
<tr>
<td style="text-align:right;">
5
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
13 th period
</td>
<td style="text-align:left;">
13 th period
</td>
<td style="text-align:left;">
13 th period
</td>
<td style="text-align:left;">
BusA
</td>
<td style="text-align:left;">
NA
</td>
<td style="text-align:left;">
NA
</td>
</tr>
<tr>
<td style="text-align:right;">
6
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
15 minute rule
</td>
<td style="text-align:left;">
15 minute rule
</td>
<td style="text-align:left;">
15 minute rule
</td>
<td style="text-align:left;">
BusA
</td>
<td style="text-align:left;">
NA
</td>
<td style="text-align:left;">
NA
</td>
</tr>
</tbody>
</table>

## Step 4: Retrieve Positions

In this step, we will identify the exact position of every term in the
document. This information will be useful for further analysis.

*Technical details:* The **`rTermCount::position_count()`** function is
used to find the positions of terms in the document. The resulting
dataframe, **`positions`**, contains information about the term
positions in the text.

``` r
positions <- rTermCount::position_count(termlist, doc_mod, sen_id)
```

Below the first six entries of this table:

<table class=" lightable-paper table" style="font-family: &quot;Arial Narrow&quot;, arial, helvetica, sans-serif; margin-left: auto; margin-right: auto; font-size: 10px; margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:left;">
doc_id
</th>
<th style="text-align:right;">
tid
</th>
<th style="text-align:right;">
ngram
</th>
<th style="text-align:left;">
term
</th>
<th style="text-align:right;">
start
</th>
<th style="text-align:right;">
stop
</th>
<th style="text-align:left;">
dup
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
2020_BASF
</td>
<td style="text-align:right;">
37418
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
performance
</td>
<td style="text-align:right;">
8
</td>
<td style="text-align:right;">
8
</td>
<td style="text-align:left;">
FALSE
</td>
</tr>
<tr>
<td style="text-align:left;">
2020_BASF
</td>
<td style="text-align:right;">
43368
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
sale
</td>
<td style="text-align:right;">
18
</td>
<td style="text-align:right;">
18
</td>
<td style="text-align:left;">
FALSE
</td>
</tr>
<tr>
<td style="text-align:left;">
2020_BASF
</td>
<td style="text-align:right;">
50960
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:left;">
year end
</td>
<td style="text-align:right;">
28
</td>
<td style="text-align:right;">
29
</td>
<td style="text-align:left;">
FALSE
</td>
</tr>
<tr>
<td style="text-align:left;">
2020_BASF
</td>
<td style="text-align:right;">
37542
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
personnel
</td>
<td style="text-align:right;">
47
</td>
<td style="text-align:right;">
47
</td>
<td style="text-align:left;">
FALSE
</td>
</tr>
<tr>
<td style="text-align:left;">
2020_BASF
</td>
<td style="text-align:right;">
18126
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
expense
</td>
<td style="text-align:right;">
48
</td>
<td style="text-align:right;">
48
</td>
<td style="text-align:left;">
FALSE
</td>
</tr>
<tr>
<td style="text-align:left;">
2020_BASF
</td>
<td style="text-align:right;">
23521
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
group
</td>
<td style="text-align:right;">
56
</td>
<td style="text-align:right;">
56
</td>
<td style="text-align:left;">
FALSE
</td>
</tr>
</tbody>
</table>

| Column   | Description                                                                                                                                                                                                                                                                 |
|----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `doc_id` | This column contains the identifier for each document.                                                                                                                                                                                                                      |
| `tid`    | This column represents the unique identifier for each term in the term list.                                                                                                                                                                                                |
| `ngram`  | This column contains the length of the n-gram for each term.                                                                                                                                                                                                                |
| `term`   | Contains the term associated with the Term ID (tid)                                                                                                                                                                                                                         |
| `start`  | This column represents the starting position of the term in the document. Positions are numbered sequentially, starting from 1.                                                                                                                                             |
| `stop`   | This column represents the ending position of the term in the document. For single-word terms, the start and stop positions will be the same. For multi-word terms (n-grams), the stop position will be greater than the start position.                                    |
| `dup`    | This column contains a logical value (TRUE or FALSE) indicating whether the term is a duplicate within the same n-gram sequence. If a term appears more than once within the same n-gram sequence, the ‘dup’ value will be set to TRUE, otherwise, it will be set to FALSE. |

## Step 5: Summarize Counts

Finally, we will create a summary of the term count, which will provide
us with an overview of how frequently each term appears in the document.
The result will be a table containing the most frequently occurring
terms with more than two words (ngram \> 2).

``` r
counts <- rTermCount::summarize_count(positions)
```

Below the first ten entries of this table:

<table class=" lightable-paper table" style="font-family: &quot;Arial Narrow&quot;, arial, helvetica, sans-serif; margin-left: auto; margin-right: auto; font-size: 10px; margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:left;">
doc_id
</th>
<th style="text-align:right;">
tid
</th>
<th style="text-align:right;">
ngram
</th>
<th style="text-align:left;">
term
</th>
<th style="text-align:right;">
n_dup
</th>
<th style="text-align:right;">
n_uni
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
2020_BASF
</td>
<td style="text-align:right;">
8206
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
consolidated financial statement
</td>
<td style="text-align:right;">
401
</td>
<td style="text-align:right;">
401
</td>
</tr>
<tr>
<td style="text-align:left;">
2020_BASF
</td>
<td style="text-align:right;">
24954
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
income from operation
</td>
<td style="text-align:right;">
72
</td>
<td style="text-align:right;">
72
</td>
</tr>
<tr>
<td style="text-align:left;">
2020_BASF
</td>
<td style="text-align:right;">
39599
</td>
<td style="text-align:right;">
4
</td>
<td style="text-align:left;">
property plant and equipment
</td>
<td style="text-align:right;">
96
</td>
<td style="text-align:right;">
67
</td>
</tr>
<tr>
<td style="text-align:left;">
2020_BASF
</td>
<td style="text-align:right;">
9197
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
cost of capital
</td>
<td style="text-align:right;">
63
</td>
<td style="text-align:right;">
56
</td>
</tr>
<tr>
<td style="text-align:left;">
2020_BASF
</td>
<td style="text-align:right;">
24886
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
income after tax
</td>
<td style="text-align:right;">
44
</td>
<td style="text-align:right;">
44
</td>
</tr>
<tr>
<td style="text-align:left;">
2020_BASF
</td>
<td style="text-align:right;">
5787
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
cash flow from
</td>
<td style="text-align:right;">
43
</td>
<td style="text-align:right;">
43
</td>
</tr>
<tr>
<td style="text-align:left;">
2020_BASF
</td>
<td style="text-align:right;">
12397
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
depreciation and amortization
</td>
<td style="text-align:right;">
44
</td>
<td style="text-align:right;">
41
</td>
</tr>
<tr>
<td style="text-align:left;">
2020_BASF
</td>
<td style="text-align:right;">
5622
</td>
<td style="text-align:right;">
4
</td>
<td style="text-align:left;">
cash and cash equivalent
</td>
<td style="text-align:right;">
36
</td>
<td style="text-align:right;">
36
</td>
</tr>
<tr>
<td style="text-align:left;">
2020_BASF
</td>
<td style="text-align:right;">
42001
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
research and development
</td>
<td style="text-align:right;">
53
</td>
<td style="text-align:right;">
33
</td>
</tr>
<tr>
<td style="text-align:left;">
2020_BASF
</td>
<td style="text-align:right;">
34325
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
oil and gas
</td>
<td style="text-align:right;">
31
</td>
<td style="text-align:right;">
31
</td>
</tr>
</tbody>
</table>

The output table will have the following:

| Column   | Description                                                                                                |
|----------|------------------------------------------------------------------------------------------------------------|
| `doc_id` | This column contains the identifier for each document.                                                     |
| `tid`    | This column represents the unique identifier for each term in the term list.                               |
| `ngram`  | This column contains the length of the n-gram for each term.                                               |
| `term`   | Contains the term associated with the Term ID (tid)                                                        |
| `n_dup`  | This column represents the number of occurrences of the term in the document, including duplicates.        |
| `n_uni`  | This column represents the number of unique occurrences of the term in the document, excluding duplicates. |
