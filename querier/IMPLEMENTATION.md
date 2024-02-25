## CS50 TSE Querier
# Implementation Spec

# Data structures

We use a number of data structures in the querier as created in earlier steps of the TSE creation. For example, one of the crucial aspects of the querier's functionality is its dependence on the `index` data structure created in `index.c`. Like in `indexer.c`, the index used by the querier stores the `counters` that store each word's docID-count pairs.

Additionally, in this module we define a `twoCounters` data type which can be passed into the `counters_iterate` function while simultaneously maintaining a pointer to both counters of an intersection set.

Furthermore, in this module we define a `rankTool` data type which can be passed into the `counters_iterate` function as a void pointer while containing the pageDirectory that the user must reference to find the URL of a certain document and the counter that is iterated through for the `max_find` function.

# Control flow

The Querier is implemented in one file `querier.c`, with six primary functions and a number of helper functions. Some of the funtions defined in the `common` directory were reused for the querier.

**main**

The main function ensures that all of the inputs meet the expectations specified in the *Requirements Spec* and calls the five other primary functions within the `querier.c` file: _verifyString_, _tokenize_, _validate_, _makeCounter_, and _rank_, then exits zero if everything functioned successfully.

**verifyString**

After verifying the arguments extracted from the command line during the *main* function, takes user input for a _query_ which must be verified meets the specified conditions of the function in the *Requirements Spec*

Pseudocode:
```c
determine the size of the query
while there are more characters to scan
	if a letter is not a space and not a letter
        return false

return true
```

**tokenize**

After determining the validity of the _query_, creates a new array of strings that stores the location of every word in the query. The size of the array is capped at the *length of the string divided by 2 + 1* because there are only so many words that can be found within a string of a fixed size.

Pseudocode:
```c
determine the size of the query
initialize the array of words
initialize a boolean that indicates whether the character we reference comes after a word or starts a word.
while there are more characters to be scanned
    if the character is a space
        if the character comes right at the end of a word
            set that character to null
            flip the boolean to indicate no longer directly after a word
    
    if the character is a letter
        if the character comes right at the beginning of a word
            the pointer to the word is added to the array of strings
            flip the boolean to indicate next space is at end of word

set the last string of the array to null to be used by later iterators

return the array
```

**validate**
This function takes the array of words created in the `tokenize` function. If it meets any of the following conditions, it returns false; otherwise, it returns true: 1. the first word is *and* or *or*, 2. the last word is *and* or *or*, 3. the words *and* and *or* appear directly next to each other.

Pseudocode:
```c
store an int that corresponds to the location within the words array
if the first word is and or or
    return false

iterate through every word
    if a word is and or or
        if the word preceding it was also and or or
            return false

if the last word is and or or
    return false

return true

```

**makeCounter**
This function takes the array of arrays created in `tokenize` and verified in `validate` to create a counter that stores the docIDs of all webpages that contain the combination of words specified in the _query_. It does so using the `intersection` and `join` functions, which find the overlap between the counters of two words and the combined total of the counters, respectively.

Pseudocode:
```c
initialize an andsequence counter
initialize an orsequence counter

iterate through every word in the query
    if the word is neither and nor or
        if the andsequence counter exists,
            find the intersection of the pre-existing counter and the counter being created with the current word
        
        else
            initialize an andsequence counter
    
    if the word is or
        run the union of the andsequence and orsequence counters
        reset the andsequence counter
    
    if the word is and
        go back to the beginning with the next word
    

return the final counter
```

**rank**
This function takes the counters created by `makeCounter` and lists out the docIDs where a certain word appears most often until the document where a certain word appears the least often (at least once).

Pseudocode:
```c
Iterate through every docID-count pair in the counters created.
    Store a max count for the first pair
    Iterate through every docID-count pair in the counters created.
        if the max count supersedes the above mentioned count, replace it
    
    print the docID with the highest count
    set that docID to have a count 0 so it is never repeated.
```

**Helper functions**

*join/join_helper*

The `join` functions act as the union between two separate words' docID lists. It functions by adding the count of each respective word's counts for each docID, thus storing any docID where there is a count of at least one for either word. This is used when queries use the word `or`.


*intersect/intersect_helper*

The `intersect` functions act as the intersection between two separate words' docID lists, and as a foil to the `join` functions. It functions by taking the minimum of the two counts of two words, thus eliminating any word that doesn't appear at least once in both documents. This is used when queries use the word `and`.

**Other modules**

*pagedir*

We reuse the `pagedir` file that was created for the crawler and indexer. In the querier, we don't add any methods, but we use the `pagedir_validate` method. Detailed descriptions of these functions can be found in the `pagedir.h` file.

*word*

Similarly, we reuse the `normalizeWord` function defined in the `word.c` file during indexer.

*index*

Once again, we reuse the `index` data structure created for the indexer, specifically making use of the `index_load` feature to take a file and create a readable index data structure.

# Function prototypes

**querier**
```c
int main(const int argc, char* argv[]);
bool verifyString(char* input);
char** tokenize(char* query);
bool validate(char** words);
counters_t* makeCounter(index_t* index, char** words);
static void intersect(counters_t* counter1, counters_t* counter2);
static void join(counters_t* counter1, counters_t* counter2);
static void join_helper(void* arg, const int key, int item);
static void intersect_helper(void* arg, const int key, int item);
void rank(counters_t* validDocs);
static void rank_compare(void* arg, const int key, int item);
static void find_max(void* arg, const int key, int item);
```

# Error handling and recovery

All the command-line parameters are rigorously checked before any data structures are allocated or work begins; problems result in a message printed to stderr and a non-zero exit status.

Out-of-memory errors are handled through the use of `mem_malloc` and `free` or various data structure `delete` functions.

All code uses defensive-programming tactics to catch and exit as seen in the `main` function of `querier.c`; e.g., if a function receives bad parameters.

That said, certain errors are caught and handled internally: for example, the `main` function returns an error if the user attempts to pass in a query that starts with *and* or *or*, or if it has an input with *and* and *or* directly next to each other. Similarly, invalid pageDirectories and indexFilenames are caught, as are queries that don't use letters and spaces.

# Testing plan

Here is an implementation-specific testing plan.

*Unit testing*

The main unit for the purpose of the querier is the querier, itself. While the `pagedir`, `word`, and `index` aid in the completion of the querier, the `querier` represents the whole system. Querier inputs by the user must be tested before verifying the functionality of the successful cases.

*Integration/system testing*

In the case that the `indexer` created by James Quirk did not function properly, the outputs from the `querier` should be cross-referenced with the `shared/tse/indices` text files to double check the correctness of the output. The `querier` will take a `testing.sh` file that prints to `testing.out` a number of edge cases, errors, and successful cases' outputs to ensure proper functionality.

Samples of successful tests include the result of `letters10` and `toscrape1`. Valgrind testing has also been performed on edge cases as well as successful cases.