## CS50 TSE Querier
# Design Spec

According to the Querier *Requirements Spec*, the TSE `querier` is a standalone program that reads a user input for a query and references the index files created by the `indexer`, rebuilds the corresponding index, and matches the user's query to a number of docs that contain each word specified.

# User interface

The `querier` takes the command specified by the user, parsing each argument to assure its validity. 

`querier pageDirectory indexFilename`

For example, if letters10 is a pageDirectory in ../data,

`$ querier ../data/letters10 ../data/letters10/.index`

It then takes an additional line of input that corresponds to the user's _query_. For example, if the user wanted to find a document that contained the words *algorithm* and *this*,

`Input query: algorithm AND this`

On initialization, the command must always have two arguments. The user's _query_ must meet a few conditions:
1. The user's input cannot begin with the word *and* or *or*
2. The user's input cannot end with the word *and* or *or*
3. The user's input cannot contain a sequence with multiple *and*s or *or*s without words in between.

# Inputs and outputs

**Input:** the `querier` first ensures that the `pageDirectory` parameter provided by the user is valid, and that the `indexFilename` is a readable file. Afterward, it takes a _query_ of words that the user is searching for within the documents referenced in the domain of the directory.

The `querier` will create a new `index` from the file specified at the `indexFilename` using the `index_load` function.

**Output:** We output, in order, the docIDs that have the most frequent appearances of each word specified in the _query_ by the user as described in the *Requirements Spec*.

# Functional decomposition into modules

We anticipate the following modules or functions:

1. *main*, which parses arguments and initializes other modules;
2. *verifyString*, which ensures that the string given contains only letters and spaces;
3. *tokenize*, which creates an array of strings from the user's input;
4. *validate*, which validates that the user did not break any of the rules specified above in their query;
5. *makeCounter*, which creates the counter storing the docIDs that meet the user's _query_'s specifications;
6. *rank*, which prints the docIDs by appearance in order;


We will also reuse the following modules or functions:

1. *index*, for its *index_load* function;
2. pagedir, for its *pagedir_validate* function;
3. word, for its *normalizeWord* function.

# Pseudo code for logic/algorithmic flow

The `querier` will run as follows:

```c
parse the command line
validate parameters
call verifyString
if verified
    call tokenize

    if validated
        call makeCounter

        call rank

delete allocated memory
```

All major functions' functionality is described further in the `IMPLEMENTATION.md` file.

# Major data structures

The key data structure is the `index`, which as mentioned in the `DESIGN.md` file for indexer, works by "mapping from word to (docID, #occurrences) pairs."

Additionally, we will define a new `twoCounters` structure which will allow us to reference the data found in two counters which are being compared by passing them into the `counters_iterate` function as the pointer that must be passed into it.

# Testing plan

**Unit testing.** A program `testing.sh` will be designed with a number of edge cases and successful cases to demonstrate the capacity of the function to accomplish the tasks outlined in the *Requirements Spec*. The output from these tests will be placed in the `testing.out` file.

**Integration testing.** The `querier` will be tested by loading an index that was created by the `indexer` module from a file as an index using the `index_load` function. The resulting index will be used to find counters that correspond to each word parsed in the user's _query_. By manually comparing the counts at each docID to the output of the `querier`, we can verify its functionality.

Examples of cases that will be tested for `error` output. 
1. no arguments 
2. one argument 
3. three or more arguments 
4. invalid pageDirectory 
5. invalid pageDirectory (not a crawler directory) 
6. invalid indexFile (non-existent path) 
7. invalid indexFile (read-only directory)

Additionally, the `make valgrind` command will be run to assure that the function successfully allocates and frees all memory.