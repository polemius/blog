---
title: How I reduce the size of the library with genetic algorithm
date: "2019-10-14T20:00:21"
description: The blog post about using genetic algorithm to reduce the size of library
---

**TL; DR** I've decreased the size of [nanoid](https://github.com/ai/nanoid) by 1 byte using a genetic algorithm.

[Nanoid](https://github.com/ai/nanoid) is a tiny (139 bytes) string ID generator for JavaScript.

The server sends to browsers gzipped files, so if we can optimize the library's code for gzip algorithm then the amount of transferred data would be lower.

The size of this library contains the code itself of course and the *alphabet* to get the symbols.


If we look in [git history](https://github.com/ai/nanoid/commit/395e57f83d0bf565d0daf5b012a61464e120a1cb#diff-07fa9a7ce00e46c7d975483d790d7cee) of nanoid library we can see that the first commit has this string:

```javascript
module.exports =
    '_~0123456789' +
    'abcdefghijklmnopqrstuvwxyz' +
    'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
``` 

If we check the size of the library using [size-limit](http://github.com/ai/size-limit) than we get that this simple alphabet weight is 73 bytes.

The lates (2.1.6) version of nanoid has *alphabet* looking like this:

```javascript
module.exports =
    'ModuleSymbhasOwnPr-0123456789ABCDEFGHIJKLNQRTUVWXYZ_cfgijkpqtvxz' 
```

You can see that this string contains a word like *Module, Symb, has, Own*. Because the code contains these words and gzip can pack *url.js* in more efficient way (only 64 bytes).

In one of the issues on github repository of nanoid, I've read that [genetic algoritm](https://en.wikipedia.org/wiki/Genetic_algorithm) can help to find the best string that can be packed as much as possible. And I will try to do it.

I've used the library [geneticalgorithm](https://github.com/panchishin/geneticalgorithm). This library needs to define 3 functions:  function to mutate *chromosome*, function to crossover *chromosomes* and function to check how good *chromosome* is.

I've started with a fitness function. This function has one input parameter and returns the number:

```javascript
function fitnessFunction (phenotype) {
    const file = js.replace(/[A-Za-z0-9-_]{30,}/, phenotype.alphabet)
    const size = gzipSize.sync(file)

    return -1 * size
}
```

To check the size I've used [gzip-size](https://github.com/sindresorhus/gzip-size) library.

After that I've defined a function to mutate chromosome:

```javascript
function mutationFunction (phenotype) {
    const i = Math.floor(Math.random() * phenotype.alphabet)
    const j = Math.floor(Math.random() * phenotype.alphabet)
    
    return {
        alphabet: swapChars(alphabetTest, i, j)
    }
}

function swapChars (str, index1, index2) {
    let l = index1 < index2 ? index1 : index2
    let h = index1 > index2 ? index1 : index2
    return str.substring(0, l) +
        str[h] +
        str.substring(l + 1, h) +
        str[l] +
        str.substring(h + 1, str.length)
}
```

And also the crossover function:

```javascript
function crossoverFunction (phenotypeA, phenotypeB) {
    const alphabetA = phenotypeA.alphabet
    const alphabetB = phenotypeB.alphabet
    const indexA =
        Math.floor(Math.random() * alphabetA.length / 2 + alphabetA.length / 2)
    const indexB =
        Math.floor(Math.random() + alphabetA.length / 2)
    const newStrA = alphabetA.substring(indexA, alphabetA.length)
    const newStrB = alphabetB.substring(0, indexB)

    return [
        { alphabet: addMissingCharacter(newStrA, alphabetB) },
        { alphabet: addMissingCharacter(newStrB, alphabetA) }
    ]
}

function addMissingCharacter (str, proto) {
    let newStr = str
    for (let i = 0; i < proto.length; i++) {
        if (str.indexOf(proto[i]) === -1) {
            newStr += proto[i]
        }
    }
    return newStr
}
```

I've started from the population size of 1000 and the 500 generations. And I get another alphabet string but the size was the same. After that I've increased the population size to 10000 and 1000 generations and after I wait a while I get this string:

```javascript
RAHVfgFctiUEv1z0_KSymbhasOwnPr69GqYTJk2L47xpZXIDjQBW3C-8N5Module 
```

How you can see this string also contains some words but lighter on 1 byte.

Size limit show that `url.js` is only 63 bytes.

After I get this result I was trying to *normalize* this string a little. I've moved all words to the start of the string and trying symbol by symbol moved all characters in alphabetic order. And here what I got:

```javascript
ModuleSymbhasOwnPr-0123456789ABCDEFGHNRVfgctiUvz_KqYTJkLxpZXIjQW
```

I know that ain't much but with 3 simple functions and a half an hour I managed to find a better solution to decrease the size.

All code you can find in my [pull request](https://github.com/ai/nanoid/pulls). Actually, you can run this code and maybe you will find a better string that I've found.

Thanks for reading.