---
title: Reasoning about Reasoning
tags: [Tutorial]
style: fill
color: info
description: Probabilistic programs for the Theory of Mind
---

<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-chtml.js"></script>

This post is part of a series of posts explaining some basic concepts of probability theory with the help of [POPACheck](oppas).
Check out the first post [here](tutorial-get-started).


## An experiment on Theory of Mind

Thomas C. Schelling (1921--2016) was awarded the Nobel Prize in Economics in 2005 for his contributions to game theory in the area of cooperation and conflict analysis.
He studied the problem of how people can reach an agreement on how to achieve a common goal without being able to communicate.

<img class="main-image illustration-left" src="/assets/pics/station.webp"/>
An example of one of such problems is an experiment that he conducted with a group of students<sup>1</sup>.
He told them that they had to meet in New York City on a given date at the same time and in the same place.
However, he did not suggest any time or place, and told the students that they could not communicate to reach an agreement.
Thus, each student picked a time and place that they deemed likely to be picked by other students too.
The majority of the students chose to meet at the Grand Central Station at noon.

This and other experiments by Schelling demonstrate that it is possible to reach a common agreement without communication.
The way people achieve such agreements is by thinking about what other people would probably choose.
Thus, they have a mental model of other people's thinking.
A person's model may include the fact that other people will reason about how that very person might reason, in a possibly unboundedly recursive way.

The ability of reasoning about other people's cognitive abilities is called Theory of Mind in philosophy and psychology.
But why are we writing about cognitive science in a blog post about probabilistic programming?

It turns out that probabilistic programs can be used to formalize the Theory of Mind,
providing mathematical models that try to explain human reasoning patterns.
Noah D. Goodman has been a proponent of this use of probabilistic programs.
In an article with Andreas Stuhlmüller<sup>2</sup>, he proposed a simplified coordination game, inspired to Schelling's experiments.
In this post, we show a variation of this game.

<img class="main-image illustration-right" src="/assets/pics/cafe.webp"/>

## A Schelling Coordination Game

Two friends, Alice and Bob, would like to meet in a café.
They have agreed on a time, but they postponed the decision of which café to meet in.
Unfortunately, due to a widespread outage of the telecommunications infrastructure in their town, they cannot text each other to agree on the location.
However, they always meet in one of two cafés, and both Alice and Bob have a slight preference for one of them.
Thus, Alice tries to simulate Bob's reasoning in her mind.
Bob's reasoning includes, in turn, Alice's reasoning.
Bob, however, is not completely fair: he might sometimes decide not to simulate Alice's reasoning, and just picks one of the cafés by following his own judgment.


### Modeling Alice's and Bob's choice with a probabilistic program

We model this situation with a probabilistic program containing two functions, one representing Alice's reasoning, and one Bob's.

Alice's function is the following:
```
alice(bool &x) {
    bool prior_alice, bob_choice;
    prior_alice = true {11 : 20} false;
    query bob(bob_choice);
    observe prior_alice == bob_choice;
    x = prior_alice;
}
```
It has one parameter that is passed by reference, and two local variables.
The first local variable, `prior_alice`, represents Alice's prior assumption about Bob's choice.
This assumption consists in a slight preference for one of the two cafés.
We represent it as a probabilistic assignment, that returns `true` with probability \\(\frac{11}{20} = 0.55\\).
Thus, here `true` represents the most popular café, and `false` the other one.

The other variable, `bob_choice`, is Bob's choice according to the reasoning that Alice simulates for him.
We represent the result of this simulation as the value returned by function `bob`, that represents Bob's reasoning.
We call `bob` with a `query` statement, because we are drawing from the probability distribution it represents:
Alice *queries* Bob's model, asking what will be his most likely choice.

Alice then *conditions* her probability distribution on the fact that the location she chose is the same as Bob's with the `observe` statement.
Finally, the function returns the resulting choice by assigning it to parameter `x`, passed by reference.

The `observe` statement is essential for the model to result in an agreement.
Conditioning the model on the fact that Alice's and Bob's choices agree means, in a sense, *assuming* that they agree.
The probability distribution resulting from such a program will only contain values for the café choice that result from converging choices.

Bob's function is similar, but not symmetric.
```
bob(bool &y) {
    bool prior_bob, recurse, alice_choice;
    prior_bob = true {11 : 20} false;
    recurse = true {1 : 5} false;
    if (recurse) {
        query alice(alice_choice);
        observe prior_bob == alice_choice;
    } else {}
    y = prior_bob;
}
```
Recall that Bob is more biased towards his own choice, and he may not always care of Alice's reasoning.
We represent this feature through another Boolean variable, `recurse`.
Its value is drawn from a "biased coin flip", that yields `true` with probability \\(\frac{1}{5} = 0.2\\), and `false` otherwise.
If `recurse` is `true`, Bob will take Alice's reasoning in his decision process; otherwise, he won't.
The rest of the function is symmetric to Alice's.

Finally, we write a `main` function that queries Alice's choice:
```
main() {
  bool res;
  query alice(res);
}
```
At the end of the program's execution, variable `res` will tell us in which café Alice and Bob will meet.


### Analyzing the thought process

What is interesting, of course, is not the value of a single execution of the program, but rather the resulting stochastic distribution.
We can ask POPACheck to tell us facts about the probability distribution of variables in the program.

Of course, we want to know what is the probability that Alice and Bob will agree on choosing the most popular café, the one represented by value `true`.
To do so, we ask for the probability of the following formula being true:
```
XNu ([main|res == true])
```
`XNu` is a special operator that checks what happens when the current function returns.
If it not nested in any other operators, the "current function" is the `main` function.
So we check what is the probability that variable `res` is `true` when the `main` function terminates.
We give to POPACheck an input file with the following contents:
```
probabilistic query: quantitative;
formula = XNu ([main|res == true]);

program:
main() {
  bool res;
  query alice(res);
}

alice(bool &x) {
    bool prior_alice, bob_choice;
    prior_alice = true {11 : 20} false;
    query bob(bob_choice);
    observe prior_alice == bob_choice;
    x = prior_alice;
}

bob(bool &y) {
    bool prior_bob, recurse, alice_choice;
    prior_bob = true {11 : 20} false;
    recurse = true {1 : 5} false;
    if (recurse) {
        query alice(alice_choice);
        observe prior_bob == alice_choice;
    } else {}
    y = prior_bob;
}
```

POPACheck answers as follows:
```
uantitative Probabilistic Model Checking
Query: XNu [main| (res == [1]1)]
Result:  (102 % 167,9803 % 16050)
Floating Point Result:  (0.6107784431137725,0.6107788161993769)
```

Thus, the probability of choosing the most favored café is around \\(0.61\\), which is higher than the preference bias \\(0.55\\).
Indeed, \\(0.55\\) would be the probability of Alice and Bob choosing the café represented by `true` if they didn't engage in reasoning about each other's reasoning process.
Thus, this recursive reasoning process does influence the choice.

But what is the probability that they will converge on the same choice?
We ask POPACheck by modifying the beginning of the previous input file as follows:
```
probabilistic query: quantitative;
formula = XNu ([main|resa == resb]);

program:
main() {
  bool resa, resb;
  query alice(resa);
  query bob(resb);
}
```
Here we query both Alice's and Bob's mental models, and then we check whether the two resulting choices are the same (`resa == resb`).
We obtain the following answer:
```
Quantitative Probabilistic Model Checking
Query: XNu [main| (resa == resb)]
Result:  (1996 % 3885,485 % 944)
Floating Point Result:  (0.5137709137709138,0.513771186440678)
```

Thus, the probability that they actually manage to meet is around \\(0.51\\).

But what would be the probability that the come to the same choice if both of them ignored each other's reasoning?
Try to answer this question by modifying the probabilistic program above and feeding it to POPACheck.

(Hint: remove the `query` and `observe` statements in functions `alice` and `bob`.)


**Keep following this blog posts series to discover more interesting facts about random phenomena,
and how to reason on them with POPACheck.**


---

1. T.C. Schelling: *Bargaining, communication, and limited war*. Conflict Resolution, 1(1), pp. 19-36, 1957. [[Link]](https://www.jstor.org/stable/172548){:target="_blank"}
2. A. Stuhlmüller, N.D. Goodman: *Reasoning about Reasoning by Nested Conditioning: Modeling Theory of Mind with Probabilistic Programs*. Cognitive Systems Research, 28, pp. 80-99, 2014. [[Link]](https://doi.org/10.1016/j.cogsys.2013.07.003){:target="_blank"}

