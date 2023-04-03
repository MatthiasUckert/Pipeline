
<!-- README.md is generated from README.Rmd. Please edit that file -->

# Pipeline

In this guide, we will walk you through a step-by-step process to
perform a complete term count using the termlist from the paper

**“The Standardization of Accounting Language”.**

The guide is designed to be accessible to both non-technical and
technical audiences, and will explain each step in detail, starting with
a general description followed by in-depth details.

First, we load the required libraries and source the “functions.R” file:

``` r
library <- function(...) suppressPackageStartupMessages(base::library(...))

library(tidyverse); library(rTermCount); library(readtext); library(openxlsx); library(stringi)
library(qdapTools)

source("functions.R", encoding = "UTF-8")
```

## Step 1: Retrieving a document

We will start by obtaining an Annual Report from BASF SE for the year
2020. This document is already included in the package, so there is no
need to download it again. The document will be stored in a folder
within the project called “00_documents”.

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
“readtext” to achieve this.

*Technical details:* The **`readtext::readtext()`** function reads the
PDF file and stores its content in a dataframe called **`doc_raw`**.

``` r
doc_raw <- readtext::readtext(.fil_doc) %>%
  dplyr::mutate(doc_id = gsub(".pdf$", "", doc_id)) 

head(doc_raw)
#> readtext object consisting of 1 document and 0 docvars.
#> # Description: df [1 x 2]
#>   doc_id    text               
#> * <chr>     <chr>              
#> 1 2020_BASF "\"BASF Repor\"..."
```

After executing this step, we will have a table (dataframe) containing
two columns:

1.  `doc_id`: The name of the document

2.  `text`: The entire text of the PDF stored in a single row

Once the PDF is read, we will use a function from the `rTermCount`
Package to create a table containing individual words (tokens) from the
document.

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
for text preprocessing. These rules are stored in a list of data frames,
which are then used by the **`tokenize_corpus()`** function. The
**`tokenize_corpus()`** function starts by checking whether the input is
a file path or a text string. If it’s a file path, the text is read
using the **`readtext::readtext()`** function; otherwise, the text is
directly assigned to a variable. Next, replaces words using the ‘split’
list from the adjustment rules. The **`rTermCount::prep_document()`**
function is then called to prepare the document for term counting.
Finally, the function replaces words in the token column using the
‘us_uk’ and ‘lemma’ lists from the adjustment rules. In summary, this
script provides a comprehensive tokenization and preprocessing workflow
for text data, using the pipeline described in the Online Appendix to
the paper, in detail:

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

head(doc_mod)
#> # A tibble: 6 x 6
#>   doc_id    pag_id par_id sen_id tok_id token        
#>   <chr>      <int>  <int>  <int>  <int> <chr>        
#> 1 2020_BASF      1      1      1      1 basf         
#> 2 2020_BASF      1      1      1      2 report       
#> 3 2020_BASF      1      1      1      3 2020         
#> 4 2020_BASF      1      1      1      4 economic     
#> 5 2020_BASF      1      1      1      5 environmental
#> 6 2020_BASF      1      1      1      6 and
```

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

## Step 4: Retrieve Positions

In this step, we will identify the exact position of every term in the
document. This information will be useful for further analysis.

*Technical details:* The **`rTermCount::position_count()`** function is
used to find the positions of terms in the document. The resulting
dataframe, **`positions`**, contains information about the term
positions in the text.

``` r
positions <- rTermCount::position_count(termlist, doc_mod, sen_id)

head(positions)
#> # A tibble: 6 x 7
#>   doc_id      tid ngram term        start  stop dup  
#>   <chr>     <dbl> <int> <chr>       <int> <dbl> <lgl>
#> 1 2020_BASF 37418     1 performance     8     8 FALSE
#> 2 2020_BASF 43368     1 sale           18    18 FALSE
#> 3 2020_BASF 50960     2 year end       28    29 FALSE
#> 4 2020_BASF 37542     1 personnel      47    47 FALSE
#> 5 2020_BASF 18126     1 expense        48    48 FALSE
#> 6 2020_BASF 23521     1 group          56    56 FALSE
```

The **`positions`** dataframe contains the following columns:

1.  **`doc_id`**: This column contains the identifier for each document.

2.  **`tid`**: This column represents the unique identifier for each
    term in the term list.

3.  **`ngram`**: This column contains the length of the n-gram for each
    term.

4.  **`term`**: Contains the term associated with the Term ID (tid)

5.  **`start`**: This column represents the starting position of the
    term in the document. Positions are numbered sequentially, starting
    from 1.

6.  **`stop`**: This column represents the ending position of the term
    in the document. For single-word terms, the start and stop positions
    will be the same. For multi-word terms (n-grams), the stop position
    will be greater than the start position.

7.  **`dup`**: This column contains a logical value (TRUE or FALSE)
    indicating whether the term is a duplicate within the same n-gram
    sequence. If a term appears more than once within the same n-gram
    sequence, the ‘dup’ value will be set to TRUE, otherwise, it will be
    set to FALSE.

In the **`positions`** dataframe, the exact positions of terms in each
document are stored, which can be useful for further text analysis.

## Step 5: Summarize Counts

Finally, we will create a summary of the term count, which will provide
us with an overview of how frequently each term appears in the document.
The result will be a table containing the most frequently occurring
terms with more than two words (ngram \> 2).

``` r
counts <- rTermCount::summarize_count(positions)

counts %>%
  dplyr::arrange(-n_uni) %>%
  dplyr::filter(ngram > 2) %>%
  dplyr::slice(1:10)
#> # A tibble: 10 x 6
#>    doc_id      tid ngram term                             n_dup n_uni
#>    <chr>     <dbl> <int> <chr>                            <int> <int>
#>  1 2020_BASF  8206     3 consolidated financial statement   401   401
#>  2 2020_BASF 24954     3 income from operation               72    72
#>  3 2020_BASF 39599     4 property plant and equipment        96    67
#>  4 2020_BASF  9197     3 cost of capital                     63    56
#>  5 2020_BASF 24886     3 income after tax                    44    44
#>  6 2020_BASF  5787     3 cash flow from                      43    43
#>  7 2020_BASF 12397     3 depreciation and amortization       44    41
#>  8 2020_BASF  5622     4 cash and cash equivalent            36    36
#>  9 2020_BASF 42001     3 research and development            53    33
#> 10 2020_BASF 34325     3 oil and gas                         31    31
```

The output table will have the following:

1.  **`doc_id`**: This column contains the identifier for each document.

2.  **`tid`**: This column represents the unique identifier for each
    term in the term list.

3.  **`ngram`**: This column contains the length of the n-gram for each
    term.

4.  **`term`**: Contains the term associated with the Term ID (tid)

5.  **`n_dup`**: This column represents the number of occurrences of the
    term in the document, including duplicates. If a term appears more
    than once within the same n-gram sequence, each appearance will be
    counted.

6.  **`n_uni`**: This column represents the number of unique occurrences
    of the term in the document, excluding duplicates. Only the first
    appearance of a term within the same n-gram sequence will be
    counted.

  
