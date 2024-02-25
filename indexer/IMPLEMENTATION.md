## CS50 TSE Indexer
# Implementation Spec

# Data structures

We use a number of data structures between the indexer and its shared functions in the `common` directory. This includes an `index`, which implements a `hashtable` of `counters` to store a word with the corresponding docIDs where it appears and its number of appearances in each file. We continue to use the `webpage` data structure as introduced during `crawler`.

# Control flow

The Indexer is implemented in one file `indexer.c`, with three functions.

**main**

The main function ensures that all of the inputs meet the expectations specified in the *Requirements Spec* and calls *indexBuild* and *indexPage*, then exits zero.


**indexBuild**

Given arguments extracted from the command line during the *main* function, choose a directory that will be referenced for the creation of the `index`. This will be in the form of a `webpage` with all of its information.

Pseudocode:
```c
initialize the file
initialize the relevant webpage information
create the name of the file corresponding to each ID
while more files remain in the directory
	create a webpage with the information in the file
	pass the webpage to indexPage
	increment to the next docID
	extrapolate the parameters for the new file
	close the original file
delete the counters
delete the hashtable
delete the index
```

**indexPage**
This function takes the information provided by the *indexBuild* function to determine which words will be incremented in the index based on their size and which ones will not. Each webpage needs to be verified to have an HTML to seek before continuing.

Pseudocode:
```c
while there is an HTML in the page being searched
	if the word found in that HTML is of size greater than or equal to 3
		normalize the word to be only lower case
        pass the word to the index_save function to be stored
	free the word
```

**Other modules**

*pagedir*

We reuse the `pagedir` file that was created for the crawler. In the indexer, we add two methods: `pagedir_validate` and `pagedir_load`. Detailed descriptions of these functions can be found in the `pagedir.h` file.

*word*

This function is responsible for normalizing every word that is found through the process of running `indexPage`. Turning words to lower case words is its only purpose.

*index*

This function defines the `index` data structure. Using it, we will be able to call a number of methods such as `index_save` which will save each word within the counters corresponding to its docID and `index_load` which can turn a file that was created by an indexer into another `index` data structure.

# Function prototypes

**indexer**
```c
int main(const int argc, char* argv[]);
void indexBuild(char* pageDirectory, char* outputFilename);
void indexPage(index_t* index, webpage_t* page, int docID);
```

**pagedir**
```c
bool pagedir_init(const char* pageDirectory);
void pagedir_save(const webpage_t* page, const char* pageDirectory, const int docID);
bool pagedir_validate(char* directory);
webpage_t* pagedir_load(char* filename, char* URL);
```

**word**
```c
char* normalizeWord(char* word);
```

**index**
```c
index_t* index_new(void);
hashtable_t* indextable_get(index_t* index);
void index_save(index_t* index, char* word, int docID);
void index_delete(index_t* index);
void index_print(index_t* index, FILE *fp);
void index_delete_helper(void* item);
void index_print_helper(void* arg, const char* key, void* item);
void counters_iterate_helper(void* arg, const int key, int item);
void index_load(char* oldIndexFilename, char* newIndexFilename);
```

# Error handling and recovery

All the command-line parameters are rigorously checked before any data structures are allocated or work begins; problems result in a message printed to stderr and a non-zero exit status.

Out-of-memory errors are handled through the use of `mem_malloc` and `free` or various data structure `delete` functions.

All code uses defensive-programming tactics to catch and exit as seen in the `main` function of `indexer.c`; e.g., if a function receives bad parameters.

That said, certain errors are caught and handled internally: for example, `index.c` returns an error if the user attempts to place a value within an index data structure that has not yet been initialized.

# Testing plan

Here is an implementation-specific testing plan.

*Unit testing*

There are a few units (`indexer`, `pagedir`, and `index`, and `word`). The `indexer` represents the whole system, the `pagedir` represents functions that handle more database-side issues such as storage of the outputs. `word` is only responsible for normalizing words, and `index` is the data structure that contains a `hashtable` that is used throughout the indexer program. Each item should be individually tested before running a system test.

*Integration/system testing*

The `indexer`, unlike the `crawler`, runs faster with longer text files. As a result, longer files should be tested to prove that testing was thorough. Similar to the `crawler`, the `indexer` will take a `testing.sh` file that prints to `testing.out` a number of edge cases, errors, and successful cases' outputs to ensure proper functionality.

Samples of successful tests include the result of `letters0`, `letters1`, `letters2`, `letters3`, and `toscrape0`.