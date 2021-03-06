---
layout: overview-large
title: Blackbox Vs Whitebox

disqus: true

partof: macros
num: 2
---
<span class="label warning" style="float: right;">EXPERIMENTAL</span>

**Eugene Burmako**

Separation of macros into blackbox ones and whitebox ones is implemented in the recent milestone builds of Scala 2.11, starting from 2.11.0-M7 (however, in 2.11.0-M8, the syntax underwent some changes, so this documentation isn't applicable to earlier milestone builds of 2.11). The blackbox/whitebox separation is not implemented in Scala 2.10.x or in macro paradise. Follow the instructions at [http://www.scala-lang.org/download/](http://www.scala-lang.org/download/) to download and use the latest milestone of 2.11.

## Why macros work?

With macros becoming a part of the official Scala 2.10 release, programmers in research and industry have found creative ways of using macros to address all sorts of problems, far extending our original expectations.

In fact, macros became an important part of our ecosystem so quickly that just a couple months after the release of Scala 2.10, when macros were introduced in experimental capacity, we had a Scala language team meeting and decided to standardize macros and make them a full-fledged feature of Scala by 2.12.

Macro flavors are plentiful, so we decided to carefully examine them to figure out which ones should be put in the standard. This entails answering a few important questions. Why are macros working so well? Why do people use them?

Our hypothesis is that this happens because the hard to comprehend notion of metaprogramming expressed in def macros piggybacks on the familiar concept of a typed method call. Thanks to that, the code that users write can absorb more meaning without becoming bloated or losing
compehensibility.

## Blackbox and whitebox macros

However sometimes def macros transcend the notion of "just a regular method". For example, it is possible for a macro expansion to yield an expression of a type that is more specific than the return type of a macro. In Scala 2.10, such expansion will retain its precise type as highlighted in the ["Static return type of Scala macros"](http://stackoverflow.com/questions/13669974/static-return-type-of-scala-macros) article at Stack Overflow.

This curious feature provides additional flexibility, enabling [fake type providers](http://meta.plasm.us/posts/2013/07/11/fake-type-providers-part-2/), [extended vanilla materialization](/sips/pending/source-locations.html), [fundep materialization](/overviews/macros/implicits.html#fundep_materialization) and [extractor macros](https://github.com/paulp/scala/commit/84a335916556cb0fe939d1c51f27d80d9cf980dc), but it also sacrifices clarity - both for humans and for machines.

To concretize the crucial distinction between macros that behave just like normal methods and macros that refine their return types, we introduce the notions of blackbox macros and whitebox macros. Macros that faithfully follow their type signatures are called **blackbox macros** as their implementations are irrelevant to understanding their behaviour (could be treated as black boxes). Macros that can't have precise signatures in Scala's type system are called **whitebox macros** (whitebox def macros do have signatures, but these signatures are only approximations).

We recognize the importance of both blackbox and whitebox macros, however we feel more confidence in blackbox macros, because they are easier to explain, specify and support. Therefore our plans to standardize macros in Scala 2.12 only include blackbox macros. In future releases of Scala we might make whitebox macros non-experimental as well, but it is too early to tell.

## Codifying the distinction

In the 2.11 release, we take first step of standardization by expressing the distinction between blackbox and whitebox macros in signatures of def macros, so that `scalac` can treat such macros differently. This is just a preparatory step, so both blackbox and whitebox macros remain experimental in Scala 2.11.

We express the distinction by replacing `scala.reflect.macros.Context` with `scala.reflect.macros.blackbox.Context` and `scala.reflect.macros.whitebox.Context`. If a macro impl is defined with `blackbox.Context` as its first argument, then macro defs that are using it are considered blackbox, and analogously for `whitebox.Context`. Of course, the vanilla `Context` is still there for compatibility reasons, but it issues a deprecation warning encouraging to choose between blackbox and whitebox macros.

Blackbox def macros are treated differently from def macros of Scala 2.10. The following restrictions are applied to them by the Scala typechecker:

1. When an application of a blackbox macro expands into tree `x`, the expansion is wrapped into a type ascription `(x: T)`, where `T` is the declared return type of the blackbox macro with type arguments and path dependencies applied in consistency with the particular macro application being expanded. This invalidates blackbox macros as an implementation vehicle of [type providers](http://meta.plasm.us/posts/2013/07/11/fake-type-providers-part-2/).
1. When an application of a blackbox macro still has undetermined type parameters after Scala's type inference algorithm has finished working, these type parameters are inferred forcedly, in exactly the same manner as type inference happens for normal methods. This makes it impossible for blackbox macros to influence type inference, prohibiting [fundep materialization](/overviews/macros/implicits.html#fundep_materialization).
1. When an application of a blackbox macro is used as an implicit candidate, no expansion is performed until the macro is selected as the result of the implicit search. This makes it impossible to [dynamically calculate availability of implicit macros](/sips/pending/source-locations.html).
1. When an application of a blackbox macro is used as an extractor in a pattern match, it triggers an unconditional compiler error, preventing [customizations of pattern matching](https://github.com/paulp/scala/commit/84a335916556cb0fe939d1c51f27d80d9cf980dc) implemented with macros.

Whitebox def macros work exactly like def macros used to work in Scala 2.10. No restrictions of any kind get applied, so everything that could be done with macros in 2.10 should be possible in 2.11.
