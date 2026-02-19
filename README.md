# Regex FSM Tool

This is a web application that visualizes the minimum-state DFA equivalent to a
regular expression.

This tool supports only the printable class of ASCII characters.

\[WIP\]

## Demo

View a demo at [the GitHub page](https://aktriv.github.io/re-fsm-tool).

## Regex flavors

 - POSIX.2 Basic/Extended
   - Intended to be compliant with the specification. Rejects expressions that
     are deemed implementation-defined by the specificiation.
 - University of Maryland CSMC330 syntax
   - Supports "|" for alternatation, "*" for repetition, and "("/")" for
     grouping, and only alphabetic characters
   - There is no way to encode a regex for the empty string.

## Build & run locally

```sh
git clone https://github.com/aktriv/re-fsm-tool.git
cd re-fsm-tool
zig build run-server -Dopen
```

The [Zig](https://ziglang.org/download/) compiler is a dependency for building
this project.

Running `zig build run-server -Dopen` will spawn the webserver locally and
open it in a browser.

## On regular expressions, FSMs, and DFAs

*Regular expressions (regexes? regices??)* are said to be discovered in 1951 by
mathematician Stephen Cole Kleene. The kind of regular expressions we see today
in text processing software can be attributed to Ken Thompson, who added
regex-based capabilities to early programs such as `ed`. Ken Thompson's initial
implementation of regex used *finite state machines (FSMs)* based on the
mathematical theory. However most modern regex engines nowadays use
backtracking instead.

A regex is a pattern which can be used to match certain strings. A regex and a
string can either match or fail to match. Thus, for any regex, there is a set
of all strings that match it.

An regex engine implemented with FSMs converts the regex to an *equivalent*
FSM, an FSM that accepts precisely the strings that the input regex matches.
There are two common variants of FSMs: *nondeterministic finite automata
(NFAs)* and *deterministic finite automata (DFAs)*. It is straightforward to
convert a regex to an NFA with a number of states roughly proportional to the
length of the regex, while a DFA may require an exponential number of states.
Therefore, NFAs are the more practical option for regex engines. However, DFAs
have the cool property that for any regular language, there is a unique DFA
with the smallest number of states that precisely accepts the language. In
other words, when asked to find the minimum DFA equivalent to a given regex,
*there is only one right answer*. The goal of this project is to find that
answer, for any regex in any common flavor.

A pitfall of FSMs as opposed to backtracking is that the set of strings that an
FSM accepts must be a *regular language*. Many features supported by modern
regex libraries, like backreferences, allow regular expressions to match
non-regular languages (making "regular expression" somewhat of a mathematical
misnomer for these patterns). Other features, such as capture groups, can be
implemented with special modifications to FSMs. Since this project is focused
on generating pure FSMs, backreferences, capture groups, and other
regular-language-unfriendly features will not be supported. Surprisingly, FSMs
can still support many complicated regex features including [lookahead and
lookbehind](https://www.regular-expressions.info/lookaround.html).

## Spec

Currently, this program is designed to support ASCII printable characters, the
95 characters from ' ' (0x20) to '~' (0x7E), inclusive. Future versions may
support the full set of ASCII characters, NUL (0x00) to DEL (0x7F).

Implemented Re -> AST -> NFA -> DFA -> minimized DFA.

DFA minimization is implemented by reversing the FSM twice.

### Code organization

Written for Zig 0.14.0

 - `src/lib`: the backend/library
   - `src/lib/Ast.zig`: a common abstract syntax tree for the language of
     regular expressions
   - `src/lib/Ast/parsers.zig`: parsers for the supported flavors
   - `src/lib/Nfa.zig`: deterministic finite-automaton, represented logically
     as a set of edges and a set of states. has an additional concept of a
     "gate edge", which is an edge conditioned on the acceptance of a sister
     FSM, to make lookaheads/lookbehinds/conjunction easier to implement
   - `src/lib/Dfa.zig`: deterministic finite-automaton, represented logically
     as a map (state, symbol) -> state
   - `src/lib/root.zig`: minimization is implemented here
 - `src/bin`: local binaries
   - `src/bin/server.zig`: run a webserver for the static site locally
   - `src/bin/cli.zig`: command line interface for testing the library
 - `src/web`: files for creating the static site that serves a WASM binary
   - `src/web/app.js`: code to load the WASM binary
   - `src/web/{index.html,styles.css`: code for the static site
   - `src/web/re-fsm.zig`: code for the WASM binary
   - `src/web/favicon.ico`: for browser icon

### TODO

Update to Zig master

Construct a reversed DFA initially, and then reverse once to minimize.

Canonicalize DFA state numbers (if not done already).

Add support for a regex programming language. An "expression" evaluates to
a regex, and can be one of the following forms:
 - let \<var\> = \<exprA\> in \<exprB\>
 - if \<exprA\> then \<exprB\> else \<exprC\>
 - \<exprA\> or \<exprB\>
 - \<exprA\> and \<exprB\>
 - not \<exprA\>
 - reverse(\<exprA\>)
 - concat(\<exprA\>, \<exprB\>)
 - posix_bre(\<regex\>)
 - posix_ere(\<regex\>)
 - cmsc330(\<regex\>)

Eventually, to support the following flavors:

 - Vim (extension of Posix Basic)
 - PCRE/PCRE2
 - Python, Java, Golang, Rust, C\#

### Renderers

 - Viz.js looks nice, but has a low state limit
 - Dagre/D3 looks a little worse, but has an incredibly high state limit

The new plan is to use Dagre with a custom interactive renderer
