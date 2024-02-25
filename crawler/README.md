## USING EXTENSION! Palmer granted me a bit of grace on submitting it after 10pm too as I have been in office hours trying to solve my memory leaks since noon. You can reach out to him for confirmation.

# CS50 TSE: Crawler
## CS50 Spring 2023

### crawler

The `crawler` scans a `webpage` to find all of the webpages mentioned within it and leading to another webpage within the same domain.


### Usage

The `crawler` can be used for the TSE as it will allow the user to search at a given `depth` for another webpage while storing its `depth`, and its `HTML` containing its contents.

```c
void parseArgs(const int argc, char* argv[], char** seedURL, char** pageDirectory, int* maxDepth)
void crawl(char* seedURL, char* pageDirectory, const int maxDepth) void hashtable_print(hashtable_t* ht, FILE* fp, void (*itemprint)(FILE* fp, const char* key, void* item));
void pageScan(webpage_t* page, bag_t* pagesToCrawl, hashtable_t* pagesSeen)void hashtable_delete(hashtable_t* ht, void (*itemdelete)(void* item) );
```

### Implementation

We implement this crawler as a combination of a `struct hashtable` and a `struct bag`.

The hashtable stores the webpages that have already been visited to ensure that nothing is referenced twice.

The bag stores the webpages that are referenced within a certain webpage to be revisited later.

The `parseArgs` method ensures that to insert a new webpage, the user implements the function correctly by checking the number of arguments and the types of variables used.

The `crawl` method iterates through each webpage's mentioned webpages and selects those which have not yet been visited and that are within a certain depth to be added to the hash.

The `pageScan` method scans each page for the other webpages which are mentioned within it.

### Assumptions

For some reason, I get memory leaks when a URL has any capitalized character. I tried using normalizeURL to limit this, but unfortunately it only made the memory leaks worse.

`testing.out` does not demonstrate the completed `toscrape` at depth of 1 test because of its runtime, so I left it running for about 20 minutes and the output is what was produced.

### Files

* `Makefile` - compilation procedure
* `crawler.c` - the implementation
* `testing.out` - result of `make test &> testing.out` 
* `testing.sh` - file to derive `testing.out`


### Compilation

To compile, simply `make -f Makefile` while inside the `crawler` directory.

### Testing

The `testing.sh` program tests the edge and failure cases of `crawler.c` and returns the output to `testing.out`.

To test, simply `make test`.
See `testing.out` for details of testing and an example test run. To do this, run `make test &> testing.out`.

To test with valgrind, `make valgrind`.