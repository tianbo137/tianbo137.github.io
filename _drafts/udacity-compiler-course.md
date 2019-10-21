---
layout: post
title: Udacity Compiler Course
date: '2019-08-03 14:32'
published: true
---

This is the notes I took for [Compilers: Theory and Practice](https://classroom.udacity.com/courses/ud168), offered by Udacity.

* TOC
{:toc}

# Introduction to Compilers
Compiler is a program that *translates* a program from a *source language* into a *target language*. As a comparison, an interpreter is a computer program that directly executes instructions written in a programming or scripting language, without requiring them previously to have been compiled into a machine language program. An interpreter generally uses one of the following strategies for program execution:

1. Parse the source code and perform its behavior directly;
1. Translate source code into some efficient intermediate representation and immediately execute this;
1. Explicitly execute stored precompiled code[1] made by a compiler which is part of the interpreter system.

The goal of compilers is to avoid writing assembly program and increase developer productivity.

## How Compilers Work
A compiler consists of the front end and the back end. The **front end** of the compiler performs analysis:

1. Lexical analysis: through scanner;
2. Syntax analysis: through parser;
3. Semantic analysis: through semantic action.

The **back end** performs synthesis to convert the intermediate representation into assembly. This step is called code generation. All modern compilers perform optimizations, too.

**Scanner** goes through the entire file and produce tokens (equivalent to words in a book), which has a type and a value. For example, `157.669` is a token because it is surrounded by whitespaces, and its type is `FLOATCONST` (floating point constant) and the value is `157.669`. Besides constants, tokens include variables and keywords.

Scanning is the process of converting input text into stream of known objects called tokens. It simplifies the parsing process.

The **parser** requests tokens from the scanner and ensure that the returned token are legal in the "sentence". Parsing is the process of translating code to rules of grammar, building representation of code.

Once enough sentence is obtained, it verifies if the sentence has the right meaning. **Semantic Action** ensures the sentence that is being parsed currently is semantically meaningful. If the check fails, it returns a semantic error.

If the sentence is semantically meaningful, it is translated into an *intermediate representation* that is closer to the assembly. Scanner, parser, and semantic action together are called the **front end**, the analysis of program syntax and semantics and the transformation into the intermediate representation. Because the front end depends highly on the language syntax, it is very specific to a language.

Semantic analysis includes looking up variable name. Visible variable names are stored in the symbol table. If the sentence is a declaration of variables, it is put in the symbol table; if it is a use of variables, semantic checks are run to ensure the operation on the variable is valid.

*Lexical rules of language* dictate how legal word is formed by concatenating alphabet. Lexicon means dictionary.

*Grammar* dictates syntactic rules of language, i.e., how legal sentence could be formed.

## Scanning and Tokenization
Scanner takes a file as input and processes it character-by-character using a token buffer. Scanner reads characters into the token buffer until the next character no longer satisfies the lexical rules of the language, and a valid token is found. This process is called longest matching algorithm and it is used by all the scanners.

## Parser
Parser is the center of the front end, it controls the overall operation.

Parsing depends on the grammar rules. The following example illustrates grammar rules of a simplified `main` function of C:

```
<C-PROG> ➝ MAIN OPENPAR <PARAMS> CLOSEPAR <MAIN-BODY> // <C-PROG> is the start symbol. Every C program must start with a main function
<PARAMS> ➝ NULL // PARAMS can be NULL
<PARAMS> ➝ VAR <VARLIST> // PARAMS can be a single VAR, or a single VAR followed by a VARLIST
<VARLIST> ➝ , VAR <VARLIST> // VARLIST is comma-separated VARs followed by NULL
<VARLIST> ➝ NULL
<MAIN-BODY> ➝ CURLYOPEN <DECL-STMT> <ASSIGN-STMT> CURLYCLOSE
<DECL-STMT> ➝ <TYPE> VAR <VARLIST>;
<ASSIGN-STMT> ➝ VAR = <EXPR>;
<EXPR> ➝ VAR
<EXPR> ➝ VAR <OP> <EXPR>
<OP> ➝ +
<OP> ➝ -
<TYPE> ➝ INT
<TYPE> ➝ FLOAT
```

Note that the grammar rule does not specify everything about a language, e.g. type restrictions, and it is up to semantic checks. The reason is that parsing is context *insensitive* but semantic analysis is context *sensitive*. It is more expensive to perform semantic analysis than parsing.

Parser works by matching tokens with rules. When match takes place at certain position, move further (get next token & repeat). If expansion needs to be done, choose appropriate rule. If no rule found, declare error. If several rules found, the grammar (set of rules) is ambiguous, and well-designed grammar rules should not be ambiguous.

Expansion determines how parts in `<>` get interpreted as they have multiple ways of matches.

### Ambiguity
Ambiguity can lead to understanding a given sentence in two different ways, which allows assigning two meanings to the same sentence, something highly undesirable. It is up for the language designers to remove ambiguity in the language syntax.

## Semantic Analysis
### Symbol Table
The symbol table shows us what are the different values which are declared and available in the current scope. Most of languages are block-based, which means declaration available in the current scope is also available in inner scopes unless they are redeclared.

A basic symbol table contains the name and the type of variables as well as the scope in which they are available. Semantic analysis then look up symbols to see if they are declared and to determine the types of them.

### Semantic Actions
There are the many types of semantic actions. Some examples include:
1. Enter variable declaration into the symbol table;
2. Look up variables in the symbol table;
3. Do binding analysis of looked-up variables based on scoping rules. From the innermost scope upward, find the variable declaration and bind the variable to it.
4. Type checking for compatibility;
5. Keep the semantic context of processing. When a subexpression is checked, the subexpression itself has a type. This process keeps track of the type for further checks that the subexpression participates. For example, expression `a + b + c` can be broken down into subexpression `t1 = a + b` and `t2 = t1 + c`. The process keeps track of the type of `t1` and `t2` when they are processed.

They can be largely grouped into two categories:
1. Checking: binding, type compatibility, scoping, etc.
2. Translation: generate temporary values, propagate them to keep semantic context.

A simple way of implementing semantic analysis is to embed action symbols in the grammar. An *action symbol* is to tell us about a *semantic procedure* which is being triggered at that particular point during parsing. Semantic procedures contains semantic actions and return values. Semantic procedures are called by the parser at appropriate places during parsing. *Semantic stack* can be used to implement and store semantic records.

Semantic actions can be embedded in the grammar like this:

```
<decl-stmt> ➝ <type>#put-type<varlist>#do-decl
<type> ➝ int | float
<varlist> ➝ <var>#add-decl <varlist>
<varlist> ➝ <var>#add-decl
<var> ➝ ID#proc-decl
#put-type puts given type on semantic stack
#proc-decl builds decl record for var on stack
#add-decl builds decl-chain
#do-decl traverses chain on semantic stack using backwards pointers entering each var into symbol table
```

Declaration chain means all related variables are of the same type defined in `put-type`.

# Lexical Analysis
Scanner performs lexical analysis, which reads input one character a time, group characters into tokens, remove whitespaces and comments, and encode token types and form tuples of `<token-type, value>` and return to parser.

Lexical analysis is based on lexical rules, which determines how to form legal strings. They are expressed using regular expression `r` and its language `L(r)`, and analyzed with finite automata, a type of state machine.

## Regular Expression
A symbol is a valid character in a language, and alphabet, $$\Sigma$$, is the set of legal symbols.

Regular expression contains metacharacters/metasymbols that have special meanings:
* defining reg-ex operations (e.g. `|`, `(`, `)`, `*`, `+`, etc.)
* Escape character `\` to turn off special meanings
* Empty string $$\epsilon$$ and empty set $$\phi=\{\}$$

Basic regular expression operations include
* Alternation, denoted by `a | b`,
* Concatenation， denoted by `ab`,
* Repetition (Kleen closure, 0 or more times) `a*`.

The precedence is repetition > concatenation > alternation. Parentheses can be used to change precedence.

Unix-style regular expressions introduces shorthands to simplify regular expressions.
* `.` denotes any character in the alphabet
* `[]` denotes a character class, allowing range (e.g. `[a-d]`) and complement (e.g. `[^1-3]`)
* `+` repeat one or more times
* `?` optional (zero or one time)
* `^` and `$` denote beginning and end of line

We can use regular expression to specify valid tokens. For example,
* `[0-9][0-9]*` is unsigned integer, and `(+|-)?[0-9][0-9]*` is signed integer.
* `[+|-]?[0-9]+(\.[0-9]*)?([eE][+|-]?[0-9]+)?` is floating point number.

## Deterministic Finite Automata (DFA)
Deterministic Finite Automata (DFA) is a state machine *without ambiguity* (the behavior of the machine is repeatable using the same string).
* Deterministic means the machine is in a state. Upon receipt of a symbol, it will go to a unique state.
* Finite: have a finite number of states.
* Automata: it is a self-operating machine.

A language is called a **regular language** if some finite automaton *recognizes* it.

An example of DFA is below:

![Alt text]({{ "/assets/posts/udacity-compiler-course/DFA.png" | absolute_url }})

* There are two types state, rejecting state (single circle) and accepting state (double circle), meaning to accept or reject the string if the automaton ends in it.
* The incoming arrow indicates the start state.

The formal definition of DFA: a DFA consists of:
1. Alphabet $$\Sigma$$;
2. A set of states $$Q$$;
3. A transition function $$\delta: Q \times \Sigma \rightarrow \Sigma$$;
4. One start state $$q_0$$;
5. One or more accepting states: $$F \subseteq Q$$.

The language accepted by a DFA is the set of strings such that the DFA ends at an accepting state.
* Each string is $$c_1c_2...c_n$$ with $$c_i \in \Sigma$$;
* States are $$q_i = \delta(q_{i-1}, c_i)$$ for $$i = 1, ..., n$$;
* $$q_n$$ is an accepting state.

## Nondeterministic Finite Automata (NFA)
Nondeterministic Finite Automaton (NFA) differs from DFA in three ways:
1. Same input may produce multiple parts;
2. Allows transition with an empty string. $$q_1$$ can transition into $$q_2$$ on empty string $$\epsilon$$, also knows as "cloning".
3. Transition from one state to several different states on a given character. In other words, the NFA clones itself and operates from different states on a given character.

The NFA starts in the start state. When encountering transitions on $$\epsilon$$ or multiple transitions on the same character, it clones itself. If no legal transition from the state on the incoming alphabet, the machine dies. If one copy of the NFA ends an accepting state, then the NFA accepts the string.

The formal definition of NFA: a NFA consists of:
1. Alphabet $$\Sigma$$;
2. A set of states $$Q$$;
3. A transition function $$\delta: Q \times \Sigma_\epsilon \rightarrow P(Q)$$ where $$\Sigma_\epsilon = \Sigma \cup \{\epsilon\}$$ and $$P(Q)$$ is the power set of $$Q$$;
4. One start state $$q_0$$;
5. One or more accepting states: $$F \subseteq Q$$.

The only difference between DFA and NFA is the definition of the transition function.

The language accepted by a NFA is the set of strings such that the NFA ends at an accepting state.
* Each string is $$c_1c_2...c_n$$ with $$c_i \in \Sigma_\epsilon$$;
* States are $$q_i \in \delta(q_{i-1}, c_i)$$ for $$i = 1, ..., n$$;
* $$q_n$$ contains an accepting state.

## Relationship between DFA and NFA
>Theorem: For every language L accepted by an NFA, there is a DFA that accepts L.

In other words, DFA and NFA are equivalent computational models. The proof idea is to combine the set of states in an NFA that a substring can reach and produce a corresponding state in the DFA. By iterating through the string, we can produce a corresponding DFA from the NFA.

## Closure
> Closure property: certain operations performed on languages in a class will produce results in the same class.

Regular languages are closed under regular operations:
* Union:

$$L_1 \cup L_2=\{x|x\in L_1 \,\textrm{or}\, x\in L_2\}$$

* Concatenation:

$$L_1 \dot L_2=\{xy|x\in L_1 \,\textrm{and}\, y\in L_2\}$$

* Star/repetition:

$$L_1*=\{x_1...x_k|x_j\in L_1 \,\textrm{for all}\, 1\leq j\leq k, k \geq 0\}$$

To prove operations are regular, assuming $$M_1$$ and $$M_2$$ are NFA for $$L_1$$ and $$L_2$$, respectively, we can construct a NFA $$M_3$$ such that $$L(M_3)$$ equals the result of the regular operations.

* Union: Add a start state with $$\epsilon$$ transitions into either $$M_1$$ or $$M_2$$. Therefore, if a string is in $$L_1$$ then the clone to $$M_1$$ will end in accepting state.
![Alt text]({{ "/assets/posts/udacity-compiler-course/closure-union.png" | absolute_url }})

* Concatenation: Add $$\epsilon$$ transitions from accepting states in $$M_1$$ to the start state in $$M_2$$.
![Alt text]({{ "/assets/posts/udacity-compiler-course/closure-concatenation.png" | absolute_url }})
* Start: Add a new start state that is an accepting state (for empty string) with $$\epsilon$$ transition to the original start state, and add $$\epsilon$$ transitions from accepting states to the original start state.
![Alt text]({{ "/assets/posts/udacity-compiler-course/closure-star.png" | absolute_url }})

Therefore, we can use regular operations to construct larger NFAs from simple NFAs.

Because a regular language has some DFA recognizes it, DFA can be converted from NFA, and NFA can be constructed from regular operations (i.e. regular expressions), we can conclude:

> A language is regular **if and only if** some regular expression describes it.

## Building Lexical Analysis
To convert a language specification into code:
* Write down the Regular Expression for the input language;
* Build a big NFA;
* Build the DFA that simulates the NFA;
* Systematically shrink the DFA;
* Turn it into code.

### RE to NFA: Thompson's Construction
Key idea:
* Construct NFA patterns for each symbol and each operator. For example, the NFA for character `a` is as follows.
![Alt text]({{ "/assets/posts/udacity-compiler-course/thompsons-construction-symbol.png" | absolute_url }})

* Join them with $$\epsilon$$ moves by using the precedence order. For example, the NFAs for character `ab` and `a*` are as follows.
![Alt text]({{ "/assets/posts/udacity-compiler-course/thompsons-construction-concatenation.png" | absolute_url }})
![Alt text]({{ "/assets/posts/udacity-compiler-course/thompsons-construction-star.png" | absolute_url }})

A more complex example of `(b|c)*`:
![Alt text]({{ "/assets/posts/udacity-compiler-course/thompsons-construction-join.png" | absolute_url }})

### NFA to DFA: Subset Construction
Two key functions:
* $$Move(s_i,\alpha)$$ is the set of states reachable from $$s_i$$ by $$\alpha$$.
* $$\epsilon -closure(s_i)$$ is the set of states reachable from $$s_i$$ by $$\epsilon$$.

The idea of the algorithm is
1. Start state $$S_0$$ is derived from $$s_0$$ of the NFA: $$S_0=\epsilon-closure({s_0})$$.
2. Take the image of $$S_0$$, $$Move(S_0,\alpha)$$ for each $$\alpha\in \Sigma$$, and take its $$\epsilon-closure$$.
3. Iterate until no more states are added.

The algorithm is
![Alt text]({{ "/assets/posts/udacity-compiler-course/subset-construction-algo.png" | absolute_url }})

Note:
1. $$W$$ is the list of states to explore;
2. $$T$$ is the transition function.

A DFA state is an accepting state if its $$\epsilon$$-closure contains an NFA accepting state.

Subset construction is an example of **fixed-point computation**:
* *Monotone construction* of some finite set;
* *Halts* when it stops adding to the set.

### DFA Minimization
The goal of DFA minimization is to reduce the size of the transition table.

> Observation: Subset construction algorithm merges prefixes in the NFA.

However, it does not merge suffixes. To do so, use subset construction twice.

For an NFA $$N$$:
* Let $$reverse(N)$$ be the NFA constructed by making initial state final (and vice versa) and reversing the edges.
* Let $$subset(N)$$ be the DFA that results from applying the subset construction to $$N$$.
* Let $$reachable(N)$$ be $$N$$ after removing all states that are not reachable from the initial state.

Then

$$reachable(subset(reverse(reachable(subset(reverse(N))))))$$

is the minimal DFA that implements $$N$$. This essentially is to merge suffixes first and merge prefixes second by subset construction.