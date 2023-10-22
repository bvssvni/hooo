# Hooo
Propositional logic with exponentials

To install Hooo, type:

```text
cargo install --example hooo hooo
```

Then, to check a file with Hooo, type:

```text
hooo <file.hooo>
```

### Example: Absurd

```text
fn absurd false -> a {
    x : false;
    let r = match x : a;
    return r;
}
```

### Working with projects

To check a whole project, type:

```text
hooo <project directory>
```

Hooo will generate a "Hooo.config" file, which you can modify to use other libraries:

```text
dependencies {
    path("../my_library");
}
```

By default, Hooo adds a "std" library which you can use in the following way:

```text
use std::not_double;
```

This will add a term `not_double : a -> !!a`.

## Introduction to Hooo

Intuitionistic Propositional Logic (IPL) is the most important mathematical language
in the world, because it is the foundation for constructive logic and many type systems.

Usually, IPL is thought of as a "simple language" that is generalized in various ways.
For example, by adding predicates, one gets First Order Logic.

Previously, IPL was thought of as "complete" in the sense that it can derive every
formula that is true about propositions.
To reason about IPL at meta-level, mathematicians relied on some meta-language (e.g. Sequent Calculus)
or some modal logic.

For example, in Provability Logic, `□(a => b)` is introduced by proving `⊢ a => b` in Sequent Calculus.

Recently, I discovered that, while logical implication (`=>`) in IPL corresponds to lambda/closures,
there is no possible way to express the analogue of function pointers (`->`).
People thought previously that, since logical implication is a kind of exponential object,
that IPL covered exponentials in the sense of Category Theory.
However, there can be more than one kind of exponential!
The extension of IPL to include function pointers is called "exponential propositions".

Exponential propositions allows a unification of the meta-language of IPL with its object-language.
This means that IPL in its previous form is "incomplete", in the sense that there are no ways
to express exponential propositions.

Hooo finalizes intuitionistic logic by introducing exponential propositions (HOOO EP).
This solves the previous problems of using a separate meta-language.
Inference rules in Hooo are first-class citizens.

There is no separation between the language and meta-language in Hooo.
All types, including rules, are constructive propositions.

### Relation to Provability Logic

HOOO EP might sound like Provability Logic, but it is not the same thing.
Provability Logic is incompatible with HOOO EP, since Löb's axiom is absurd.
For proof, see `lob_absurd` in "source/std/modal.hooo".

# HOOO EP

"HOOO EP" stands for "Higher Order Operator Overloading Exponential Propositions".

HOOO EP was first developed in the [Prop](https://github.com/advancedresearch/prop) library,
which exploited function pointers in Rust's type system.

There are 3 axioms in HOOO EP:

```text
pow_lift : a^b -> (a^b)^c
tauto_hooo_imply : (a => b)^c -> (a^c => b^c)^true
tauto_hooo_or : (a | b)^c -> (a^c | b^c)^true
```

The philosophy of HOOO EP is that the axioms are intuitive.
This is how people can know that the axioms can be trusted.

From these axioms, there are infinitely complex logical consequences.
It is important to keep the axioms few and simple to not cause trouble later on.

However, some statements many people find "obviously true" can be difficult to prove.
This is why a library is needed to show how to prove such statements.

For example, in Hooo, one can prove that function composition is possible:

```text
fn pow_transitivity b^a & c^b -> c^a {
    x : b^a;
    y : c^b;

    fn f a -> ((b^a & c^b) => c) {
        x : a;

        lam g (b^a & c^b) => c {
            y : b^a;
            z : c^b;

            let x2 = y(x) : b;
            let x3 = z(x2) : c;
            return x3;
        }
        return g;
    }

    use hooo_imply;
    let x2 = hooo_imply(f) : (b^a & c^b)^a => c^a;

    use pow_lift;
    let x3 = pow_lift(x) : (b^a)^a;
    let y3 = pow_lift(y) : (c^b)^a;

    use refl;
    let x4 = refl(x3, y3) : (b^a)^a & (c^b)^a;

    use hooo_rev_and;
    let x5 = hooo_rev_and(x4) : (b^a & c^b)^a;

    let x6 = x2(x5) : c^a;
    return x6;
}
```

Notice how complex this proof is compared to proving lambda/closure composition:

```text
fn imply_transitivity (a => b) & (b => c) -> (a => c) {
    x : a => b;
    y : b => c;
    lam f (a => c) {
        arg : a;
        let x2 = x(arg) : b;
        let x3 = y(x2) : c;
        return x3;
    }
    return f;
}
```

Intuitively, function composition is possible, so most people just take it for granted.
Previously, mathematicians needed a meta-language to prove such properties.
Now, people can prove these properties directly using Hooo-like languages.

Hooo is designed for meta-theorem proving in constructive/intuitionistic logic.
This means that Hooo can reason about its own rules.
From existing rules, you can generate new rules, by leveraging the full
power of meta-theorem proving in constructive logic.

An exponential proposition is a function pointer, or an inference rule.
You can write the type of function pointers as `a -> b` or `b^a`.

## Syntax

```text
a -> b     Exponential/function pointer/inference rule
b^a        Exponential/function pointer/inference rule
a => b     Imply (lambda/closure)
a & b      And (struct type)
a | b      Or (enum type)
!a         Not (sugar for `a => false`)
a == b     Equal (sugar for `(a => b) & (b => a)`)
a =^= b    Pow equal (sugar for `b^a & a^b`)
excm(a)    Excluded middle (sugar for `a | !a`)
sd(a, b)   Symbolic distinction (see section [Symbolic distinction])
true       True (unit type)
false      False (empty type)
all(a)     Lifts `a` to matching all types
foo'       Symbol `foo`
foo'(a)    Apply symbol `foo` to `a`

x : a      Premise/argument
let y      Theorem/variable

return x   Helps the solver make a conclusion

axiom foo : a               Introduce axiom `foo` of type `a`
() : a                      Prove `a`, e.g. `() : true`
f(x)                        Apply one argument `x` to `f`
f(x, y, ...)                Apply multiple arguments
match x : a                 If `x : false`
match x (l, r) : c          If `x : (a | b)`, `l : a => c` and `r : b => c`
use <tactic>                Imports tactic
fn <name> a -> b { ... }    Function
lam <name> a => b { ... }   Lambda/closure
```

The `^` operator has high precedence, while `->` has low precedence.

E.g. `b^a => b` is parsed as `(b^a) => b`.
`b -> a => b` is parsed as `b -> (a => b)`.

### Symbols

The current version of Hooo uses ad-hoc symbols.
This means that instead of declaring data structures
or predicates, one can just use e.g. `foo'(a, b)`.

An explicit symbol `foo` is written `foo'`.

Symbols are global, so `foo'` is `foo'` everywhere.

### Congruence

An operator `f` is normal congruent when:

```text
a == b  ->  f(a) == f(b)
```

An operator `f` is tautological congruent when:

```text
(a == b)^true -> f(a) == f(b)
```

Most operators are normal congruent.
Some theories treat all operators within the theory
as normal congruent, e.g. predicates in First Order Logic.

However, in [Path Semantics](https://github.com/advancedresearch/path_semantics/tree/master) there
are some operators which are tautological congruent.

For example, path semantical quality and symbolic distinction are tautological congruent.

### Symbolic distinction

Paper: [Symbolic distinction](https://github.com/advancedresearch/path_semantics/blob/master/papers-wip2/symbolic-distinction.pdf).

In normal logic, there is no way to distingish propositions from each other
without adding explicit assumptions.
With other words, logic is not allowed to know internally
the accurate representation of data.
This is because logic is used to reason hypothetically.

For example, you know that the symbol `x'` is identical to `x'`.
Yet, logic can not know this internally, because it would lead to unsoundness.
If one has two indistinct symbols which are non-equal, then this would be unsound in logic.

You must always be allowed to treat propositions as equal,
although they are strictly symbolic distinct:

- `a == b` (normal equality)
- `(a == b)^true` (tautological equality)

There should be no stronger notion of equality in logic than tautological equality.
Most operators are congruent by normal equality,
but a few operators are only congruent by tautological equality.

However, there no problems in logic by using symbolic distinction.

- `sd(a, b)` (`a` and `b` are symbolic distinct)

For example, in Hooo, one can distinguish two symbols `foo'` and `bar'`:

`let x = () : sd(foo', bar');`

This is useful when you are working with theories that need
some form of uniqueness.

For example, in Type Theory, a member of a type is
only allowed to have a unique type.
When we work with Type Theory in logic,
we are allowed to assume that two types are equal,
but the theory decides whether this is permitted.
The theory can not prevent us from making assumptions,
but it can control the consequences.
Without symbolic distinction, there is no way to tell
in logic that two types are not permitted to be equal.

Symbolic distinction allows you to add axioms for such cases
and still be able to reason hypothetically.
