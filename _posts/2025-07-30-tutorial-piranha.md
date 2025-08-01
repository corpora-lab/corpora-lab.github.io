---
title: The Fish Bowl
tags: [Tutorial]
style: fill
color: info
description: Or how you can express probabilistic conditioning in POPACheck
---

<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-chtml.js"></script>

This post is part of a series of posts explaining some basic concepts of probability theory with the help of [POPACheck](oppas).
Check out the first post [here](tutorial-get-started).


# A piranha in a fishbowl

<img class="main-image illustration-right" src="/assets/pics/fishbowl.webp"/>
In this post we will explore *conditioning*, a very important concept in probability theory.
We'll do so with the help of the following problem:

> One fish is contained within the confines of an opaque fishbowl. The fish is equally
> likely to be a piranha or a goldfish. A sushi lover throws a piranha into the fish
> bowl alongside the other fish. Then, immediately, before either fish can devour the
> other, one of the fish is blindly removed from the fishbowl. The fish that has been
> removed from the bowl turns out to be a piranha. What is the probability that the
> fish that was originally in the bowl by itself was a piranha?

taken from
Henk Tijms, *Understanding Probability: Chance Rules in Everyday Life*. Cambridge University Press, 2007. [[Link]](https://doi.org/10.1017/CBO9780511619052){:target="_blank"}


## Solving the problem with POPACheck

We can model this problem as a probabilistic program in MiniProb,
the input language of POPACheck,
and then let POPACheck do the math.

We use two Boolean variables, `f1` and `f2`, to represent the two fish:
`f1` is the fish that was already in the bowl, and `f2` is the one we throw in.
Each variable takes the value `true` if the corresponding fish is a piranha,
and `false` if it is a goldfish.
For instance, if `f1` = `false` and `f2` = `true`,
then the bowl contained a goldfish, and the fish we threw in was a piranha.

First, we state the fact that the fish in the bowl is equally likely to be a piranha or a goldfish
by stating that variable `f1` is equally likely to be initially `true` or `false`.
We do so with a probabilistic assignment:
```
f1 = true {1 : 2} false;
```
which assigns to `f1` respectively `true` or `false` with probability \\(\frac{1}{2}\\).

Then we "put" a piranha in the bowl by deterministically assigning `true` to `f2`:
```
f2 = true;
```

Finally, we simulate extracting one fish: we are equally likely to get anyone of the two fish in the bowl,
so we represent the extracted fish with a new Boolean variable called `extracted`,
to which we assign the value of either `f1` or `f2` both with probability \\(\frac{1}{2}\\):
```
extracted = f1 {1 : 2} f2;
```

The entire program is the following:
```
fish() {
  bool f1, f2, extracted;
  f1 = true {1 : 2} false;
  f2 = true;
  extracted = f1 {1 : 2} f2;
}
```

We can already start asking POPACheck to tell us some interesting facts about the program.
For instance, what is the probability that we extract a piranha, at this point?
We create a new text file with the following content:
```
probabilistic query: quantitative;
formula = F (ret And fish And [fish | extracted == true]);
program:
fish() {
  bool f1, f2, extracted;
  f1 = true {1 : 2} false;
  f2 = true;
  extracted = f1 {1 : 2} f2;
}
```
Here we are asking POPACheck to approximately determine the probability that a certain property is true about the program.
The property is written as a formula in Linear Temporal Logic,
a mathematical formalism often used to reason about programs.
The formula is
```
F (ret And fish And [fish | extracted == true])
```
The `F` operator is called *Finally* or *Eventually*, and it means that at some point the sub-formula to its right must be true.
The formula to its right says that three things must be true together:
`ret`, which means that a function returns,
`fish`, which means that the currently active function is `fish` (the only one we have in this program),
and `[fish | extracted == true]`, which means that the variable `extracted` must have the value `true`.
The last one could have also been written as `[fish | extracted]`, a syntax more familiar to programmers.
To sum up, we ask POPACheck for the probability that **at some point function `fish` returns with `extracted` = `true`**.
POPACheck prints the following:
```
Quantitative Probabilistic Model Checking
Query: F ((ret And fish) And [fish| extracted])
Result:  (3 % 4,3 % 4)
Floating Point Result:  (0.75,0.75)
...
```
Thus, the probability of extracting a piranha is \\(\frac{3}{4}\\), or \\(0.75\\).

What is the probability of extracting a goldfish?
We just change `[fish | extracted == true]` (or `[fish | !extracted]` for programmers) in the file,
and POPACheck outputs:
```
Quantitative Probabilistic Model Checking
Query: F ((ret And fish) And [fish| ! extracted])
Result:  (312500 % 1250001,1 % 4)
Floating Point Result:  (0.24999980000016,0.25)
```
The probability is indeed \\(\frac{1}{4}\\).


But the problem statement asked us something a bit more complicated.
It assumes that when we draw the fish, we get a piranha,
and then asks us what's the probability that the fish originally in the bowl was a piranha.
In our programmatic setting, this means that we know that `extracted` = `true`,
and we want to know what was the probability that `f1` was assigned `true`.

POPACheck can help us with this.
We need a new statement called `observe`, that tells POPACheck something that is true about the program.
In our case, this has to be `extracted == true` (or just `extracted`).
To properly use this statement, we need to call our function `fish()` with a special statement called `query`.
It means we are querying the value of a probability distribution represented by a function.
Hence we write:
```
probabilistic query: quantitative;
formula = F (ret And fish And [fish | f1]);
program:
main() {
  query fish();
}

fish() {
  bool f1, f2, extracted;
  f1 = true {1 : 2} false;
  f2 = true;
  extracted = f1 {1 : 2} f2;
  observe extracted;
}
```
Note that, this time, in the formula we have `[fish | f1]` because we want to know the probability that `f1` was assigned `true` (= piranha).
POPACheck outputs:
```
Query: F ((ret And fish) And [fish| f1])
Result:  (2 % 3,1111113 % 1666669)
Floating Point Result:  (0.6666666666666666,0.6666668666663866)
```
Thus, the probability that the fish initially in the bowl was a piranha is \\(\frac{2}{3}\\),
and POPACheck has solved the problem for us.


## Solving the problem manually

But what if we wanted to solve this problem ourselves?
Let's make a table!
We will use 🐟 for the piranha, and 🐠 for the goldfish.

| F1 | F2 | Extracted (which one) | Extracted (Type) |
|----|----|-----------------------|------------------|
| 🐟  | 🐟  | F1                    | 🐟                |
| 🐟  | 🐟  | F2                    | 🐟                |
| 🐠  | 🐟  | F1                    | 🐠                |
| 🐠  | 🐟  | F2                    | 🐟                |

By looking at the last column, we can see that the we can extract a piranha 🐟 in 3 cases over 4, hence with a probability of \\(\frac{3}{4}\\).
Clearly, we extract a goldfish 🐠 in one case over four, hence with a probability of \\(\frac{1}{4}\\).

But what's the answer to the problem statement?
To get our answer, we consider only the three rows that contain a piranha in the last column,
hence considering only the cases in which we extract a piranha.
Among these three rows, two have a piranha in the first column, indicating that the fish already in the bowl was a piranha.
These are two cases out of three, so the probability that the fish in the bowl was a piranha knowing that we extracted a piranha is \\(\frac{2}{3}\\).

Note that we can also obtain our result as follows:
\\[
\frac{\frac{2}{4}}{\frac{3}{4}} = \frac{1}{2} \cdot \frac{4}{3} = \frac{2}{3}
\\]

Where does \\(\frac{2}{4}\\) come from?
It's the *joint probability* that the fish already in the bowl was a piranha and that we extract a piranha (which can be any of the two fish in the bowl).
We write it as \\(P(\mathit{F1} = 🐟 \land \mathit{Extracted} = 🐠)\\).
We can easily see that it's \\(\frac{1}{2}\\) by thinking that, if the fish in the bowl was a piranha, then we extract a piranha with probability one, because the fish we put in the bowl is a piranha.
Hence, the joint probability is only given by the probability that the fish in the bowl was a piranha.

The probability that the problem asks us to estimate is called a *conditional probability*,
or the probability that the fish in the bowl was a piranha *conditioned* on the observed fact that we extracted a piranha.
We write it as \\(P(\mathit{F1} = 🐟 \mid \mathit{Extracted} = 🐠)\\).
We have the following relation, that we can use to compute a conditional probability:
\\[
P(\mathit{F1} = 🐟 \mid \mathit{Extracted} = 🐠) = \frac{P(\mathit{F1} = 🐟 \land \mathit{Extracted} = 🐠)}{P(\mathit{Extracted} = 🐠)}
\\]
where \\(P(\mathit{Extracted} = 🐠)\\) is the probability of extracting a piranha.

What do you think? Is it easier to come up with the argument above, or to just let POPACheck do it for us?

---

This blog post was inspired by an example in the article:
F. Olmedo, F. Gretz, N. Jansen, B.L. Kaminski, J.-P. Katoen and A. McIver. [*Conditioning in probabilistic programming*](https://doi.org/10.1145/3156018){:target="_blank"}. ACM Transactions on Programming Languages and Systems (TOPLAS), 40(1), pp.1-50, 2028.


**Keep following this blog posts series to discover more interesting facts about random phenomena,
and how to reason on them with POPACheck.**
