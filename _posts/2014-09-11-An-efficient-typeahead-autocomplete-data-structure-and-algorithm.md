---
layout: post
title: An efficient typeahead/autocomplete data structure and algorithm
tags: [Algorithms]
---

I've been wanting to implement a typeahead lookup for universities and thought it would be an interesting weekend project to write an efficient typeahead lookup dictionary structure.

It turns out that quite a lot of thought needs to be put into this in order for it to work efficiently.

## Initial Considerations

I looked at quite a few different algorithms that might solve this problem. There are quite a few directions that a typeahead dictionary could take and it really depends on its intended use. These are the key considerations that I looked at and a discussion of each follows.

* Concurrent read/write.
* Objects with multiple names to match against (i.e., a university has one canonical name and alternate names or abbreviations).
* Input matching of the user's input (the partial string) against the dictionary of typeahead strings: matching the input as a prefix of each typeahead string, matching the input as a substring of each typeahead string, matching the input as a prefix of each word in each typeahead string, or performing fuzzy string matching.
* Input filters on geographical and other arbitrary criteria.
* Ordering and limiting the resultset.
* Try to make the dictionary interface generic enough that different data structures could be used for future requirements.
* String preprocessing.

Let's break each of these down.

### Concurrent Read/Write

In order to focus more on the actual algorithm and structure rather than concurrency, my initial solution does not support concurrent read/write operations and must be initialized once at runtime. However, it would be rather straightforward to wrap the dictionary and have a read/write lock that prevents read/write access when a write is in progress and prevents write access when one or more reads are occurring.

If it ends up being unacceptable to block reads while a write is occurring, an alternate solution is to rebuild the dictionary in memory and then switch the reference from the old to the new dictionary when the new one is ready. However, that would be very costly if we are only looking to insert or remove a single typeahead string. A tree-like search structure might also support locks at specific nodes in the tree where a write is taking place so that mutually exclusive subtrees could be written concurrently and reads only block at the topmost node where a write is in progress.

### Canonical + Alternate Names

Universities often have acronyms and it's reasonable to expect that typing "SFU" would correctly offer "Simon Fraser University" as a typeahead suggestion. For this reason, I defined an interface `StringSearchable` that any object can extend and contains one method that returns a collection of the various spellings for the object (in this case, a university). It's perfectly acceptable for two objects to contain the same typeahead strings.

```java
public interface StringSearchable extends Serializable {
    /**
     * Returns a list of strings that identify this object.
     */
    public Set<String> getSearchStrings();
}
```

I can now define a University class that implements StringSearchable and returns the appropriate spelling variations. Ultimately, a call to the dictionary to find a string fragment returns a list of matching `StringSearchable` instances that were provided when the dictionary was initialized.

### Input Matching

This is the area where I spent the most time thinking and planning, as the intended use of the typeahead has a huge impact on the underlying data structure.

[BK-Trees](http://nullwords.wordpress.com/2013/03/13/the-bk-tree-a-data-structure-for-spell-checking/) are good structures for fuzzy match searches. A [BK-Tree](http://blog.notdot.net/2007/4/Damn-Cool-Algorithms-Part-1-BK-Trees) is a tree data structure containing words in each node, making use of Levenshtein edit distances between each node. Lookups are pretty efficient and this is a good data structure for spell checkers but breaks down in a typeahead. This would only support fuzzy matches on entire words and the dictionary will contain strings with each consisting of many words.

To support substring searches anywhere in the string, [Rabin-Karp](http://www-igm.univ-mlv.fr/~lecroq/string/node5.html) (also [here](http://courses.csail.mit.edu/6.006/spring11/rec/rec06.pdf)) is a good starting point to efficiently compute string hashes. I would envision taking every typeahead string and calculating the hash value for every substring defined as index `i` in the string and length `n` for `i` in `[0..length(s)-1]` and `n` in `[i..length(s)-1]`. This massive list of hash values could consequently be stored in a hash table that maps the hash values to source strings (and possibly indexes in the strings). Lookups would be really fast at the cost of memory.

String prefix matching (either at the beginning of each typeahead string or the beginning of each word of each typeahead string) are both good candidates for tree structures. A [trie](https://www.cs.bu.edu/teaching/c/tree/trie/) is a good choice here, but it also carries a huge memory cost and isn't very realistic for the size of real-world datasets. A [ternary search tree](http://www.drdobbs.com/database/ternary-search-trees/184410528) is a better solution.

In my use case of universities, it doesn't really make sense for someone to search for a word in the middle of the university's name. In my previous example, someone searching for "Simon Fraser University" might start typing "Simon," but certainly wouldn't type "Fraser." For this reason, I chose to optimize for prefix lookups against a large number of strings by using a ternary search tree with some custom modifications.

### Filters

Since universities exist across the world, if I know the user is in a specific country and is only interested in universities within that country, it would be nice to provide a filter mechanism to omit matches based on criteria defined at runtime. It would be simple to create a separate dictionary structure per country, but this doesn't scale well and wouldn't solve my problem if I later wanted to break it down further into regions within a country or perhaps filter universities within a specific distance of the user's current location.

### Ordering and Limits

When a typeahead search is performed, I'd like the ability to rank each match based on custom criteria for the purpose of ordering. I might prioritize matches that are closer to a user's location by ranking them higher, or perhaps rank universities higher based on how users interact with them.

An efficient way to support ordering and limits is to collect the results in a [min-heap](http://www.cs.cmu.edu/~tcortina/15-121sp10/Unit06B.pdf) with a maximum size. Each time a match is found, it gets ranked and added to the min-heap, evicting the lowest-ranked match if its rank is equal to or exceeds the highest-ranked match in the min-heap. This offers O(log n) insertion time and O(n) retrieval time. Retrieval time is O(n) because although a min-heap is nearly sorted, it still requires a sorting step and an [insertion sort](http://en.wikipedia.org/wiki/Insertion_sort) generally performs with O(n) complexity on an almost-sorted list.

### Generic Interface

I've made design decisions based on my specific use case, but perhaps in the future I'd like to optimize my typeahead for other purposes that would require a completely different data structure. Defining a common interface means I can write my code without really worrying about the underlying implementation and I could swap different dictionary implementations around if the need arose.

The interface at its simplest would look something like this.

```java
public interface Dictionary {
    public void addSearchable(StringSearchable searchable);
 
    public List<StringSearchable> searchExactMatch(
        String searchString,
        DictionarySearchResultFilter filter,
        int limit);
 
    public List<StringSearchable> searchPrefix(
        String searchPrefix,
        DictionarySearchResultFilter filter,
        int limit);
}
```

### String Preprocessing

Although fuzzy matching isn't supported, it would still make sense to strip off all whitespace, punctuation, and accents from a typeahead string and normalize it to lower or uppercase for the purpose of lookups. This allows us to have a name like "SFU, Burnaby Campus" but support a partial string of "sfu burnaby." It also makes support of non-English names such as "Université de Montréal" possible by permitting input of "universite" without the accent. As long as the same string preprocessing step is applied to the typeahead strings and the search strings, we can support quite a bit of variation and still find good matches.

### Summary

The TL;DR of all the above is that my data structure must support:

* Filters are applied to matches, specifying inclusion and ranking instructions.
* The dictionary must support searchable objects, each with canonical and alternate names.
The optimal design choice is a ternary search tree with some optimizations discussed further below and a min-heap with a maximum size to aggregate the results.

## The Design

### Ternary Search Tree

Each node in the tree must have a reference to a collection of searchable objects terminating at that node (or a null reference if no objects terminate at the node). It also contains references to three children: left, middle, and right. The left and right children are traversed if the desired character is less than or greater than the one at the current node and the child is traversed when the node's character matches the desired character. Let's examine a simple example TST structure for three strings:

```
sfu
sfu vancouver
simon fraser university

        s
        |
        f
        |\
       *u i
        | |
        v m
        | |
        a o
        . .
        . .
        . .
       *r y*
```

I've marked the terminating nodes with asterisks. This is a good start, but the real advantage of a TST is the lookup speed when nodes contain many left and right children. From the above example, once we've traversed past the second and third nodes, we could represent the remaining string using just one node each. This saves quite a bit of memory in a real-life example with thousands of typeahead strings. An optimized structure would look as follows.

```
        s
        |
        f
        |\
       *u imonfraseruniversity*
        |
        vancouver*
```

This removes a lot of objects and memory references, which will add up when the structure is filled up with real data.

### Bounded Min-Heap

Java provides a [PriorityQueue](http://docs.oracle.com/javase/8/docs/api/java/util/PriorityQueue.html) and we can build off this quite easily to provide the necessary functionality. Google's guava library also provides a [MinMaxPriorityQueue](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/MinMaxPriorityQueue.html), but it doesn't provide a way to retrieve the nodes in sorted order in as efficient a way and also acts as a min-max heap, which is a bit more than what is needed here.

The code for this bounded min-heap is quite concise and can be reviewed on my github page at [BoundedMinHeap](https://github.com/pushinginertia/pushinginertia-commons/blob/master/pushinginertia-commons-collect/src/main/java/com/pushinginertia/commons/collect/BoundedMinHeap.java).

## Final Code

The entire package of classes implementing this functionality can be found on GitHub at [com.pushinginertia.commons.collect.typeahead](https://github.com/pushinginertia/pushinginertia-commons/tree/master/pushinginertia-commons-collect/src/main/java/com/pushinginertia/commons/collect/typeahead).

The unit test [TstDictionaryTest](https://github.com/pushinginertia/pushinginertia-commons/blob/master/pushinginertia-commons-collect/src/test/java/com/pushinginertia/commons/collect/typeahead/TstDictionaryTest.java) provides a good demonstration of its use.

