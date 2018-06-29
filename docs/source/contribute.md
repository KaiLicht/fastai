# How to contribute
_Jeremy’s notes on fastai coding style_

## Introduction

This is a brief discussion of fastai's coding style, which is loosely informed by (a much diluted version of) the ideas developed over the last 60 continuous years of development in the [APL](https://en.wikipedia.org/wiki/APL_\(programming_language\)) / [J](https://en.wikipedia.org/wiki/J_\(programming_language\)) / [K](https://en.wikipedia.org/wiki/K_\(programming_language\)) programming communities, along with Jeremy's personal experience contributing to programming language design and library development over the last 25 years. The style is particularly designed to be aligned with the needs of scientific programming and iterative, experimental development.

Everyone has strong opinions about coding style, except perhaps some very experienced coders, who have used many languages, who realize there's lots of different perfectly acceptable approaches. The python community has particularly strongly held views, on the whole. I suspect this is related to Python being a language targeted at beginners, and therefore there are a lot of users with limited experience in other languages; however this is just a guess. Anyway, I don't much mind what coding style you use when contributing to fastai, as long as:

- You don't change existing code to reduce its compliance with this style guide (especially: don't use an automatic linter / formatter!)
- You make some basic attempt to make your code not wildly different from the code that surrounds it.

Having said that, I do hope that you find the ideas in this style guide at least a little thought provoking, and that you consider adopting them to some extent when contributing to this library.

My personal approach to coding style is informed heavily by [Iverson's](https://en.wikipedia.org/wiki/Kenneth_E._Iverson) Turing Award (the "Nobel of computer science") lecture of 1979, [Notation as a Tool For Thought](http://www.eecg.toronto.edu/~jzhu/csc326/readings/iverson.pdf). If you can find the time, the paper is well worth reading and digesting carefully (it's one of the most important papers in computer science history), representing the development of an idea that found its first expression in the release of APL in 1964. Iverson says:

> The thesis of the present paper is that the advantages of executability and universality found in programming languages can be effectively combined, in a single coherent language, with the advantages offered by mathematical notation

One key idea in the paper is that "*brevity facilitates reasoning*", which has been incorporated into various guidelines such as "[shorten lines of communication](http://archive.vector.org.uk/art10009750)". This is sometimes incorrectly assumed to just mean 'terseness', but it is a much deeper idea, as described in [this Hacker News thread](https://news.ycombinator.com/item?id=13595729). I can't hope to summarize this thinking here, but I can point out a couple of key benefits:

- It supports [expository programming](http://vector.org.uk/art10000980), particularly when combined with the use of Jupyter Notebook or a similar tool designed for experimentation
- The most productive programmers I'm aware of in the world, such as the extraordinary [Arthur Whitney](https://en.wikipedia.org/wiki/Arthur_Whitney_\(computer_scientist\)) often use this coding style (which may or may not be a coincidence!)

## Style Guide

Python has over time incorporated a number of ideas that make it more amenable to this form of programming, such as:

- List, dictionary, generator, and set comprehensions
- Lambda functions
- Python 3.6 interpolated format strings
- Numpy array-based programming.

Although Python will always be more verbose than many languages, by using these features liberally, along with some simple rules of thumb, we can aim to keep all the key ideas for one semantic concept in a single screen on code. This is one of my main goals when programming&mdash;I find it very hard to understand a concept if I have to jump around the place to put the bits together. (Or as Arthur Whitney says "I hate scrolling"!)

### Symbol naming

- Follow standard Python casing guidelines (CamelCase classes, under\_score for most everything else).
- In general, aim for what Perl designer [Larry Wall](https://developers.slashdot.org/story/16/07/14/1349207/the-slashdot-interview-with-larry-wall) describes as metaphorically as *Huffman Coding*:

> In metaphorical honor of Huffman's compression code that assigns smaller numbers of bits to more common bytes. In terms of syntax, it simply means that commonly used things should be shorter, but you shouldn't waste short sequences on less common constructs.

- A fairly complete list of abbreviations is in [abbr.md](abbr.md); if you see anything missing, feel free to edit this file.
- For example, that in computer vision code, where we say 'size' and 'image' all the time, we use shortened forms `sz` and `img`. Or in NLP code, we would say `lm` instead of 'language model'
- Use `o` for an object in a comprehension, `i` for an index, and `k` and `v` for a key and value in a dictionary comprehension.
- Use `x` for a tensor input to an algorithm (e.g. layer, transform, etc), unless interoperating with a library where
  this isn't the expected behavior (e.g. if writing a pytorch loss function, use `input` and `target` as is standard
  for that library).
- Take a look at the naming conventions in the part of code you're working on, and try to stick with them. E.g. in
  `fastai.transforms` you'll see 'det' for 'deterministic', 'tfm' for 'transform', and 'coord' for coordinate.
- Assume the coder has knowledge of the domain in which you're working
  - For instance, use `kl_divergence` not `kullback_leibler_divergence`; or (like pytorch) use `nll` not `negative_log_liklihood`. If the coder doesn't know these terms, they will need to look them up in the docs anyway and learn the concepts; if they do know the terms, the abbreviations will be well understood
  - When implementing a paper, aim to follow the paper's nomenclature, unless it's inconsistent with other widely-used conventions. E.g. `conv1` not `first_convolutional_layer`

Although it's hard to design a really compelling experiment for this kind of thing, there is some [interesting research](https://www.sciencedirect.com/science/article/pii/S0167642309000343) supporting the idea that overly long symbol names negatively impact code comprehension.

### Layout

- Code should be less wide than the number of characters that fill a standard modern small-ish screen (currently 1600x1200) at 14pt font size. That means around 160 characters. Following this rule will mean very few people will need to scroll sideways to see your code. (If they're using a jupyter notebook theme that restricts their cell width, that's on them to fix!)
- One line of code should implement one complete idea, where possible
- Generally therefore an `if` part and its 1-line statement should be on one line, using `:` to separate
- Using the ternary operator `x = y if a else b` can help with this guideline
- If a 1-line function body comfortably fits on the same line as the `def` section, feel free to put them together with `:`
- If you've got a bunch of 1-line functions doing similar things, they don't need a blank line between them

```python
def det_lighting(b, c): return lambda x: lighting(x, b, c)
def det_rotate(deg): return lambda x: rotate_cv(x, deg)
def det_zoom(zoom): return lambda x: zoom_cv(x, zoom)
```

- Aim to align statement parts that are conceptually similar. It allows the reader to quickly see how they're
  different. E.g. in this code it's immediately clear that the two parts call the same code with different parameter
  orders.

```python
if self.store.stretch_dir==0: x = stretch_cv(x, self.store.stretch, 0)
else:                         x = stretch_cv(x, 0, self.store.stretch)
```

- Put all your class member initializers together using destructuring assignment. When doing so, use no spaces after
  the commas, but spaces around the equals sign, so that it's obvious where the LHS and RHS are.

```python
self.sz,self.denorm,self.norm,self.sz_y = sz,denorm,normalizer,sz_y
```

- Avoid using vertical space when possible, since vertical space means you can't see everything at a glance. For
  instance, prefer importing multiple modules on one line.

```python
import PIL, os, numpy as np, math, collections, threading
```

- Indent with 4 spaces. (In hindsight I wish I'd picked 2 spaces, like Google's style guide, but I don't feel like
  going back and changing everything...)
- When it comes to adding spaces around operators, try to follow notational conventions such that your code looks
  similar to domain specific notation. E.g. if using pathlib, don't add spaces around `/` since that's not how we write
  paths in a shell. In an equation, use spacing to lay out the separate parts of an equation so it's as similar to
  regular math layout as you can.
- Avoid trailing whitespace

### Algorithms

- fastai is designed to show off the best of what's possible. So try to ensure that your implementation of an
  algorithm is at least as fast, accurate, and concise as other versions that exist (if they do), and use a profiler to
  check for hotspots and optimize them as appropriate (if the code takes more than a second to run in practice).
- Try to ensure that your algorithm scales nicely; specifically, it should work in 16GB RAM on arbitrarily large
  datasets. That will generally mean using lazy data structures such as generators, and not pulling everything in to a
  list.
- Add a comment that provides the equation number from the paper that you're implementing in the appropriate part of
  the code.
- Use numpy/pytorch broadcasting, not loops, where possible.
- Use numpy/pytorch advanced indexing, not specialized indexing methods, where possible.
- Don't submit a PR that implements the latest hot paper until you've actually tried using it on a few datasets,
  compared it to existing approaches, and confirmed it's actually useful in practice! Ideally, include a notebook as a
  gist link with your PR showing these results.

### Other stuff

- Feel free to assume the latest version of python and key libraries is installed. But do mention in the PR and docs if
  you're relying on something that's only a couple of months old (including recently fixed bugs). Don't rely on any
  unreleased or beta versions however.
- Avoid comments unless they are necessary to tell the reader *why* you're doing something. To tell them *how* you're
  doing it, use symbol names and clear expository code.
- If you're implementing a paper or following some other external document, include a link to it in your code.
- If you're using nearly all the stuff provided by a module, just `import *`. There's no need to list all the things
  you are importing separately! To avoid exporting things which are really meant for internal use, define
  [`__all__`](https://stackoverflow.com/questions/44834/can-someone-explain-all-in-python). (As I write this, we're not
  currently following the `__all__` guideline, and welcome PRs to fix this.)
- Assume the user has a modern editor or IDE and knows how to use it. E.g. if they want to browse the methods and
  classes, they can use code folding - they don't need to rely on having two lines between classes. If they want to see
  the definition of a symbol they can jump to the reference/tag, then don't need a list of imports at the top of the
  file. And so forth...
- Don't use an automatic linter like autopep8 or formatter like yapf. No automatic tool can lay out your code with the care and domain understanding that you can. And it'll break all the care and domain understanding that previous contributors have used in that file!
- Keep your PRs small, and for anything controversial or tricky discuss it on [the forums](http://forums.fast.ai)
  first.
- When submitting a PR on a notebook, don't re-run the whole thing such that the diff ends up with changes for every
  bit of meta-data. Just change the bits of code you have to, and double-check the diff only contains those code
  changes before you push.

### Documentation

- We haven't figured out something we're happy with here yet. We're working on it...
- My ideal would be to have a decorator with a single line of documentation that links to a more detailed markdown doc.

## Fastai Abbreviation Guide
As mentioned in the [fastai style](https://github.com/fastai/fastai/blob/master/docs/style.md), 
we name symbols following the *Huffman Coding* principle, which basically means 

> Commonly used and generic concepts should be named shorter. You shouldn't waste short sequences on less common concepts. 

Fastai also follows the life-cycle naming principle:
> The shorter life a symbol, the shorter name it should have.

which means:
- **Aggressive Abbreviations** are used in *list comprehensions*, *lambda* functions, *local helper* functions.
- **Aggressive Abbreviations** are sometimes used for *local temporary variables* inside a function.
- **Common Abbreviations** are used most elsewhere, especially for *function arguments*, *function names*, and *variables*
- **Light or No Abbreviations** are used for *module names*, *class names* or *constructor methods*, since they basically live forever. 
However, when a class or module is very popular, we could consider using abbreviations to shorten its name.

This document lists abbreviations of common concepts that are consistently used across the whole fastai project. For naming of 
domain-specific concepts, you should check their corresponding module documentations. Concepts are grouped and listed by semantic order. 
Note that there are always exceptions, especially when we try to comply with the naming convention in a library.

|                  | **Concept**                         | **Abbr.**      | **Combination Examples**                         |
|------------------|-------------------------------------|----------------|--------------------------------------------------|
| **Suffix**       |                                     |                |                                                  |
|                  | multiple of something (plural)      | s              | xs, ys, tfms, args, ss                           |
|                  | internal property or method         | _              | data_, V_()                                      |
| **Prefix**       |                                     |                |                                                  |
|                  | check if satisfied                  | is_            | is_reg, is_multi, is_single, is_test             |
|                  | On/off a feature                    | use_           | use_bn                                           |
|                  | Number of something (plural)        | n_             | n_embs, n_factors, n_users, n_items              |
|                  | count something                     | num_           | num_features(), num_gpus()                       |
|                  | convert to something                | to_            | to_gpu(), to_cpu(), to_np()                      |
| **Infix**        |                                     |                |                                                  |
|                  | Convert between concepts            | 2              | name2idx(), label2idx(), seq2seq                 |
| **Aggressive**   |                                     |                |                                                  |
|                  | function                            | f              |                                                  |
|                  | input                               | x              |                                                  |
|                  | key                                 | k              |                                                  |
|                  | value                               | v              |                                                  |
|                  | index                               | i              |                                                  |
|                  | object                              | o              |                                                  |
|                  | variable                            | v              | V(), VV()                                        |
|                  | tensor                              | t              | T()                                              |
|                  | array                               | a              | A()                                              |
|                  | use first letter                    |                | *w*eight -> w, *m*odel -> m                      |
| **Generic**      |                                     |                |                                                  |
|                  | function                            | fn             | opt_fn, init_fn, reg_fn                          |
|                  | process                             | proc           | proc_col                                         |
|                  | transform                           | tfm            | tfm_y, TfmType                                   |
|                  | evaluate                            | eval           | eval()                                           |
|                  |                                     |                |                                                  |
|                  | argument                            | arg            |                                                  |
|                  | input                               | x              |                                                  |
|                  | input / output                      | io             |                                                  |
|                  | object                              | obj            |                                                  |
|                  | string                              | s              |                                                  |
|                  | class                               | cls            |                                                  |
|                  | source                              | src            |                                                  |
|                  | destination                         | dst            |                                                  |
|                  | directory                           | dir            |                                                  |
|                  | percentage                          | p              |                                                  |
|                  | ratio, proportion of something      | r              |                                                  |
|                  | count                               | cnt            |                                                  |
|                  |                                     |                |                                                  |
|                  | configuration                       | cfg            |                                                  |
|                  | random                              | rand           |                                                  |
|                  | utility                             | util           |                                                  |
|                  |                                     |                |                                                  |
|                  | threshold                           | thresh         |                                                  |
| **Data**         |                                     |                |                                                  |
|                  | number of elements                  | n              |                                                  |
|                  | number of batches                   | nb             |                                                  |
|                  | length                              | len            |                                                  |
|                  | size                                | sz             |                                                  |
|                  | array                               | arr            | label_arr                                        |
|                  | dictionary                          | dict           |                                                  |
|                  | sequence                            | seq            |                                                  |
|                  |                                     |                |                                                  |
|                  | dataset                             | ds             |                                                  |
|                  | dataloader                          | dl             |                                                  |
|                  | dataframe                           | df             |                                                  |
|                  | train                               | trn            | trn_ds                                           |
|                  | validation                          | val            | val_ds                                           |
|                  | number of classes                   | c              |                                                  |
|                  | batch                               | b              |                                                  |
|                  | batch size                          | bs             |                                                  |
|                  | multiple targets                    | multi          | is_multi                                         |
|                  | regression                          | reg            | is_reg                                           |
|                  | iterate, iterator                   | iter           | trn_iter, val_iter                               |
|                  |                                     |                |                                                  |
|                  | input                               | x              |                                                  |
|                  | target                              | y              |                                                  |
|                  | prediction                          | pred           |                                                  |
|                  | output                              | out            |                                                  |
|                  | column                              | col            |                                                  |
|                  | continuous                          | cont           | conts, cont_cols                                 |
|                  | category                            | cat            | cats, cat_cols                                   |
|                  |                                     |                |                                                  |
|                  | index                               | idx            |                                                  |
|                  | identity                            | id             |                                                  |
|                  | first element                       | head           |                                                  |
|                  | last element                        | tail           |                                                  |
|                  |                                     |                |                                                  |
|                  | unique                              | uniq           |                                                  |
|                  | residual                            | res            |                                                  |
|                  | label                               | lbl            | (not common)                                     |
|                  | augment                             | aug            |                                                  |
|                  | padding                             | pad            |                                                  |
|                  |                                     |                |                                                  |
|                  | probability                         | pr             |                                                  |
|                  | image                               | img            |                                                  |
|                  | rectangle                           | rect           |                                                  |
|                  | color                               | colr           |                                                  |
|                  | anchor box                          | anc            |                                                  |
|                  | bounding box                        | bb             |                                                  |
|                  |                                     |                |                                                  |
| **Modeling**     |                                     |                |                                                  |
|                  | initialize                          | init           |                                                  |
|                  | language model                      | lm             |                                                  |
|                  | recurrent neural network            | rnn            |                                                  |
|                  | convolutional neural network        | convnet        |                                                  |
|                  |                                     |                |                                                  |
|                  | model data                          | md             |                                                  |
|                  | linear                              | lin            |                                                  |
|                  | embedding                           | emb            |                                                  |
|                  | batch norm                          | bn             |                                                  |
|                  | dropout                             | drop           |                                                  |
|                  | fully connected                     | fc             |                                                  |
|                  | convolution                         | conv           |                                                  |
|                  | hidden                              | hid            |                                                  |
|                  |                                     |                |                                                  |
|                  | optimizer (e.g. Adam)               | opt            |                                                  |
|                  | layer group learning rate optimizer | layer_opt      |                                                  |
|                  | criteria                            | crit           |                                                  |
|                  | weight decay                        | wd             |                                                  |
|                  | momentum                            | mom            |                                                  |
|                  | cross validation                    | cv             |                                                  |
|                  | learning rate                       | lr             |                                                  |
|                  | schedule                            | sched          |                                                  |
|                  | cycle length                        | cl             |                                                  |
|                  | multiplier                          | mult           |                                                  |
|                  | activation                          | actn           |                                                  |
|                  |                                     |                |                                                  |
| **CV**           | _computer vision_                   |                |                                                  |
|                  | figure                              | fig            |                                                  |
|                  | image                               | im             |                                                  |
|                  | transform image using opencv        | _cv            | zoom_cv(), rotate_cv(), stretch_cv()             |
| **NLP**          | _natural language processing (nlp)_ |                |                                                  |
|                  | token                               | tok            |                                                  |
|                  | sequence length                     | sl             |                                                  |
|                  | back propagation through time       | bptt           |                                                  |


## Module Decisions

There are many ways of doing one thing in programming. Instead of getting into debates about the one right way of doing things, in `fastai` library we would like to make decisions and then stick with them. This page is to list down any such decisions made.

### Image Data
- Coordinates
 - Computer vision uses coordinates in format `(x, y)`. e.g. PIL
 - Maths uses `(y, x)`. e.g. Numpy, PyTorch
 - `fastai` will use `(y, x)`
- Bounding Boxes
 - Will use `(coordinates top right, coordinates bottom right)` instead of `(coordinates top right, (height, width))`

## Testing Style

We chose pytest as a framework since it's more modern, concise, and [recommended by coders](https://www.slant.co/topics/2621/~python-unit-testing-frameworks).

We also try to follow this suggestion from the [python testing guide](http://docs.python-guide.org/en/latest/writing/tests/):

_Use long and descriptive names for testing functions. The style guide here is slightly different than that of running code, where short names are often preferred. The reason is testing functions are never called explicitly. square() or even sqr() is ok in running code, but in testing code you would have names such as test_square_of_number_2(), test_square_negative_number(). These function names are displayed when a test fails, and should be as descriptive as possible._

More generally, aim to write tests that also explain the code they are testing. A really good test suite can also serve as really good documentation.

### Testing patterns

* Do not use mock or fake objects. The library is nice enough that real versions of required objects can be used without prohibitive overhead.
* Keep test methods small and tidy, just like any other code.
* Aim to add a regression test as part of any bug fix PR.
* Add tests before refactoring, so they can help prove correctness.


## FAQ

<dl>
  <dt>Why not use PEP 8?</dt><dd>I don't think it's ideal for the style of programming that we use, or for math-heavy code. If you've never used anything except PEP 8, here's a chance to experiment and learn something new!</dd>
  <dt>My editor is complaining about PEP 8 violations in fastai; what should I do?</dt><dd>Pretty much all editors have the ability to disable linting for a project; figure out how to do that in your editor.</dd>
  <dt>Are you worried that using a different style guide might put off new contributors?</dt><dd>Not really. We're really not that fussy about style, so we won't be rejecting PRs that aren't formatted according to this document. And whilst there are people around who are so closed-minded that they can't handle new things, they're certainly not the kind of people we want to be working with!</dd>
  <dt></dt><dd></dd>
</dl>

