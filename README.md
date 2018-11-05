# prolog-lambda-abstractions
This library provides a minimal set of predicates (currently about 30 lines of code) to implement anonymous predicates (i.e. lambda expressions) in Prolog.


## Features

Compared to other lambda libraries, the present implementation has several advantages:
- Lambda expressions are represented using a simple and natural Prolog syntax.
- Lambda expressions are defined through a local anonymous predicate `_/N` whose form is the well-known `(Head :- Body)` construct. Therefore, there is no need for parameter passing (e.g., using the `^` operator) and no continuations. Note that Body is mandatory (it can be set to true, see below for an example).
- The scope of variables is easy to see. Singletons are treated as globals. Therefore, there is no need for additional syntax (e.g., `\` and `+\` operators as in Ulrich Nuemerkel's lambda libray or `/` and `>>` operators and `{}` construct as in Paulo Moura's LogTalk lambdas).
- There is no constraint on the anonymous predicate name. Therefore, code can be made more readable by using informative names in the tradition of Prolog.


## Examples

Example 1:
```prolog
?- maplist(greater_than_2(X) :- X>2, [3, 4, 5]).
true.
```

Example 2:
```prolog
?- maplist(successor(X, Y) :- Y is X+1, [1, 2, 3], List).
List = [2, 3, 4].
```

The following examples are adapted from http://www.complang.tuwien.ac.at/ulrich/Prolog-inedit/ISO-Hiord.html.

Example 3:
```prolog
?- use_module(library(dif)).
true.

?- Xs = [A,B], maplist(@(Y) :- dif(X,Y), Xs).
Xs = [A, B],
dif(X, A),
dif(X, B).
```

Example 4:
```prolog
?- use_module(library(clpfd)).
true.

?- Xss = [[1,2],[3]], maplist(maplist(@(X,Y,Z) :- X+Y#=Z), Xss, Yss, Zss).
Xss = [[1, 2], [3]],
Yss = [[_3308, _3314], [_3326]],
Zss = [[_3350, _3356], [_3368]],
1+_3308#=_3350,
2+_3314#=_3356,
3+_3326#=_3368.

?- Xss = [[1,2],[3]], maplist(maplist(@(X,Z) :- X+Y#=Z), Xss, Zss).
Xss = [[1, 2], [3]],
Zss = [[_4406, _4412], [_4424]],
3+Y#=_4424,
2+Y#=_4412,
1+Y#=_4406.
```

The following examples are adapted from https://blog.logtalk.org/tag/lambdas/.

Example 5:
```prolog
?- maplist(swap(A-B, B-A) :- true, [1-a,2-b,3-c], Zs).
Zs = [a-1, b-2, c-3].
```

Example 6:
```prolog
?- use_module(library(clpfd)).
true.

?- maplist(whatever(X, Y) :- Z#=X+Y, Xs, Ys).
Xs = Ys, Ys = [] ;
Xs = [_3210],
Ys = [_3228],
_3210+_3228#=Z ;
Xs = [_3800, _3806],
Ys = [_3824, _3830],
_3806+_3830#=Z,
_3800+_3824#=Z ;
Xs = [_4390, _4396, _4402],
Ys = [_4420, _4426, _4432],
_4402+_4432#=Z,
_4396+_4426#=Z,
_4390+_4420#=Z ;
...
```

## Download and installation

To install and use the module, type:
```prolog
?- pack_install(lambda_abstractions).
true.
?- use_module(library(lambda_abstractions)).
true.
```
from the SWI Prolog toplevel.

Please notice that this library has been tested only under SWI Prolog (it relies on the `term_singletons/2` predicate).

Enjoy!
