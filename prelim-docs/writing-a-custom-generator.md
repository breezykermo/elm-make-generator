## Writing A Custom Generator

The Elm Compiler codebase is well-structured to easily add different generators. The code relevant to generating JavaScript is almost exclusively in [src/Generate](https://github.com/elm-lang/elm-compiler/tree/master/src/Generate):

Generate
└───JavaScript
│   │   Builder.hs
│   │   BuiltIn.hs
│   │   Expression.hs
│   │   Foreign.hs
│   │   Helper.hs
│   │   Literal.hs
│   │   Variable.hs
|   JavaScript.hs
```

The entrypoint for the generator is the `generate` function in [JavaScript.hs](https://github.com/elm-lang/elm-compiler/blob/master/src/Generate/JavaScript.hs#L23). This function generates the JS equivalents of the Elm library and custom code from the `generateDef` function, which is imported from [Expression.hs](https://github.com/elm-lang/elm-compiler/blob/master/src/Generate/JavaScript/Expression.hs#L89). This *def* (presumably 'definition', meaning the set of definitions) is then combined with the *effectManager*.

TODO: what exactly does the effect manager do, and how are they constructed in the compiler? See [here for some basics](https://newfivefour.com/elm-lang-effects-managers-basics.html), and [here for an example](https://github.com/fredcy/localstorage).