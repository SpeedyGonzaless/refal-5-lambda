This is translation of historical paper [note000.txt] from Russian.

... (not translated yet) ...

## (2.4) The lexical analysis of Modular Refal (v. 0.1.953).

Now I bring the lexical analysis method for comparison into Modular Refal
itself. In Modular Refal the lexical analysis executes the classical scheme
without using functions of the highest order. The module of lexical analysis
was written for a long time ago, at these times I had have the programming
skills only in procedural and object-oriented languages (generally with C++).
That’s why I was implementing lexical analyzer with the following scheme with
using of ADT: creates the abstract scanner type, which after receiving the
NextToken message returns the next lexeme. The scanner itself, in its turn, for
receiving the source text, creates and use another ADT – a symbol stream
(SymStream), encapsulating within the way of reading the file data and
calculating the numbers of strings.

In simplifying the working with defined sets of symbols purposes (for example,
the tail of name symbols – leters, figures, `-`, `_`, `?`, `!`), the symbols
stream class have methods for extracting the sequence of symbols from some set.
The symbol stream class itself includes the file descriptor and read file
string-by-string, as if it needed.

Such architecture decision (scanner class on demand reads the lexems, symbol
stream class on demand reads the following file strings), what differs from the
first three methods, never fully occupy memory with the file and lexical parse.
That’s why Modular Refal could quite easy (without calculation of lost time)
process the module, which includes 100 Mb of empty strings or 100 Mb of empty
strings with semicolon (what is acceptable to declare within the module) –
extra memory doesn’t required. However, this architecture is more complicated,
as if the whole source text and/or the whole lexical convolution will be
uploaded to memory.

## (2.5) Comparison with different ways.

The most convenient for me is first and third ways: first way mixing up the
sufficient presentation with the code laconicism, third way is more flexible
and high-leveled. To the lacks of first way we could carry, that it in some
grade oriented on table interpreting (for symbol search is needed
double-leveled iteration by open e-variables), what means that, maybe, less
sufficient. The lack of third way is, that code, created as a result, became
too long. The amount of sentences in each function-condition is linear related
from unity of sets power in each of alternatives. Generated code doesn’t
consist of open e-variables, that’s why supposingly it could be sufficient
enough. It could be. But because of code generator in compiler all the function
sentences processing independently, every time it needs a recognizing the same
type format `(e.Accum) '3' e.Text`. As if compiler was able to deduce general
format for several neighbor sentences and in case of samples with similar
structure the general code could be “putted out of commas”, the state machine
will be sufficient enough.

The second variant too well enough, because it’s let us cut the source code
short, putting out the general search code within set to the separate function.
But method isn’t sufficient: as we can recognize by a row of functions, sets
every time require reload, because the sets to handler functions is not
transmitted.

An method with lexical analisator generator could be mixed up with method of
lexical analysis within Modular Refal – with using the same format of
transfer table describing, it’s possible to create code, which reads lexems
from file sequentially. That’s why there possible to unite advantages of both
methods: the high-leveled lexem description and memory economy.

# \[3] Compiler features.
In this laboratory work while I were constructing compiler I check out for
myself some new tricks. In particular, had been used triple-leveled simplified
parsing algorithm and independent generation of different code elements. For
generation of separate sentences under construction was abstract algorithm
(for each sentence) like an imperative commands sequence. Later this algorithm
transformed yet to code, and the separate commands were generating almost
independent one of other.

The compiler itself left unfinished for the ready product, but as for the
prototype it’s good enough. From several possibilities, which were ought to be
realized or remake: is standard library catalogs support — into the present
version all the file pathes is setting or as absolute, or as relative from the
present folder; the choosing possibility of C like compiler – within present
version program calls program `call_cpp_compiler`, which appears to be the
.bat-like into the present folder; the support of arithmetics at least at the
intermediate level; the support of escape-sequences within the generated code
(with hex-symbols like `\xNN`); overfill checks within integer operations and
a lot of other things.

## (3.1) A brief language overview.

That dialect is practically basis REFAL subset: all the functions presenting
the pairs of sentences `Sample=Result`, terms are only atoms and regular
parenthesis structures (means, that abstract data type, which are nominated
brackets itself, there are absent). The atoms set includes the characters
within, unsigned integers from 0 to 2\*\*32−1 (this aren’t macrofigures),
functions and file descriptors, which could be created only with FOpen function.

Shortly about numbers. The integers doesn’t interpretates as macrodigits of
some limitless by length number, because of library functions Add and Sub have
format `<s.Func s.Num1 s.Num2> == s.Res`, means that it supports only twin atomic
argument (which must be the numbers) and return integer result as single
symbol. If only numbers could be interpretated as macrodigits, the library
functions will be working with rows of numbers. By the way, all the arithmetic
within this dialect is limited only with two functions Add and Sub. For
transforming the number <---> string separate primitive functions used. The
limitation is connected with the fact, that Modular Refal is too limited with
that pair of functions and while I was working on that laboratory work I didn’t
wanted to enlarge it.

The ordering function of twin symbols too haven’t presented, that’s why the
realizing of sufficient escape-consequences support is failed. However for
prototype it isn’t so important.

The representation of functional atom within the program is name of function
itself, and this name ought to be defined. The local and lambda functions
doesn’t exist within language, all the functions should be described
statically into the global scope. All the functions, which present in
translation unit (The modules in meaning of Pascal or Modula in language
doesn’t exist. The program dividing on some separate files of source texts is
corresponds with the meaning of translation unit – the relic, which at first
appeared on Fortran (language was able to support the separate translation, at
expense of what on Fortran was written a lot of libraries, some of which in
still in use nowadays) and after saved within C and C++.) divide on two
classes: local and entry-functions. Those classes corresponds functions with
static and external configuration of C\C++ languages.

> _Mazdaywik, 2018-01-18:_ yes, previous paragraph has very ugly language,
> because ugly language has original paragraph in Russian source. Thanks
> to translator (Dmitry Starchenko) for save original style of text.

For the access to functions from the different translation unit using `$EXTERN`
directive, which corresponds with same memory class with the same name in
C/C++. In REFAL programing often in use the symbol names, which have sense of
flags and tags, instead of callable functions. Many (but not REFAL-2 and Simple
Refal) have built-in support the symbol names – atoms of corresponding type is
nominated as identifiers or tags (in literature I met the different
nominations). Simple Refal doesn’t consist of such atoms type. For imitation
of them are used functions. It is obvious, that if function created for using
only as name, the function body never will be (at least mustn’t) called. Such
function could define as empty function, because an empty function with any
argument calls dump (what guarantee program stop in case of (unintentional)
function launch). For cursive writing such functions in language there is
directives `$ENUM` and `$EENUM`. Both functions receive the name list, divided with
commas. The setting up function name in list `$ENUM` is equivalent to declaration
it as empty local, the setting up in list `$EENUM` – as empty entry-function
(eenum — entry enum). For declaration such functions I didn’t use the `$EMPTY`
directive (as in REFAL-2), because _enum_ word more corresponds with their
purpose – the symbol names introduction – as transfer (enum) in such languages
as C++, C#, Visual Basic.

Functions comparing (by the repeated variables way, not in meaning of ordering
function, which is absent) by functions addresses in meaning C language, not by
names (in contrast to REFAL-2 and Refal-7, within which the names is
representate functions and these functions comparing by string representation
of their name). That’s why symbolic names, created in some translation unit
with `$EENUM` assistance, we have to import with the directive `$EXTERN` help.
However, atoms-functions, nevertheless including string name representation –
it is done in debugging purposes, because of sorting out the hexadecimal
address point of view dump extreme difficultness.

The syntax is similar with REFAL 5 syntax with the difference, that the empty
functions (such as `F {}`) is acceptable and exists `$ENUM` and `$EENUM`
directives. There is syntax in EBNF:

    TranslationUnit = Element* .

    Element =
      '$ENUM' NameList |
      '$EENUM' NameList |
      '$EXTERN' NameList |
      '$ENTRY' Function |
      Function .

    NameList = Name ',' NameList | Name ';' .
 
    Function =
      Name '{' Sentence* '}'

    Sentence = Pattern '=' Result ';' .

    Pattern = PatternTerm* .

    PatternTerm = Char | '(' Pattern ')' | Number | Name | Variable .

    Result = ResultTerm* .

    ResultTerm = PatternTerm | '<' Result '>' .

    Pattern = PatternTerm* .

I won’t be bringing the lexis, because of it’s easy to understand from the
state machine table, with which had been constructing lexical analyzer.

Some features of lexical analysis. The functions names and variables indexes
could include hyphen, which fully equivalent with underlining. The need of
replacing `-` to `_` linked with the fact, that C/C++ language didn’t support
names with hyphen, but the source texts based on REFAL looks pretty good with
usage of hyphen in names. When reading integer variable overfilling check
hadn’t started – at excess  of size 2\*\*32−1 the result appears to be
undefined.

Within language strangely enough, was no place for the global variables: I’ve
got used to program on REFAL without them, that’s why compiler complication
with useless remedy I didn’t done. In case of sudden need, I able to write on C
stack support.

## (3.2) Several words about regular functions.

It is obvious, that with REFAL remedies (identification with the sample and
result constructing) a number of operations execution is impossible: it is I/O
operations and atom operations. Of course, the arithmetic functions could be,
theoretically, described in that way:

    Add {
      0 0 = 0;
      0 1 = 1;
      1 0 = 1;
      ...
      234 456 = 690;
      ...
    }

But even if functions of such kind will be written (with auto-generation, to
example with program based on C), then practical using of them will be
complicated – they extremely insufficient. That’s why language need primitive
functions, executing primitive operations.

Within actual compiler accepted the following model. Inputing the built-in
functions to language is ideologically disgusting for me, because the functions
is related to user-level  definitions, but execute language-level actions.
Besides for the built-in functions executes special rules, for instance, they
don’t need to introduce. That’s why I made the primitive operations as external
– as in C like language — there is too absent built-in functions. Because of
being the primitive  operations (those, that impossible create with REFAL
sources) is external, than exactly user too able to write a him/herself
primitive functions based on C++ language and it too will be external.

From the standard external functions to us given the following:

* Standard tag-names: `Success`, `Fails`, `True`, `False`.

* Arithmetic:
  * `Add`, `Sub` – arithmetical operations.

* Input\output:
  * `<ReadLine> == e.Line` – reads the input from console,
  * `<WriteLine e.Line> == empty` – prints to console,
  * `<FOpen s.Mode e.Name> == s.FileHandle` – opens file with preset name
    within preset configuration: `'r'` – for reading, `'w'` – for writing.
  * `<FReadLine s.FileHandle> == s.FileHandle e.Line`,
  * `<FWriteLine s.FileHandle e.Line> == s.FileHandle`,
  * `<FClose s.FileHandle>` — close the file.

The input/output file functions made on an image of corresponding with module
FileIO functions in Modular Refal, because for developing  preprocessor as for
the primitive operations I used library functions of Modular Refal.

* Types transforming:
  * `<StrFromInt s.Int> == e.Digits` – transforming number to string,
  * 
    ```
    <IntFromStr e.Text>
    == Success s.Number e.Rest
    == Fails e.Text
    ```
    If only `e.Text` begins from the numbers chain, that chain transforms
    to number `s.Number` and returns remains as `e.Rest`. If transforming
    is impossible, function returns `Fails` and own argument.
  * `<Chr s.Num> == s.Char`, `<Ord s.Char> == s.Num` – obvious.

* OS capabilities.
  * `<System e.Command> == empty` – call this command line,
  * `<Exit s.ReturnCode>` — ends program with distribution return code,
  * `<Arg s.Number> == e.Argument` – return argument of command line with
    preset number,
  * `<ExistFile e.FileName> == True | False` – file presence check.

Also to compiler attached regular library extension LibraryEx, which fully
written on REFAL and includes such useful functions, as `Map`, `Reduce`,
`MapReduce`, `LoadFile`, `SaveFile`, and many others. I won’t bring their
description, because semantic is easy to understand from the source code.

## (3.3) Syntax analysis features.

We can recognize, that if nonterminals `PatternTerm` and `ResultTerm` will be
described in following way (with loss of balanced brackets), the grammar will
become practically regular (not by appearance, but easy to transform to
regular).

    PatternTerm = Char | '(' | ')' | Number | Name | Variable .

    ResultTerm = PatternTerm | '<' | '>' .

And describe such state machine will be much easyer (in REFAL it is surprisingly
easy to write state machine programs). After finishing the pattern and result
analysis it is possible to separately check the brackets balance in them. In
program I went much further: in separate procedure I took out the variables
check too (within pattern inacceptable the meeting of two variables with
different types (s,t or e) with the same index, in result expression can’t
exist unrepresented into the pattern variables).

This state machine worked as transducer – by the reading of separate
elements (directives `$ENUM`, `$EENUM`, `$EXTERN`, the functions beginning with
curly bracket, the separate sentence, enclosing curly bracket) momentarily
generated code (in comparison with extern – C like function representing, empty
functions generation, which ends with failure, with the different memory
classes, the non-empty function beginning, sentence processing code, function
finishing). The separate element generation was fully independent one of other,
which I got to consider while I was creating the output files. The exception
was only symbol table, which consist of functions names – for the fact check,
that all names are ought to be represented before the first executing. About
generation I will tell later.

As it has appeared, this syntax analysis dividing to the three ways (which were
carried out only by separate sentences, not by the whole transmition length)
drastically simplified the compiler structure. Such driving to the state machine
became possible in order with the fact, that terms, from which consists left
and right sentences parts, can’t include dividers (corresponding, ‘=’ and ‘;’).
If only that dialect could support such nameless or local functions as terms
( which includes within sentences, dividing with assistance of ’=’ and ’;’),
then such driving of context-free grammar to regular would be imposible.

To sum it up, practice show, that dividing on two ways the free context syntax
and context relations is very useful — the compiler structure became easyer.
At paired brackets relations check we could don’t care about unexpected lexems,
at variables presence check — about brackets paired relations.

 In that case independent-context syntax check, which easy to formalize, could
be executed automatically – it is possible to  write syntax BNF analyzer
generator, fabricating the syntax tree. Further that tree could be bypassed by
context relations check and construct abstract syntax representation.

## (3.4) Virtual REFAL-machine implementation.

The virtual REFAL-machine was created at classical scheme with point of view
usage. Point of view is the double-linked node list, each of which represents
the atom, or this or that bracket (one from `(`, `)`, `<`, `>`). Nodes includes
links to the neighbor-nodes, type field and info field, which is union of
several fields with different types. Numbers-nodes in info-field include
`unsigned long`, symbol-nodes – `char`, function-nodes includes function
pointer and `const char` pointer, including function name. Nodes, which
corresponds to structural brackets, includes the linked brackets relations.
Call brackets pointers situated in calls stack in that order, in which they got
to execute.  If function call is replacing with active expression with the
different evaluation brackets, then evaluation brackets will be putted on top
of stack in proper order – so realizing saving stack invariant. For
implementation of the stack is used info field in brackets nodes (as in REFAL
2).

C++ function, corresponding to REFAL function, when performing it could or
replace the own call within the point of view, having returned refalrts::
cSuccess, or end with failure because of recognition impossibility or lack of
memory, having returned refalrts::cRecognitionImpossible; in the end of
function.

A function is composed of handlers of separated sentences, each of can ends
the function successfully (perform `return refalrts::cSuccess`) or message
of lack of memory (perform `return refalrts::cNoMemory`). If handler don’t end
function with one of such messages, control passes to next handler, in case of
end handler — to statement `return refalrts::cRecognitionImpossible`) at end
of the function.

The executing of sentence processor passes through three phases:

1. Argument recognition. At the argument recognition process argument remains
   without change. If recognition is impossible, the exit procedure executes.  
2. Variables replicating and allocating the result elements, which preset
   by literals, including structural and functional brackets (the ready brackets
   and atoms from the pattern are not in use – it’s done in compiler simplifying
   purpose). At this stage function could produce message `refalrts::NoMemory`.
   New elements allocating within the free blocks list (see below). The function
   argument at abnormal termination doesn’t changes.
3. Building of result from the pattern variables and elements, result
   constructing goes with the assistance of operations on double-linked lists,
   which doesn’t violate invariant – the part of a row transmitting with
   exception from the source splicing. After result constructing all what left
   within the pattern is moving to the free blocks list. All the operations in
   that phase can’t fail – that phase ends with the `refalrts::cSuccess`
   returning.

   For the memory optimization there in use the double-linked list of free
   nodes. All the operations, giving memory (copying the variables or
   constructing new nodes), creates own argument within that list. Later the
   list parts with rebuilded elements transmits into the field of view. Such
   strategy provides consistency of both lists at raising a message 
   `refalrts::cNoMemory` – created elements remain in free blocks list,
   argument doesn’t change.

## (3.5) Sentences generation.

Sentences  generation proceed in two stages: from the very beginning by
intermediate representation creates processor based on abstract imperative
language (means, that sentence transforms to the sequence of procession
commands), and then processor rebuilds from abstract imperative representation
to C++, and the separate commands trasmiting practically independently one of
other. The last two phases operations: giving new memory and result fabrication
realizing not so difficult, but the pattern recognition is very interesting.

I bring rules of  pattern matching so, as it shown in REFAL-5 guide, and then
comment, how they realized in real compiler.

### General requirements to mapping P on E (matching E : P)

1. If a node N2 is positioned in P to the right of a node N1, then the
   projection of N2 in E can either coincide with, or be positioned to the
   right of, the projection of N1 (projection lines cannot cross).
2. Projections of brackets and symbols must be identical to themselves.
3. Projections of variables must meet the syntax requirements of their values;
   i.e., to be symbols, terms, or arbitrary expressions for s-, t-, and
   e-variables, respectively. Different entries of the same variable must have
   equal projections.

> Dmitry Starchenko’s independed translation:
> ### The general requirements for P at E representing (comparing E : P)
>
> 1. If node N2 situated in P at the right side of node N1, the projection N2
>    in E could or compare with N1 projection, or be situated from the right side
>    of it (projection lines can’t be crossed).
> 2. Brackets and symbols should be the same with their projections.
> 3. Variables projections ought to compare with  syntax requirements ; means,
>    be the symbols, terms or free expressions for s-,t- and e- variables. The
>    different inputs of the same variable should have same projections.

> *Translation to English of this hunk of historical paper is prepared by*
> **Starchenko Dmitry <starchenko_dmitry@mail.ru>** _at 2018-01-07_

... (not translated yet) ...