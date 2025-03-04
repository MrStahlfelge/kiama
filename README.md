
# Kiama

[![License: MPL v2.0](https://img.shields.io/badge/License-MPL%20v2-blue.svg)](http://mozilla.org/MPL/2.0/)
[![Sonatype Nexus (Releases)](https://img.shields.io/nexus/r/https/oss.sonatype.org/org.bitbucket.inkytonik.kiama/kiama_2.12.svg?label=Sonatype%20Nexus%20Release)](https://search.maven.org/#search%7Cga%7C1%7Cg%3A%22org.bitbucket.inkytonik.kiama%22)
[![Sonatype Nexus (Snapshots)](https://img.shields.io/nexus/s/https/oss.sonatype.org/org.bitbucket.inkytonik.kiama/kiama_2.12.svg?label=Sonatype%20Nexus%20Snapshot)]()
![Test](https://github.com/inkytonik/kiama/workflows/Test/badge.svg)

Kiama is a Scala library for language processing.
In the Kiama project we are investigating embedding of language processing formalisms such as grammars, parsers, rewriters and analysers into general-purpose programming languages.
Learn about Kiama by reading our [research papers](https://github.com/inkytonik/kiama/blob/master/wiki/Research.md) starting with [Lightweight Language Processing in Kiama](https://doi.org/10.1007/978-3-642-18023-1_12).

IMPORTANT NOTE: Kiama is a research project, so many details will change. Consult with us before you rely on it for serious work. We make no guarantees about the features or performance of the Kiama library if you do choose to use it.

## Contact

Tony Sloane

inkytonik@gmail.com

https://github.com/inkytonik

## Latest News

 * Dec 23, 2020: Version 2.4.0 released
 * Dec 18, 2019: Version 2.3.0 released
 * Apr 26, 2019: Version 2.2.1 released
 * Jun 23, 2018: Move Kiama source repo to Git
 * Apr 11, 2018: Version 2.2.0 released
 * Jun 19, 2017: Version 2.1.0 released
 * Apr 6, 2016: Version 2.0.0 released
 * Jun 24, 2015: moved project to BitBucket
 * Nov 10, 2014: Version 1.8.0 released
 * Aug 11, 2014: Version 1.7.0 released
 * May 16, 2014: Version 1.6.0 released
 * Apr 21, 2014: Version 1.5.3 released

## Documentation

Documentation about how to build, install and use Kiama can be found on the [Kiama wiki](https://github.com/inkytonik/kiama/blob/master/wiki/Documentation.md).

The main documentation for Kiama takes the form of wiki pages covering library features and examples. The [User Manual](https://github.com/inkytonik/kiama/blob/master/wiki/UserManual.md) is a good place to start.

See the [Research wiki page](https://github.com/inkytonik/kiama/blob/master/wiki/Research.md) for links to papers and presentations about Kiama.

For summary information about Kiama releases, including dependencies on other software and links to API documentation, see the [Releases wiki page](https://github.com/inkytonik/kiama/blob/master/wiki/Releases.md).

See the [Installation wiki page](https://github.com/inkytonik/kiama/blob/master/wiki/Installation.md) for instructions on how to install Kiama.

## Licensing

Kiama is distributed under the Mozilla Public License, v. 2.0. See the file LICENSE for details of the license. More information can be obtained from [Mozilla](http://mozilla.org/MPL/2.0/).

## Some Examples

A traditional approach to language processing is to represent the data to be processed as a hierarchical structure (a tree).  Kiama provides different ways to operate on such trees to carry out typical language processing tasks.

* [Context-sensitive attribute equations](https://github.com/inkytonik/kiama/blob/master/wiki/Attribution.md)

Attribute equations define properties of tree nodes in terms of constants or the properties of other nodes.  In this example, the local and global minima of a binary tree (`locmin` and `globmin`) are defined using simple local equations.  Accessing an attribute (property) of a node is just a function call (also accessible via the arrow notation in Kiama 1.x).  The `attr` function provides caching and circularity checking to the equations.

Kiama version 1.x:

    val locmin : Tree ==> Int =
        attr {
            case Fork(l, r) => (l->locmin) min (r->locmin)
            case Leaf(v)    => v
        }

    val globmin : Tree ==> Int =
        attr {
            case t if t isRoot => t->locmin
            case t             => t.parent[Tree]->globmin
        }

Kiama version 2.x:

    val locmin : RepminTree => Int =
        attr {
            case Fork(l, r) => locmin(l).min(locmin(r))
            case Leaf(v)    => v
        }

    val globmin : RepminTree => Int =
        attr {
            case tree.parent(p) => globmin(p)
            case t              => locmin(t)
        }

* [Dataflow Circular attribute equations](https://github.com/inkytonik/kiama/blob/master/wiki/Dataflow.md)

Sometimes it is useful to define attributes using a mutual dependency.  With `attr` this approach would lead to an error since it would not be possible to calculate the values of such attributes. The `circular` function goes further by implementing mutually dependent attributes via an iteration to a fixed point. In this example, we are calculating variable liveness information for a imperative language statements using the standard data flow equations.

Kiama 1.x:

    val in : Stm ==> Set[Var] =
        circular(Set[Var]()) {
            case s => uses(s) ++ (out(s) -- defines(s))
        }

    val out : Stm ==> Set[Var] =
        circular(Set[Var]()) {
            case s => (s->succ) flatMap (in)
        }

Kiama 2.x:

    val in : Stm => Set[Var] =
        circular(Set[Var]())(
            s => uses(s) ++ (out(s) -- defines(s))
        )

    val out : Stm => Set[Var] =
        circular(Set[Var]())(
            s => succ(s) flatMap (in)
        )

* [Rewrite rules and higher-order rewriting strategies](https://github.com/inkytonik/kiama/blob/master/wiki/Lambda2.md)

While attributes provide a way to decorate a tree with information, rewriting is concerned with transforming trees, perhaps for translation or for optimisation. Kiama contains a strategy-based rewriting library heavily inspired by the [http://strategoxt.org/ Stratego] program transformation language. In this example, we are implementing an evaluation strategy for lambda calculus, using generic strategies such as innermost rewriting.

Kiama 1.x:

    val beta =
        rule {
            case App(Lam(x, t, e1), e2) =>  Let(x, t, e2, e1)
        }

Kiama 2.x:

    val beta =
        rule[Exp] {
            case App(Lam(x, t, e1), e2) => substitute(x, e2, e1)
        }

Kiama 1.x and 2.x:

    val lambda =
        beta + arithop + subsNum + subsVar + subsApp + subsLam + subsOpn

    def innermost (s : => Strategy) : Strategy =
        all(innermost(s) <* attempt(s <* innermost(s)))

    val s : Strategy =
        innermost(lambda)

* [Pretty-printing](https://github.com/inkytonik/kiama/blob/master/wiki/PrettyPrinting.md)

Kiama's pretty-printing library provides a flexible way to describe how you want your output to be produced within constraint of line length. For example, the following describes how to pretty-print the constructs of a simple imperative language, where `group`, `nest` and `line` cooperate to produce nicely indented code that breaks lines  at sensible place when needed.

Kiama 1.x and 2.x:

    def toDoc(t : ImperativeNode) : Doc =
        t match {
            case Num(d)      => value(d)
            case Var(s)      => s
            case Neg(e)      => parens("-" <> toDoc(e))
            case Add(l, r)   => binToDoc(l, "+", r)
            case Sub(l, r)   => binToDoc(l, "-", r)
            case Mul(l, r)   => binToDoc(l, "*", r)
            case Div(l, r)   => binToDoc(l, "/", r)
            case Null()      => semi
            case Seqn(ss)    => group(braces (nest (line <> ssep (ss map toDoc, line)) <> line))
            case Asgn(v, e)  => toDoc(v) <+> "=" <+> toDoc(e) <> semi
            case While(e, b) => "while" <+> parens(toDoc(e)) <> group(nest(line <> toDoc(b)))
        }

    def binToDoc(l : ImperativeNode, op : String, r : Im****perativeNode) : Doc =
        parens(toDoc(l) <+> op <+> toDoc (r))

## Mailing lists

* kiama (General announcements and discussions)

    * `http://groups.google.com/group/kiama`
    * `kiama@googlegroups.com`

## Acknowledgements

The main Kiama Project team is:

* Tony Sloane
* Matthew Roberts
* Dominic Verity

Other contributors have been:

* Len Hamey
* Lennart Kats and Eelco Visser (some aspects of attribution)
* Ben Mockler (the first version of the Oberon-0 example)

Kiama is currently concentrating on incorporating existing language processing
formalisms, so credit goes to the original developers of those formalisms.  See
the code for details of the sources of ideas that come from elsewhere.

Many of the library rewriting strategies are based on the Stratego library.
See http://releases.strategoxt.org/docs/api/libstratego-lib/stable/docs/.

## Supporters

Work on this project has been supported by the following Universities, funding agencies
and companies.

* University of Minnesota

* Delft University of Technology, The Netherlands

* Eindhoven University of Technology, The Netherlands

* Netherlands Organization for Scientific Research

    * Combining Attribute Grammars and Term Rewriting for Programming Abstractions project (040.11.001)
    * MoDSE: Model-Driven Software Evolution project (638.001.610)
    * TFA: Transformations for Abstractions project (612.063.512)

* YourKit

[![YourKit](https://www.yourkit.com/images/yklogo.png)](http://www.yourkit.com)

YourKit is kindly supporting open source projects with its full-featured Java Profiler. YourKit, LLC is the creator of innovative and intelligent tools for profiling Java and .NET applications. Take a look at YourKit's leading software products: [YourKit Java Profiler](http://www.yourkit.com/java/profiler/index.jsp) and [YourKit .NET Profiler](http://www.yourkit.com/dotnet/index.jsp).

* CloudBees

[![CloudBees](http://www.cloudbees.com/sites/default/files/Button-Built-on-CB-1.png)](http://cloudbees.com)

CloudBees provides [generous support to FOSS projects](http://www.cloudbees.com/foss/index.cb) for continuous builds and other services, for which we are very grateful. [nightly builds](https://inkytonik.ci.cloudbees.com/job/Kiama) are built on a CloudBees Jenkins instance, part of their DEV@cloud service.
