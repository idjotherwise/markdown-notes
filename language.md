---
title: 'Language'
date: '2022-02-27'
---

Language is a fundamental part of the way people communicate with each other, but it far from a simple task to understand fully what is going on in a piece of text.

# Morphosyntax
Morphosyntax is the study of both morphology and syntax in language. Without such considerations, there would be no difference between just a collection of words and a proper sentence. The morphosyntax of a language is the constraints that it places on how words can be combined both in form and in the resulting meaning.

## Noun classes
One such example of constraints placed on the well-formdness of a sentence is _noun classes_ (or 'gender'). Consider the following example from the Ingush language (a Nakh-Daghestanian language of the Caucasus):
```
a.  jett arra-b.ealar
    cow  out-B.go.WP
    'The cow went out.'

b.  zhwalii arra-d.ealar
    dog     out-D.go.WP
    'The dog went out.'
```
The verb forms in the above two examples (the b vs d) reflects the noun class of the subject - the dog and the cow. So the constraint in this case is:

> The verb must agree with the gender of the noun bearing nominative case, else the sentence is ill-formed.

In the above example, the _WP_ stands for _witnessed past tense_, which is distinct in the tense system of Ingush from the _present_, _future_ and _non-witnessed past_ forms. Also the **B** and **D** are noun classes specific to Ingush.

## What kind of who did what kind of thing to what kind of whom, where, why, when and how?

In many natural language processing tasks, we would like to extract (or decoding) from a sentence exactly _**who** did **what** to **whom**_ as well as other similar pieces of information. Therefore understanding how different languages do the reverse problem of actually imputing (or encoding) this information can help inform effective design of systems that extract it.

## Influence of other languages

One can group languages together into families in many different ways. By studying shared precursor languages, or by looking at the (possibly historic) geographical location of where the language was spoken, or by looking at grammatical similarities (e.g, strategies for indicating and encoding syntactic and semantic roles). 

Every language evolves over time, but there are many factors that influence exactly _how_ they change. As on over-simplified example, consider a tribe all speaking one language, living a quiet but hard life in the hot desert. Then over several generations, some of the tribe moves across the continent to a mountinous region with lots of snow and forests. Some move and lay the foundations for a bustling city and establish a trade hub. Others build boats and travel the oceans.
In each of the now seperate groups of people that spoke the originally same language, words may carry very different weight of importance. 

## The mountaineers
In the first group which moved to the mountains, it may be too cold at times to use long sentences and so gradually some word endings may get droped, sentences get more 'to the point'. People waste less energy on speaking, so the natural rythm of the language slows down. Being able to distinguish between a snow leopard who is _sleeping_ and a snow leopard that is _hunting for prey_ may be extremely important, and so a new noun class is introduced ('sleopard' => 'a leopard who is sleeping', 'hleopard' => 'a leopard who is hunting'??) which may influence other words in the sentence. Suddenly a whole new set of restrictions is introduced into the language. 
> The action of the subject must agree with the action observed by the observer

## Cityfolk
Those that founded a city had to import many construction materials to build up infrastructure. Trading and bartering became second nature, the speed of the language increased. Many phrases and sayings revolved around money and traded goods. Measuring something based on it's monitary value becomes engrained in the language to the point that every noun and verb can have an ending indicating what value the subject places on it. There was safety in numbers, technological advancement boomed. Using language to communicate _how you feel_ about something became less and less common - there was a smart watch to deal with feelings. Asking for help became a old-fashioned thing because it showed weakness, and weakness is not a traders best attribute. The language becomes a hard, emotionless and money-driven one. Infact, a new set of restrictions emerge:
> The action of the subject must be an action of something with the same exact objective monitary value, and not with what the oberver believes to be true.

## Boaters



## The others
 Even within those groups, there may be socio-economic divides that influence how people speak and communicate with eachother. Perhaps a class divide emerges where only the rich are educated from text-books while others only learn verbally. The variants of the orignal language will vary dramatically in each of these different situations - some may form sub-families of languages, others whole new families.

# Conclusion

Understanding how languages have evolved and asking why they have evolved in the way that they have is a very interesting topic. Can we understand and reason about what happened to a nation 2000 years ago based on how the language evolved from after its decline? Can we use our knowledge of historical change in languages to predict and highlight socio-economic changes that are happening _now_ or will happen in the future?
