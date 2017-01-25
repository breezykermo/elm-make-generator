# Creating A New Generator

The [Elm compiler](https://github.com/elm-lang/elm-compiler) is designed to generate a JS file through the interface of [Elm Make](https://github.com/elm-lang/elm-make), a command line tool that reads an Elm file as input and generates either a JS file. Elm Make also provides the option to produce HTML output, which is very similar to the JS output, but includes some HTML boilerplate to set up the right hooks for a fully functional application.

## Elm Make

When Elm compiles a source file to JS, its entrypoint is [Elm Make](https://github.com/elm-lang/elm-make). Elm Make interprets the options passed at the command line, invokes the appropriate functions in Elm Compiler (see below) to generate JS, and then packages that code with the appropriate boilerplate to create either a JS file or an HTML file.

Add your new generator as a custom data type in [BuildManager.hs](https://github.com/elm-lang/elm-make/blob/master/src/BuildManager.hs#L38-L49):

```haskell
data Output
    = Html FilePath
    | JS FilePath
    -- ADD THE FOLLOWING LINE
    -- ----------------------
    | OCaml FilePath
    -- ----------------------
    | DevNull

outputFilePath :: Config -> FilePath
outputFilePath config =
  case _output config of
    Html file -> file
    JS file -> file
    -- ADD THE FOLLOWING LINE
    -- ----------------------
    OCaml file -> file
    -- ----------------------
    DevNull -> "/dev/null"
```

There are many ways to make your custom generator accessible from the command line interface, but the easiest way is to simply interpret which generator to use from the extension of the specified output file, which is the way the compiler distinguishes between the HTML and JS generation processes. Add the extension through a conditional in [src/Flags.hs](https://github.com/elm-lang/elm-make/blob/master/src/Flags.hs#L160-L164):

```haskell
if ext == ".html" then
  Just (BM.Html path)

else if ext == ".js" then
  Just (BM.JS path)

-- ADD THE FOLLOWING LINES
-- -----------------------
else if ext == ".ocaml" then
  Just (BM.OCaml path)
-- -----------------------
```

Finally, add the case in [src/Pipeline/Generate.hs](https://github.com/elm-lang/elm-make/blob/master/src/Pipeline/Generate.hs#L78-L92):

```haskell
case BM._output config of
  BM.Html outputFile ->
    liftIO $
      do  js <- mapM File.readTextUtf8 objectFiles
          let (TMP.CanonicalModule _ moduleName) = head rootModules
          let outputText = html (Text.concat (header : js ++ [footer])) moduleName
          LazyText.writeFile outputFile outputText

  BM.JS outputFile ->
    liftIO $
    File.withFileUtf8 outputFile WriteMode $ \handle ->
        do  Text.hPutStrLn handle header
            forM_ objectFiles $ \jsFile ->
                Text.hPutStrLn handle =<< File.readTextUtf8 jsFile
            Text.hPutStrLn handle footer

-- ADD THE FOLLOWING LINES
-- -----------------------
  BM.OCaml.outputFile ->
    liftIO $
    -- perform your generation (TODO: work out exactly what's happening here.)
-- -----------------------
```

This is where you can do any adding of headers and footers that you might need, if you are generating code that requires it.

## Elm Compiler

The generate process is invoked in [Elm/Compiler.hs](https://github.com/elm-lang/elm-compiler/blob/master/src/Elm/Compiler.hs#L91), in the `compile` function. To use a different generator, simply reconfigure the intermediate result to use custom generator code, rather than the built-in JS generator. For example, to use an OCaml generator you might have written, you would change the import statement at the top of the file, and the invocation code in the `generate` function:

```haskell
-- import qualified Generate.JavaScript as JS
import qualified Generate.OCaml as OCaml

- ...
- ...

-- let javascript = JS.generate modul
let ocaml = OCaml.generate modul

-- return (Result docs interface javascript)
return (Result docs interface ocaml)
```

The Elm Compiler codebase is now set up to compile through your custom generator, rather than the built-in JS generator.

