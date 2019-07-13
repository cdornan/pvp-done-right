# PVP Done Right

It is sometimes easy to forget how successful The Haskell Package Versioning
Policy ([PVP](https://pvp.haskell.org/) for short) has been.

The problem comes in the "Dependencies in Cabal" section where it says

```
When publishing a Cabal package, you SHALL ensure that your dependencies in the
build-depends field are accurate. This means specifying not only lower bounds,
but also upper bounds on every dependency.

At some point in the future, Hackage may refuse to accept packages that do not
follow this convention. The aim is that before this happens, we will put in
place tool support that makes it easier to follow the convention and less
painful when dependencies are updated.
```

The community is clearly divided on whether is is desirable to require users
to specify conservative upper bounds &mdash; specifying that a future version
of a dependent package will be incompatible on the basis that it is free
to not be upwards compatible according to the PVP.

The author of this proposal believes that requiring users to specify
conservative upper bounds was a mistake but that we should, as suggested by the
PVP itself, try to strengthen the tools to address concerns that folks have with
the proposal and get broad approval across the community.


## In Brief

We propose:

  * adding sufficient annotations to the cabal file format so that tools can
    generate strict-PVP-compliant cabal files without discarding any
    information;

  * this will allow front-end tools to generate PVP-compliant cabal files
    that have the original cabal files embedded in them for use by
    infrastructure that works best with packages that strictly comply with
    the PVP;

  * it will also allow Hackage to mechanically carry out this step without
    discarding information (though it would be best to allow folks the option
    of preventing Hackage from doing this if they feel strongly enough);

  * all of this should be done by extending the cabal file syntax in a way
    that is backwards compatible with existing cbal file processors.


## How do we do this

In brief, editors may add bounds constraints to make a cabal file PVP compatible
provided:

  * the original constraint is pushed into a new `x-build-depends-history`
    clause that will be ignored by existing tools but which can be used to
    recover history by new tools;

  * the new constraint is annotated with

      + an epistemology declaration, saying whether the constraint is
        `projected` or `known`;

      + a statement of who has edited this constraint (the author, a maintainer,
        a trustee, the Hackage service, etc.);

      + a UTC time stamp.


## An example

We take as an example the array dependency of the `text-1.2.3.1` package,
where we replace
```
    array      >= 0.3
```
with
```
    array      >= 0.3 && < 0.6,
```
as follows:
```
  build-depends:
      -- #EDITED projected trustee:bob 2019-07-12T12:42:30Z
    array      >= 0.3 && < 0.6,
    base       >= 4.2 && < 4.13,
    binary,
    deepseq    >= 1.1.0.0,
    ghc-prim   >= 0.2
  x-build-depends-history:
    array      >= 0.3
```
The `#EDITED` comment could appear before or after the edited dependency
to accommodate styles that place commas at the start or the end of the line.

Notes:

  * The details of the syntax are not important; this is merely illustrative;

  * More details on `trustee:bob` (a proper name and email address) could be
    provided by way of a `x-trustee:` clause at the top level of the cabal file.


## What about conditional dependencies?

The same procedure applies for dependencies specified conditionals: the edited
conditional gets tagged in the same way and the original conditional gets logged
in an adjacent `x-build-depends-history` (see the example edit in the
accompanying `text.example-cabal` for details).
