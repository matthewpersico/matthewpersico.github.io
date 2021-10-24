# A Response to "13 reasons why you should not use Perl"

When I first read
https://dev.to/parvezmrobin/13-reasons-why-you-should-not-use-perl-20d5, I was
mad. Yet Another Perl Hater, one who apparently didn't use the language outside
of academia. My first instinct was to write a scathing and insulting
rebuttal. I love Perl. I've been using it to feed my family since 1997 (and
trust me, if you looked at me, I am not starving).

However, during the writing of said rebuttal, I not only learned some things
that I didn't know, but got tripped up on a rather arcane issue that would
completely confuse a new user of the language.

So, what I've finally settled on doing is just explaining why Perl is the way
it is.  If after this explanation, you are unconvinced and you still don't like
Perl, so be it.  But at least, instead of saying "Perl sucks", you'll be able
to say "Ah, now I see *why* Perl does this. It still sucks."

Two notes before we begin:

* There may still be a bit of snark in what follows; you can take the boy out
of The Bronx...

* And although I am using Python as counterpoint, this is not an anti-Python
article; it's simply the most similar language I know to be able compare to and
contrast against.

## Weird variable access

### Quote
"I need to prefix a variable by either $ or @ or % to tell Perl about the type
of variable. And I need to do it every time I access the variable."

### Discussion
Addressing these concerns in reverse order, well, yeah, it's not a typed
language; of course you are going to need to be consistent in the syntax of the
language across all references.

As for sigils (that's the official name for $, @, and %), they serve *two*
purposes: they not only tell the interpreter what kind of variable it is
dealing with, they also tell *you* what kind of variable you are dealing with.

When I see `foo` in most languages, I have *no clue* what type of data that
is. None. Not without scrolling back through the code to find a prior usage or
definition. Not without hovering the cursor in an IDE.

In Perl:

* `$foo` is a single value. String, number, reference. Now, that begs the
question "reference to what", which puts us back in the "scrolling and
hovering" lookup phase, but it's more information that we had without the
sigil.

* `@foo` is an array. '@' kind of looks like an 'a'.

* `%foo` is a dictionary. We call it a 'hash', after the hashing function for
the keys. If you squint hard enough, the '%' looks like a stylized 'h'. Also,
you could think of it as a stylized key/value representation; but I digress.

What you must remember is that the creator of Perl, Larry Wall, was not a
CompSci major; he was a linguist. Therefore, you are going to see punctuation
used to provide information and you are going to get what you initially
perceive to be strange changes in behavior, but are actually expressions of
"context"; words mean different things in different contexts in human
languages. That concept extends to Perl.

In this context, the sigil indicates data type.

### Quote

"Even more - the prefix changes time to time based on the usage."

### Discussion

Yes. In Perl sigils will change from what you would instinctually expect them
to be. This is probably one of the top two issues new learners have with Perl
(the other being regular expressions, but we'll get to that later). In fact,
our sister language Raku (formerly known as Perl6) changed this; a room full of
Perl experts created a language with *invariant* sigils. Maybe that's telling;
I don't know. But Perl's invariance is not as confusing as people make it out
to be. It is, in fact, rather useful.

Simply put, in the context of an indexed access to a portion of an array or
hash, the sigil tells you what type of data to expect in return after applying
any referencing information; in other words, it's a "cast". Examples:

To access the 6th element of the array `@foo=(10, 20, 30, 40, 50, 60, 70, 80)`, you write
`$foo[5]` (zero-based arrays). Why the '$' and not the '@'? Well:

```
   $   foo   [5]
   ^    ^     ^
   |    |     indexing expression (referencing an index in the array)
   |    variable name
   result is a scalar: 60
```

Given that we can use the `[]` to identify `foo` as an array, we are free to
use the sigil in order to determine what data is returned. In this case, we are
returning a single value. If we wanted to return the entire `@foo` array:

```
   @   foo
   ^    ^
   |    variable name
   result is the array, in the form of a list (10, 20, 30, 40, 50, 60, 70, 80)
```

Notice how a cast devolves into the simple regular variable access. It may be
unusual, but it is consistent.

For hash `%foo = ( 'a' => 1, 'b' => 2, 'c' => 3)`, the value of key `c` is
accessed with `$foo{'c'}`:

```
   $   foo   {'c'}
   ^    ^      ^
   |    |      indexing expression (referencing an key in the hash)
   |    variable name
   result is a scalar: 3
```

For the entire hash:

```
   %   foo
   ^    ^
   |    variable name
   result is the hash, in the form of a list ('a', 1, 'b', 2, 'c', 3)
```

Again, the notice how the cast devolves into the simple regular variable
access. It may be unusual, but it is consistent.

Now the curious reader may have noticed that although I have accessed members
of the array and hash with different characters ('[]' and '{}', respectively),
I have initialized the array and the hash with the same structure: a
parenthesized list. That is the way we do it in Perl. We identify the variable
with a sigil, which drives the interpretation of a list. This is a good example
of "context" in Perl.

Which leads us to our final examples for this topic: slices.

In Python `foo[2:4]` is an array slice. In Perl, we can write that as:

```
   @   foo   [2..4]
   ^    ^     ^
   |    |     indexing expression (referencing a number of indicies in the array)
   |    variable name
   result is a value list: (20, 30)
```

If we want the indicies with that:

```
   %   foo   [2..4]
   ^    ^     ^
   |    |     indexing expression (referencing a number of indicies in the array)
   |    variable name
   result is a key/value list: (2, 20, 3, 30)
```

See? Because we have a context where we have an indexing expression that can
identify the datatype, we are free to use the sigil as a cast.

Similarly with hashes:

```
   @   foo   {'a', 'c'}
   ^    ^     ^
   |    |     indexing expression (referencing a number of keys in the hash)
   |    variable name
   result is a value list: (1, 3)
```

If we want the indicies (keys) with that:

```
   %   foo   {'a', 'c'}
   ^    ^     ^
   |    |     indexing expression (referencing a number of keys in the hash)
   |    variable name
   result is a key/value list: ('a', 1, 'c', 3)
```

It's very consistent: '@' is a values list, '%' is a key/value list. They refer
to either an entire variable or are used as a cast when referring to a slice of
some variable.

Oh and one more thing about Perl slices: the value to use for a slice
designation is just a list. In other words, this Just Works:

```
   @   foo   [4, 1, 2, 6]
   ^    ^     ^
   |    |     indexing expression (referencing a number of indicies in the array)
   |    variable name
   result is a value list: (50, 20, 30, 70)
```

You not constrained to an ordered list of indicies. The same goes for hashes:

```
   @   foo   {'c', 'a'}
   ^    ^     ^
   |    |     indexing expression (referencing a number of keys in the hash)
   |    variable name
   result is a value list: (3, 1)
```

It's all internally consistent.

## Too much implicit code
## Lacking readability

### Discussion

The same example is used for both:

```
while (<>) {
    print;
}
```

Well that's a pretty short snippet. But yes, it is idiomatic; remember, Perl
was created by a linguist. There will be idioms all over the place. You learn
them and move on.

## No clear parameter list
## Weirdly passing arrays to function

### Discussion

In a rather large project, parameter declarations are important to prevent all
sorts of errors that can result from mismatched parameter lists.

Perl *technically* has no official core way to specify a parameter
list. However, there is an implementation documented here:
(https://perldoc.perl.org/perlsub#Signatures) that has been in Perl since
version 5.20. It is marked as "experimental", but it is unlikely to be modified
at this point.

If you are paranoid about using something marked "experimental", there are a
number of modules on CPAN that you can use. A good reference would be
https://phoenixtrap.com/2021/01/27/better-perl-with-subroutine-signatures-and-type-validation/.

But let's discuss the example. This example is not canonical. This is not 'Perlish':

```
sub func1 {
    my $x = shift;
    my $y = shift;
    my $z = shift;
    // do something with the variables
}

func1(1, 2, 3);
```

A seasoned 'Perler' would write:

```
sub func1 {
    my ($x, $y, $z) = @_; # no need to shift;
    // do something with the variables
}
```

Presumably you would use more descriptive names than just 'x', 'y' and 'z'. And
if you are going to have more than three params, you should probably name them:

```
sub func1 {
    my %args = (
        start => undef,
        length => undef,
        step => 'a default value',
        reverse => 0,
        @_
        );
    for (my $i=$args{'start'};
         $i < $args{'length'};
         $i += $args{'step'} ) {
    ...
    }
    if ( $args{'reverse'} ) {
    ...
    }
}
```

Yes, the statement that assigns `%args` looks weird. Just remember:

* a  parenthesized set of values is a list.

* A list is assigned to a hash in key/value pairs.

* Keys are unique in a hash, so that multiple assignments to the same hash
  ovewrite the value. Last one in wins.

So, we create a list consisting of the expected params with default values,
followed by the actual params (`@_` is where Perl puts function params,
localized to each function). Any key value pairs in `@_` overwrite the
defaults. The defaults tell you what params the function takes. And we assign
the list to a hash.

So, why are we copying these params in the first place? There are two reasons:

The first is for clarity. Descriptive variable names are easier to grok than
`$_[0]` (remember, `_` is the name, `@` says it's an array, `$` on an index
expression says we are getting a scalar back). And copying to a hash is the
only way to use `@_` in a named parameters mode.

But the second reason for copying is that `@_` is not a copy of the values from
the function call; it is an *alias* to those params. That means if you change
the value of `$_[x]` then you have changed it *in the caller*. Sometimes, you
will see, in small or hot functions, that the `$_[x]` form is used so as to not
waste time copying data. That works fine, until someone decides to take
advantage of the aliasing and modify the data in the function, and then someone
else comes along, doesn't read the docs, and passes in a constant to the
function instead of a variable. If you use the `$_[x]` form, you have forced
your params to be read-only. This is probably what you want and expect 99.9% of
the time anyway.

So we can have named params and default params. Nice that Perl flattens lists.

But flattening makes it harder to pass arrays intact, no?

### Quote:

If you want to pass an array as an array, you need to make a reference of the
array, pass the reference and de-reference it inside the function. Hell of a
job for such a simple task!

### Discussion

Not really. Here's the code in question:

```
sub func1 { print scalar(@_); }

@a = (1, 2, 3);
@b = (4, 5, 6);
func1(@a, @b);

6
```

The first problem is with `func1`. Even if Perl "Did The Right Thing" and
passed arrays unflattened, `func1` would end up passing both arrays to
`scalar`. But that is not how `scalar` works; it returns the size of *one*
array. So we will need to recode `func1`.

The second problem is that, yes, Perl will flatten arrays by default. So, if
you want to pass them as separate arrays, pass references.

In sum:
```
sub func1 {
    for (@_) {
        print scalar(@{$_});
    }
}

@a = (1, 2, 3);
@b = (4, 5, 6);
func1(\@a, \@b);

33
```

## Weirder passing hash/map to function

### Quote

What do you think the former code [sic: see below] prints? My guess was it will
print the keys of the map. At the worst case, it will print the values
instead. Alas! The actual worst case worse than what I anticipated. It prints
both keys and values.

### Code
```
sub func1 { print join ', ', @_; }

%f = (one => 1, two => 2);
func %f;
```

### Discussion

As for the original guess, I don't think it's reasonable to expect that passing
a full dictionary to a function will have any effect other than processing the
full dictionary. There's nothing there that says to pick out the keys or the
values.

So, let's correct the function to do what was expected:

```
## We use multiple lines for examples in order to improve legibility

sub func1 {
    my %h = @_;  # Assign to a hash, so we can use hash processing.

    for (keys (%h)) {
        say join(', ', $_, $h{$_}); # In modern (5.10+) versions of Perl,
                                    # say is available, which automatically adds
                                    # one new line to the end of the output
    }
}

%f = (one => 1, two => 2);
func %f;
two, 2
one, 1
```

*Note:* Some of you that run this may see the output

```
one, 1
two, 2
```

Hashes are, by definition, a non-ordered data structure and the interpreter is
free to deliver the elements of a hash in any order it feels like. In fact a number
of changes have been made over the years to make hash key order less
predictable and more random. This has been done for security reasons that are
beyond the scope of this article. Feel free to search for `perl hash security`
for more details.

## Cryptic RegEx handling

### Quote

When you do a regular expression match, it automatically assigns some
variables like $&, $`, $' etc. They are useful when you know them to be
honest. But, when you just read the code, specially when you are from another
background, it looks like a mess.

Let's unpack some of that:

* "useful when you know them, to be honest."  As with most things.

* "it automatically assigns some variables like $&, $`, $' etc." So what? If
  you don't need them, you don't use them. I can honestly tell you that in all
  my years of Perl-ing, I rarely use these; at this moment I need to look them
  up to even figure out what they mean.

Let's be honest; the main reason regexps have gotten a bad rep is that their
dense syntax is lumped in with Perl's syntax, so now both are declared "bad".

But I say no; Perl is the only language I know that gets it right. Why? Because
Perl's regular expressions are *built in*. People complain that function
parameters are not built into Perl, but having regexps built in is considered
harmful. Really? Let's have a look:

```python
import re
p = re.compile('[a-z]+', re.IGNORECASE)
m = p.match('TEMPO')
print('>',m.group())
>TEMPO
```

```perl
'TEMPO' =~ m/([a-z]+)/i;
say(">$1");
TEMPO
```

What it comes down to is that having regular expressions built into the
language leads to them being used much more than in any other language, and
certainly, that increases the number of Jurassic[^1] uses.

But the basic difference here is that, as an add-on, regexps in most other
languages require explicit function calls to get access to the results, whereas
in Perl you get those results with symbols. The actual regular expression
languages themselves are pretty much equivalent. So it's a matter of taste. For
me, I prefer brevity and the ability to write expressions to get at the results
without all those function calls all over the place[^2].

## Less Searchability

### Quote

Consider the previous point. You are reading a perl program. You suddenly found
use of undeclared variables like $& or $'. So, you googled about it - "what
does $' means in perl". Alas! while you search something on the search engines,
they generally stripes out spaces and punctuation. You will still have relevant
results. But, they won't be as great as if you search - "what does dir mean in
python".

### Discussion

This is an education issue. You don't need to use Google for basic Perl
research. Perl has a very rich set of documentation. (See
https://phoenixtrap.com/2021/10/12/read-the-fantastic-manual-how-to-get-the-most-out-of-perl-documentation).
If you type `perldoc` at a prompt, you get:

```
Usage: perldoc [-hVriDtumUFXlT] [-n nroffer_program]
    [-d output_filename] [-o output_format] [-M FormatterModule]
    [-w formatter_option:option_value] [-L translation_code]
    PageName|ModuleName|ProgramName

Examples:

    perldoc -f PerlFunc
    perldoc -q FAQKeywords
    perldoc -v PerlVar
    perldoc -a PerlAPI

The -h option prints more help.  Also try "perldoc perldoc" to get
acquainted with the system.                        [Perldoc v3.2801]
```

Notice the `perldoc -v PerlVar` line? Well:

```
$ perldoc -v '$`'

    $PREMATCH
    $`      The string preceding whatever was matched by the last successful
            pattern match, not counting any matches hidden within a BLOCK or
            "eval" enclosed by the current BLOCK.

            See "Performance issues" above for the serious performance
            implications of using this variable (even once) in your code.

            This variable is read-only and dynamically-scoped.

            Mnemonic: "`" often precedes a quoted string.
```

Bingo. No Google needed. If `perldoc` is not installed on your machine, use
`https://perldoc.perl.org/`. Typing "$\`" into the search box at
`https://perldoc.perl.org/` immediately yields
https://perldoc.perl.org/variables/$%60.

We know that there is a 'pythonic' way of doing things. Well one of the
'perlish' ways of doing things is to use the provided documentation instead of
randomly searching the web.

## Pathetic debugging support

### Discussion

I'll let you be the judge:

Python: https://docs.python.org/3/library/pdb.html

Perl: https://perldoc.perl.org/perldebug

There is also a visual Perl debugger written *in Perl*, requiring nothing but
Perl and the Tk module (tkinter is the Python equivalent): https://metacpan.org/pod/Devel::ptkdb

There are plenty of visual debuggers for Python built into IDEs, but I have not
yet seen a pure Python/tkinter one.

## Too much implicit error handling

### Quote

While the shown error messages are cryptic, there are a lot of errors that are
not shown at all. Say, we have an array @a and we want to print it. Now,
mistakenly we have typed print $a. It will just print nothing just like if the
array was empty. In my opinion it either should through an error or print the
memory address of @a. It does nothing. And you will never know that you have an
error there.

### Discussion

Ok, you got us here. This one turns out to be very bizarre.

Let's start by saying that even when playing with snippets, one should always
use `strict` and `warnings`. Let's do that for an example that matches the
complaint:

```
$ cat tmieh.pl

use strict;
use warnings;

my @ar = ( 1,2,3);
print $ar;

$ perl tmieh.pl
Global symbol "$ar" requires explicit package name (did you forget to declare "my $ar"?) at tmieh.pl line 5.
Execution of tmieh.pl aborted due to compilation errors.
```

Remember, `@ar` and `$ar` are not the same variable and are not related to each
other at all.  So yes, I could declare triumph here and say "use strict and use
warnings" to catch those kinds of errors in real code. But I'd be hiding a
different issue.

You'll noticed I used 'ar' and not 'a' for the variable name. That shouldn't
make a difference, right?  Well...

```
$ cat tmieh2.pl

use strict;
use warnings;

my @a = ( 1,2,3);
print $a;

$ perl tmieh2.pl
Use of uninitialized value $a in print at tmieh2.pl line 5.
```

Huh? I used `strict` and `warnings`. How did this even run without a `my $a;`
declaration? Remember `$a` and `@a` are two different variables.

Well, it turns out that `$a` and `$b` are *special* in Perl. If you search for
`$a` at https://perldoc.perl.org, you get
https://perldoc.perl.org/variables/$a:

```
$a
$b

Special package variables when using sort(), see "sort" in perlfunc. Because of
this specialness $a and $b don't need to be declared (using use vars, or our())
even when using the strict 'vars' pragma. Don't lexicalize them with my $a or
my $b if you want to be able to use them in the sort() comparison block or
function.
```

`$a` and `$b` are effectively already pre-declared. That's why we didn't get a
'not defined' error to stop execution. In addition to the admonition to not
lexicallize them (don't say `my $a` or `my $b` anywhere), you should simply not
use either outside of a `sort` context.

So here, we have a confusion in the difference between `$a` and `@a` which is
made even cloudier by the specialness of `$a`. I'll need to concede that this
is the kind of thing that would frustrate a beginner. I mean, it took me a
minute or two of head scratching before I remembered that `$a` was "special".

But that's why we provide `strict` and `warnings` and that's why we say to use
descriptive names; that's what `foo` and `bar` are for.

## Inconsistent behavior

### Quote

Say, we have a function like this

```
sub one {
    return 'two';
}
```

Now we define two maps like the following

```
%m1 = (one => 1);
%m2 = (1 => one);
print %m1;
print %m2;
```

It will print one1 and 1two. Can you make any sense of it? Well, I could not
for a while. Turns out if your map's value coincides with one of the functions,
it will call that function and use the function's return value as the map's
value. But, it will not do such if your map's key coincides with a function.

### Discussion

Not exactly. First of all, let's rewrite the example in an equivalent form:

```
%m1 = ('one', 1);
%m2 = ('1', one);
print %m1;
print %m2;
```

You see, `=>` is called a 'fat comma' and has two properties:

* it is a comma, so it separates two language tokens

* it treats the left hand side of its expression as a string, as if it was
  quoted. Notice the example with the fat comma does not have quoted lefthand
  values.

With it being rewritten, we can see what is happening here:

* `%m1` has a key of the string `'one'` and a value the integer `1`.

* `%m2` has a key of the string `'1'` and a value of the function call `one()`,
  which is `'two'`. Yes, you don't need the parens to invoke a function call.

The stringification of the left hand side is very convenient when typing out a
long named argument list or a very large hash assignment.

## Poor IDE support

### Discussion

I can't argue this one either way. I've been using emacs as my editor and
https://metacpan.org/pod/Devel::ptkdb for years, so I don't have need an
IDE. But in a world where new programers live in IDEs, then Perl suffers from
circular processing: those of use who have been using Perl for a long time don't
need IDEs so we don't write them for new learners and so new learners using
those IDEs don't have a good Perl environment to learn from, so they don't.

## I just hate it

### Discussion

I just love it.

For me the expressiveness available in the brevity that is allowed is what
appeals to me. Who wants to type all those function calls?

Plus, you can bend it in ways that would be difficult in other languages. For
example, I am unfortunately spending a lot of time at work converting Perl
systems to Python. But by using `AUTOLOAD` functionality and the
`Inline::Python` module, I am able to convert a Perl system of multiple modules
into Python one module at a time, having the Perl talk to the Python
replacements *without changing the Perl calling code at all*. I can install the
Perl and Python side by side, choosing which implementation I want to use *at
runtime*, so I can switch back if there's an error. Once all the modules are
converted, then I can convert the main to Python and the job is accomplished.

I've used Perl to run client reporting and financial performance systems for a
company that had nine figures of assets under management. I've used it to run
company-wide build systems in multiple companies. I've used to help me organize
my CD collection and simplify printing labels for Christmas cards. Once of it's
nicknames is Swiss Army Chainsaw.

Its reputation may suffer from bad code in the 90's, but the fact that so many
untrained people could use it at all is a testament to its utility.

I hope that I've been able to explain and enlighten. Perl may not be as popular
as it once was, but it's still as good as it ever was.

[^1]: "Just because we can, doesn't mean we should."

[^2]: Yeah. Pretty ironic; I know. The Python folks would say, "We'd like to
get at our data without all that punctuation getting in the way."
