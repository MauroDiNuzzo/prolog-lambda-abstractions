# Prolog Lambda Expressions
## `library(lambda_abstractions)`

This library provides a minimal set of predicates (currently about 30 lines of code) 
to implement anonymous predicates (i.e. lambda expressions) in Prolog (presently
developed under SWI Prolog 7.x). 

Please notice that this library relies on `copy_term_nat/2` and `term_singletons/2` 
predicates and is NOT fully tested.


## Features

Compared to other lambda libraries, the present implementation has several advantages:

- Lambda expressions are represented using a simple and natural Prolog syntax (see
below for examples using the Y combinator and compare how the relevant terms are written
in different implementations of lambda expressions).

- Lambda expressions are defined through a local anonymous predicate `_/N`
whose form is the well-known `(Head :- Body)` construct. Therefore, there is no 
need for parameter passing (e.g., using the `^` operator) and no continuations.
Note that `Body` is mandatory (it can be set to `true`, see below for an example).

- The scope of variables is easy to see: singletons are treated as globals. 
Therefore, there is no need for additional syntax (e.g., `\` and `+\` operators as
in Ulrich Nuemerkel's lambda libray or `/` and `>>` operators and `{}` construct as
in Paulo Moura's LogTalk lambdas). While it is relatively easy to force variables
to be local (see below for examples), it is currently not possible to force
non-singletons to be global (in most of such cases, global variables are instantiated; 
see below for an example).


## Known limitations

- The library is not optimized for speed. In particular, `library(lamba_abstractions)` 
is almost as fast as `library(lamba)` but somewhat slower than `library(yall)`. 
This is largely due to the use of `term_singletons/2`.


## Examples

Example 1:
```prolog
?- maplist(( (X) :- X>2 ), [3, 4, 5]).
true.
```

Example 2:
```prolog
?- maplist(( (X, Y) :- Y is X+1 ), [1, 2, 3], List).
List = [2, 3, 4] 
```

The following examples are adapted from http://www.complang.tuwien.ac.at/ulrich/Prolog-inedit/ISO-Hiord.html.

Example 3:
```prolog
?- Xs = [A, B], maplist(( (Y) :- dif(X, Y) ), Xs).
Xs = [A, B],
dif(X, A),
dif(X, B).
```

Example 4:
```prolog
?- use_module(library(clpfd)).

?- Xss = [[1, 2], [3]], maplist(maplist(( (X, Y, Z) :- X+Y#=Z )), Xss, Yss, Zss).
Xss = [[1, 2], [3]],
Yss = [[_3308, _3314], [_3326]],
Zss = [[_3350, _3356], [_3368]],
1+_3308#=_3350,
2+_3314#=_3356,
3+_3326#=_3368.

?- Xss = [[1, 2], [3]], maplist(maplist(( (X,Z) :- X+Y#=Z )), Xss, Zss).
Xss = [[1, 2], [3]],
Zss = [[_4406, _4412], [_4424]],
3+Y#=_4424,
2+Y#=_4412,
1+Y#=_4406.
```

The following examples are adapted from https://blog.logtalk.org/tag/lambdas/.

Example 5:
```prolog
?- maplist(( (A-B, B-A) :- true ), [1-a, 2-b, 3-c], Zs).
Zs = [a-1, b-2, c-3].
```

Example 6:
```prolog
?- use_module(library(clpfd)).

?- maplist(( (X, Y) :- Z#=X+Y ), Xs, Ys).
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

The following example is adapted from https://rosettacode.org/wiki/Y_combinator#Prolog.

Example 7:
```prolog
% The Y combinator
y(P, Arg, R) :-
	Pred = ((Nb2, F2) :- call(P, Nb2, F2, P)),
	call(Pred, Arg, R).

?-
    Fib = ( (N, F, P) :- (N<2 -> F=N ; N1 is N-1, N2 is N-2, call(P, N1, F1, P), call(P, N2, F2, P), F is F1+F2) ),
    y(Fib, 10, FR), format('Fib(~w) = ~w~n', [10, FR]),
    Fact = ( (N, F, P) :- (N=1 -> F=N ; N1 is N-1, call(P, N1, F1, P), F is N*F1) ),
    y(Fact, 10, FF), format('Fact(~w) = ~w~n', [10, FF]).
Fib(10) = 55
Fact(10) = 3628800
true.
```

Other examples.

Example 8:
```prolog
% side-effects

?- X = 5, ( (X, Y, Z) :- Y is X+X, Z is Y+Y, format('Double of ~w is ~w.\nDouble of ~w is ~w.\n', [X, Y, Y, Z]) ).
Double of 5 is 10. 
Double of 10 is 20.
X = 5.
```

Example 9:
==
% side-effects and nesting 

?- X = 5, ( (X, Y, Z) :- Y is X+X, format('Double of ~w is ~w.\n', [X, Y]), ( (Y, Z) :- Z is Y+Y, format('Double of ~w is ~w.\n', [Y, Z]) ) ).
Double of 5 is 10. 
Double of 10 is 20.
X = 5.
```

Example 10:
```prolog
% Forcing variables to be local using var/1 
% Example provided by Ulrich Neumerkel

?- maplist(( (p(s(_))) :- true ), [A,B]). 
A = B, B = p(s(_2634)).

?- maplist(( (p(s(X))) :- var(X) ), [A,B]). % no singletons anymore
A = p(s(_3104)),
B = p(s(_3176)).

?- maplist(( (X, Y) :- true ), [A, B], L). 
X = A, A = B,
L = [Y, Y].

?- maplist(( (X, Y) :- var(X), var(Y) ), [A, B], L).
L = [_3116, _3210].
```

Example 11:
```prolog
% Adapted from an example provided by Jan Burse

?- Y=6, maplist(( (X, H) :- H is sqrt(X*X+Y*Y) ), [1,2,3,4,5], L).
Y = 6,
L = [6.082762530298219, 6.324555320336759, 6.708203932499369, 7.211102550927978, 7.810249675906654].

?- maplist(( (X, H) :- H is sqrt(X*X+Y*Y) ), [1,2,3,4,5], L).
ERROR: Arguments are not sufficiently instantiated
```


## Download and installation

To install and use the module, type:
```prolog
?- pack_install(lambda_abstractions).
true
?- use_module(library(lambda_abstractions)).
```
from the Prolog toplevel.


Enjoy!


## Planned Improvements
Planned improvements include:
- Test suite 
- Correct `meta_predicate` declarations (to identify arity for the cross-referencer)
This issue has been raise in comp.lang.prolog by Ulrich Neumerkel:
```prolog
Also, the meta-predicate declarations do not help in identifying 
the actual arity.  Thus, cross-referencing does not work. Consider: 

p1(Xs, Ys) :- 
   maplist(inex1, Xs, Ys). 

p2(Xs, Ys) :- 
   maplist(\inex2, Xs, Ys). 

p3(Xs, Ys) :- 
   maplist((p(X,Y):-inex3(X,Y)), Xs, Ys). 

This produces the following warnings in SWI: 

Warning: The predicates below are not defined. If these are defined 
Warning: at runtime using assert/1, use :- dynamic Name/Arity. 
Warning: 
Warning: inex1/2, which is referenced by 
Warning:         /home/ulrich/SO/f1.pl:1: 2-nd clause of '__aux_maplist/3_inex1+0'/2 
Warning: inex2/2, which is referenced by 
Warning:         /home/ulrich/SO/f1.pl:5:12: 1-st clause of p2/2 

So the inexistant predicate is identified in both p1 and p2, but not 
in your case. 
```
However, the warnings could not be reproduced using SWI Prolog (threaded, 64 bits, version 7.7.21).

## Version 0.2.1
- Removed predicate name in lambda expression (i.e. anonymous predicate)
- Provided more examples on how to handle global/local variables

## Version 0.1.1 
- First public release

