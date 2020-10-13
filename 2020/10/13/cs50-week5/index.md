# CS50 2020 Week5: Data Structures Problem Answers

<!--more-->
## [Speller](https://cs50.harvard.edu/x/2020/psets/5/speller)

[Big Board](https://speller.cs50.net/cs50/problems/2020/x/challenges/speller#user/jasonj27)

### Reduce size time
* Count words in dictionary while load dictionary

### Reduce load time, unload time, heap usage, stack usage
* Use static variable or gloabl variable

### Hash function
[murmurhash](https://github.com/jwerle/murmurhash.c)

### dictionary.c
```c
// Implements a dictionary's functionality

#include <stdbool.h>
#include <stdio.h>
#include <stdlib.h>
#include <ctype.h>
#include <string.h>
#include "dictionary.h"

// Represents a node in a hash table
typedef struct node
{
    char word[LENGTH + 1];
    struct node *next;
}
node;

// Number of buckets in hash table
const unsigned int N = 65536;

// Hash table
node *table[N];

unsigned int word_count = 0;

///

uint32_t
murmurhash (const char *key, uint32_t len, uint32_t seed) {
  uint32_t c1 = 0xcc9e2d51;
  uint32_t c2 = 0x1b873593;
  uint32_t r1 = 15;
  uint32_t r2 = 13;
  uint32_t m = 5;
  uint32_t n = 0xe6546b64;
  uint32_t h = 0;
  uint32_t k = 0;
  uint8_t *d = (uint8_t *) key; // 32 bit extract from `key'
  const uint32_t *chunks = NULL;
  const uint8_t *tail = NULL; // tail - last 8 bytes
  int i = 0;
  int l = len / 4; // chunk length

  h = seed;

  chunks = (const uint32_t *) (d + l * 4); // body
  tail = (const uint8_t *) (d + l * 4); // last 8 byte chunk of `key'

  // for each 4 byte chunk of `key'
  for (i = -l; i != 0; ++i) {
    // next 4 byte chunk of `key'
    k = chunks[i];

    // encode next 4 byte chunk of `key'
    k *= c1;
    k = (k << r1) | (k >> (32 - r1));
    k *= c2;

    // append to hash
    h ^= k;
    h = (h << r2) | (h >> (32 - r2));
    h = h * m + n;
  }

  k = 0;

  // remainder
  switch (len & 3) { // `len % 4'
    case 3: k ^= (tail[2] << 16);
    case 2: k ^= (tail[1] << 8);

    case 1:
      k ^= tail[0];
      k *= c1;
      k = (k << r1) | (k >> (32 - r1));
      k *= c2;
      h ^= k;
  }

  h ^= len;

  h ^= (h >> 16);
  h *= 0x85ebca6b;
  h ^= (h >> 13);
  h *= 0xc2b2ae35;
  h ^= (h >> 16);

  return h;
}

///

// Returns true if word is in dictionary else false
bool check(const char *word)
{
    char word_lower[LENGTH + 1];
    int word_len = strlen(word) + 1;
    for (int i = 0; i < word_len; i++)
    {
        word_lower[i] = tolower(word[i]);
    }

    unsigned int word_hash = hash(word_lower);

    if (table[word_hash] == NULL)
    {
        return false;
    }
    else if (strcmp(table[word_hash]->word, word_lower) == 0)
    {
        return true;
    }
    else
    {
        for (node *tmp = table[word_hash]->next; tmp != NULL; tmp = tmp->next)
        {
            if (strcmp(tmp->word, word_lower) == 0)
            {
                return true;
            }
        }
    }
    return false;
}

// Hashes word to a number
unsigned int hash(const char *word)
{
    return murmurhash(word, strlen(word), 0) >> 16;
}

// Loads dictionary into memory, returning true if successful else false
bool load(const char *dictionary)
{
    static node nodes[200000];
    memset(nodes, 0, sizeof(node) * 200000);
    char buff[LENGTH + 1] = {'\0'};
    FILE *fp;

    fp = fopen(dictionary, "r");
    if (fp == NULL)
    {
        return false;
    }
    else
    {
        while (fscanf(fp, "%s", buff) != EOF)
        {
            unsigned int buff_hash = hash(buff);

            if (table[buff_hash] == NULL)
            {
                table[buff_hash] = &nodes[word_count];
                memcpy(table[buff_hash]->word, buff, LENGTH + 1);
                table[buff_hash]->next = NULL;
                word_count++;
            }
            else
            {
                node *n = &nodes[word_count];
                memcpy(n->word, buff, LENGTH + 1);
                n->next = table[buff_hash];
                table[buff_hash] = n;
                word_count++;
            }
        }
        fclose(fp);
        return true;
    }
    return false;
}

// Returns number of words in dictionary if loaded else 0 if not yet loaded
unsigned int size(void)
{
    return word_count;
}

// Unloads dictionary from memory, returning true if successful else false
bool unload(void)
{
    return true;
}

```

### dictionary.h
```c
// Declares a dictionary's functionality

#ifndef DICTIONARY_H
#define DICTIONARY_H

#include <stdbool.h>

// Maximum length for a word
// (e.g., pneumonoultramicroscopicsilicovolcanoconiosis)
#define LENGTH 45

// Prototypes
bool check(const char *word);
unsigned int hash(const char *word);
bool load(const char *dictionary);
unsigned int size(void);
bool unload(void);

#endif // DICTIONARY_H

/**
 * `murmurhash.h' - murmurhash
 *
 * copyright (c) 2014 joseph werle <joseph.werle@gmail.com>
 */

#ifndef MURMURHASH_H
#define MURMURHASH_H 1

#include <stdint.h>

#define MURMURHASH_VERSION "0.0.3"

#ifdef __cplusplus
extern "C" {
#endif

/**
 * Returns a murmur hash of `key' based on `seed'
 * using the MurmurHash3 algorithm
 */

uint32_t
murmurhash (const char *, uint32_t, uint32_t);

#ifdef __cplusplus
}
#endif

#endif
```
