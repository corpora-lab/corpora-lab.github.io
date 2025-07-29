---
title: Getting started with POPACheck
tags: [Tutorial]
style: fill
color: info
description: How to install POPACheck and start using it
---

In this post we explain how to download and install POPACheck
to get started with analyzing probabilistic programs.


#  The easy way (GNU/Linux and Windows)

You may download the pre-packaged OPPAS executable files from the [GitHub release](https://github.com/corpora-lab/OPPAS/releases/tag/v3.0.0){:target="_blank"}.

The instructions below will work on a GNU/Linux terminal.
If you are running Windows, you will need to install the [Windows Subsystem for Linux (WSL)](https://learn.microsoft.com/en-us/windows/wsl/){:target="_blank"}.
Please follow the [installation instructions](https://learn.microsoft.com/en-us/windows/wsl/install){:target="_blank"} by Microsoft.
Once WSL is installed, [setup and open a terminal.](https://learn.microsoft.com/en-us/windows/wsl/setup/environment#set-up-windows-terminal){:target="_blank"}

If you are using GNU/Linux, you may open a terminal and proceed directly.

First extract the files from the archive with your favorite program.
For instance, open a terminal and run
```sh
mkdir oppas
tar -xvf oppas-3.0.0-x86_64.tar.gz -C oppas
```
The above command creates a new directory named `oppas`, and extracts the files into it.


# Getting started

Among the files extracted from the archive, you'll find the `popacheck` executable.

Let's do a couple of quick checks to see if it works.

First, just run
```sh
./popacheck --help
```
This will print something like
```
POPACheck v3.0.0
```
followed by the usage instructions.
You may ignore the command-line flags listed below,
as they toggle advanced settings that most users won't need to touch.


# A few experiments

POPACheck must be invoked by supplying an input file as its only argument.
Input files follow this format:
```
probabilistic query: <QUERY>;
program:
...
```
POPACheck supports several kinds of queries.
For now, we'll just focus on `approximate` queries,
which compute an approximate lower and upper bound for the termination probability of the first function in the program specified in the file.

Let's try with a very simple file: open your favorite text editor and paste the following text into it:
```
probabilistic query: approximate;
program:
f () { }
```
This program contains only an empty function, and we ask POPACheck for its termination probability.
Then save it as `test.pomc` and run
```sh
./popacheck test.pomc
```

POPACheck should output the following:
```
Probabilistic Termination Checking
Query: ApproxSingleQuery OVI Newton
Result:  ApproxSingleResult (1 % 1,1 % 1)
Elapsed time: 7.005 ms (total), 0.0000e0 s (upper bounds), 0.0000e0 s (PAST certificates), 0.0000e0 s (graph analysis).
Input pOPA state count: 4
Support graph size: 5
Equations solved for termination probabilities: 3
Non-trivial equations solved for termination probabilities: 0
SCC count in the support graph: 7
Size of the largest SCC in the support graph: 1
Largest number of non trivial equations in an SCC in the Support Graph: 0
```
This output is quite verbose and contains many details that we don't need to care about.
The important bits are the following:
- `Probabilistic Termination Checking` says that POPACheck is checking the termination probability of the given program
- `Result:  ApproxSingleResult (1 % 1,1 % 1)` shows the computed lower and upper bounds for the termination probability as fractions of two integer numbers.
   Thus, `1 % 1` means `1/1`, hence the program terminates with probability 1.

Let's try with a slightly less trivial program:
```
probabilistic query: approximate;
program:
f () {
  bool x;
  x = true {1 : 2} false;
  while (x) {
    x = x;
  }
}
```
This program consists of a function that declares a Boolean (`bool`) variable `x`,
and "flips a coin":
it assigns to `x` the value `true` with probability 1/2 (`1 : 2`), and false with probability 1/2.
Then, it enters an infinite loop if the coin flip returned `true`.

If we run POPACheck on it, we get the following:
```
Probabilistic Termination Checking
Query: ApproxSingleQuery OVI Newton
Result:  ApproxSingleResult (1 % 2,1 % 2)
Elapsed time: 7.713 ms (total), 0.0000e0 s (upper bounds), 0.0000e0 s (PAST certificates), 0.0000e0 s (graph analysis).
Input pOPA state count: 7
Support graph size: 10
Equations solved for termination probabilities: 7
Non-trivial equations solved for termination probabilities: 0
SCC count in the support graph: 16
Size of the largest SCC in the support graph: 1
Largest number of non trivial equations in an SCC in the Support Graph: 0
```
Indeed, the program terminates if and only if the coin flip returns `false`,
an event which has probability 1/2 = 0.5.

What if we move the coin flip into the loop?
Let's feed to POPACheck the following file:
```
probabilistic query: approximate;
program:
f () {
  bool x;
  x = true {1 : 2} false;
  while (x) {
    x = true {1 : 2} false;
  }
}
```
In this case, we get
```
Probabilistic Termination Checking
Query: ApproxSingleQuery OVI Newton
Result:  ApproxSingleResult (1 % 1,1 % 1)
Elapsed time: 9.468 ms (total), 2.3829e-4 s (upper bounds), 1.4550e-3 s (PAST certificates), 0.0000e0 s (graph analysis).
Input pOPA state count: 8
Support graph size: 11
Equations solved for termination probabilities: 9
Non-trivial equations solved for termination probabilities: 1
SCC count in the support graph: 19
Size of the largest SCC in the support graph: 1
Largest number of non trivial equations in an SCC in the Support Graph: 1
```
Hence the termination probability of the program is again 1,
or it *terminates almost surely*.
This is not surprising:
the program exists the loop as soon as the coin flip in it returns `false`.
Although the probability of this event to happen in any individual coin flip is lower than 1,
the more times we flip the coin (i.e., run the loop) the higher the probability of getting `false` *at least once*.
Hence, as the number of loop iterations approaches infinity, the probability of exiting the loop (and terminating the program) approaches 1.
This phenomenon is known as the [second Borel–Cantelli lemma](https://en.wikipedia.org/wiki/Borel%E2%80%93Cantelli_lemma#Converse_result){:target="_blank"},
and it is a consequence of the well-known [Law of Large Numbers](https://en.wikipedia.org/wiki/Law_of_large_numbers){:target="_blank"}.


**Keep following this blog posts series to discover more interesting facts about random phenomena,
and how to reason on them with POPACheck.**
