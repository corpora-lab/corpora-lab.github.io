---
name: OPPAS
tools: [Haskell]
image:
description: Operator-Precedence Program Analysis Suite
---

# OPPAS

OPPAS (Operator-Precedence Program Analysis Suite) is a tool suite for the analysis of recursive programs.

It contains two tools: POMC and POPACheck.

**POMC** is a model-checking tool for verifying procedural programs with exceptions against POTL (Precedence Oriented Temporal Logic) specifications.
POTL is a temporal logic capable of expressing context-free specifications, particularly well-suited for verifying procedural programs with exceptions.
It can express properties such as:
- Stack inspection
- Function-local properties
- Hoare-style pre/post conditions for functions
- No-throw guarantee (for exceptions)
- Exception safety

**POAPACheck** is a model-checking tool for recursive probabilistic programs formally modelled as probabilistic Operator Precedence Automata (pOPA), a subclass of probabilistic Pushdown Automata.
POPACheck support model checking of the temporal logics LTL and a fragment of POTL.

Consult the [project page](https://github.com/corpora-lab/OPPAS){:target="_blank"} for more.
