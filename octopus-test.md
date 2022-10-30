---
title: 'Understanding form and the Octopus test'
date: '2021-09-21'
---

I recently listened to an episode of the 
[Gradient Dissent (the Weights & Biases podcast) with Emily Bender](https://wandb.ai/wandb_fc/gradient-dissent/reports/Emily-M-Bender-Professor-at-UW-Language-Models-and-Linguistics--Vmlldzo4ODY0NDE?galleryTag=gradient-dissent)
in which they discussed Language Models (LM) and the dangers arising from making increasingly larger ones. The discussion was primarily centered around
the paper [On the Dangers of Stochastic Parrots: Can Language Models Be Too Big? 🦜](https://dl.acm.org/doi/10.1145/3442188.3445922). They also discuss some ideas from
[Climbing towards NLU:
On Meaning, Form, and Understanding in the Age of Data](https://aclanthology.org/2020.acl-main.463/) (E. Bender & A. Koller, 2020).
Here I wanted to present a few of the ideas discussed in the podcast episode and the referenced papers.
## Meaning, form and understanding

My favourite quote from the Bender & Koller[^1] paper is the following,

> _Meaning cannot be learned from form alone_ - E. Bender, A. Koller (2020)

Everytime a new larger language model is released, there appears new articles claiming
that the model 'understands' language. The term 'understanding' is quite the loaded term, and
needs to be carefully examined before claiming that a machine learning model possesses it.
Because language models are trained on text (_form_), it is not possible for the language model to learn _meaning_
from the data in the same way as a person would learn from it. It can learn to
associate inputs to outputs, and learn to pick out clusters of words that are associated with each other - but that is
not the same as _understanding the meaning_.

Getting an accurate and consistent definition of 'meaning' and 'knowledge' is surprisingly difficult.
Here are some useful definitions from Bender & Koller[^1]

**Form** - Any observable realization of language. For example, marks/words on a page, pixels in an image, movements of hands in sign language or the dashes and dots of Morse code.

**Communicative Intent** is grounded in the real world that the speaker and listener inhabit together.

There are many types of communicative intents: they may be to convey some information
to the other person; or to ask them to do something;
or simply to socialize.

**Conventional Meaning** of an expression is what is constant across all of it's possible context of use. Conventional meaning is an abstract object that represents the communicative potential of a form, given
the linguistic system it is drawn from. 

**Meaning** is the relation between form and communicative intent

So _meaning_ is made up out of two things, an expression of language (_form_)
and the communicative intent which may be independent of the form. 

Then to _understand_ something is the process of retrieving the _communicative intent_
given the _form_. 

So in a conversation between two people, the speaker has a certain communicative intent $i$, and chooses an expression $e$ with a
standing (conventional) meaning $s$ which is fit to express $i$ in the
current communicative situation. Upon hearing $e$,
the listener then reconstructs $s$ and uses their own
knowledge of the communicative situation and their
hypotheses about the speaker’s state of mind and
intention in an attempt to deduce $i$.


It is often impossible to say something that doesn't require knowledge/understanding about the world. Let's look at an example,

- _Person A and B are sat on opposite sides of a table. There is a clock on the wall behind person A_. 
- "**The clock says that it is almost 10pm**", says A.
- _Person A and B both get up and go to the next room to watch TV_

Person A had the _communicative intent_ of wanting to say what the time was, because A 
knew that their favourite TV show starts at 22.00, so they had better go to the TV room. 
Person A then chose the expression (the _form_) "the clock says that it is almost 10pm" which they feel
has the conventional meaning of telling the time and is enough to express their
communicative intent.

Upon hearing this expression, person B then begins the task of reconstructing
the conventional meaning of the _form_, and then use their knowledge of
the world[^note] to deduce that person A's communicative intent was that they should go to the TV room because it is
almost 22.00[^2].

Given only the expression that _A_ used, a language model would not have much hope
to really _understand_ what the intent was.

It's also interesting to think about how we know that the 'clock' is almost certainly not a thing that can speak, nor do we expect it 'say' anything else other than 
the correct time (let's assume that it is a magical clock that is never wrong). How do we know these things?
We just know that the idea of a talking clock is nonsense, and only
happens in sci-fi stories.


## The Octopus test

An interesting thought experiment that Bender and Koller discuss in their paper is a test they call the _octopus test_, which illustrates the
difficulty in learning from _form_ alone.
Imagine that person _A_ and _B_ are independently stranded on two deserted islands, but
they can communicate with each other via an underwater cable and often send
text messages in english to each other.

Without either person A or B's knowledge another entity O (a very clever octopus) who cannot speak english but has a very
advanced knowledge of statistics and pattern matching.

After some very long time, O decides to cut the wire so that they can 
speak directly to each person. The question is, could O have _learned_ enough
from the _form_ (the text messages) so that neither person knows that anything has changed? 

The answer depends on the nature of the task that O is given. If person A and B have
been using the text messages for small talk (i.e, a _communicative intent_ of 'socialising'),
then O would likely have learned enough from the _form_ to be able to 
continue this small talk. But this is not the same as O _understanding_ what
A is talking about, it just means that A is doing the mental work of attributing
meaning to the responses of O.

If person A suddenly needs advice about how to construct a weapon to
defend itself from a bear, then there is no chance that
O could give a helpful response unless it has some external knowledge about
the theory of crafting weapons from coconuts and sticks.

## Stochastic Parrots

Another important aspect of language models that needs to be understood is
that the data these models are trained on can often harbour significant biases.
For example, the Switch-C[^3] transformer model was trained Colossal Clean Common Crawl dataset 
(which is about ~750GB of data), which is text taken from the internet.
To start with, think about what kind of people are represented in this sort of data.
Internet data over-represents younger users and those from developed countries.

As we have already discussed, _understanding_ language is a process of 
pairing _form_ with _meaning_, and since these large internet datasets only
contain _form_, language models learning from such datasets cannot really gain
_understanding_.

Since large language models don't really learn to _understand meaning_ from these large datasets,
in order to understand what they actually _do_ learn, it is important to understand
what is contained in the training data. Stereotypcial associations and negative
sentiments towards specific groups can often be found in internet datasets - so these views usually find
their way into large LMs. As their usage increases, these biases may end up being reinforced - especially
if decisions start being made upon the predictions of these models.

The stochastic parrot paper suggests that we need to be mindful about how we currate
and create these datasets in future.





[^1]: [Climbing towards NLU: On Meaning, Form, and Understanding in the Age of Data](https://aclanthology.org/2020.acl-main.463/) (Bender & Koller, 2020)
[^2]: why didn't they just say "the time is now 21.56, we should move to the TV room because our favourite TV show is starting 4 minutes"??
[^3]: [Switch Transformers: Scaling to Trillion Parameter Models with Simple and Efficient Sparsity
](https://arxiv.org/abs/2101.03961) (Fedus, Zoph & Shazeer, 2021)
[^note]: Or: their knowledge of the mental state of the other person in the conversation, along with the TV schedule