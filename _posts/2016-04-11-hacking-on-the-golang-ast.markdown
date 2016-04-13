---
author: ArdaXi
comments: true
date: 2016-04-11 09:00:00
layout: post
slug: hacking-on-the-golang-ast
title: Hacking on the Golang AST
---

The Go programming language is very keen to invite you to write code that writes code. To this end, the [go](https://golang.org/pkg/go/) packages are provided.
This includes a parser, tokenizer, printer, and a bunch of other fun stuff. However, using them can be a bit counter-intuitive. Provided here is some things you can do with an AST.

First, we'll need to know what we're working with. The easiest way is to simply take an existing file and looking at the AST it generates. In this example, I'm going to take a 
package with one simple struct, and look at how we can read the types inside, and see about adding one.

<!-- more -->

This is our example code:

    package main

    type Foo struct {
      Bar string
    }

You'll note that we are including a package directive. Without it, the parser will refuse to parse the file. To dump the ast, I've taken the example code from [go/ast].

    package main

    import (
      "go/ast"
      "go/parser"
      "go/token"
      "os"
    )

    func main() {
      fset := token.NewFileSet()
      f, err := parser.ParseFile(fset, os.Args[1], nil, 0)
      if err != nil {
        panic(err)
      }

      ast.Print(fset, f)
    }

Compiling and installing this as `ast` gives us a command we can run on any Go file. Running it on our earlier snippet:

     0  *ast.File {
     1  .  Package: testast.go:1:1
     2  .  Name: *ast.Ident {
     3  .  .  NamePos: testast.go:1:9
     4  .  .  Name: "main"
     5  .  }
     6  .  Decls: []ast.Decl (len = 1) {
     7  .  .  0: *ast.GenDecl {
     8  .  .  .  TokPos: testast.go:3:1
     9  .  .  .  Tok: type
    10  .  .  .  Lparen: -
    11  .  .  .  Specs: []ast.Spec (len = 1) {
    12  .  .  .  .  0: *ast.TypeSpec {
    13  .  .  .  .  .  Name: *ast.Ident {
    14  .  .  .  .  .  .  NamePos: testast.go:3:6
    15  .  .  .  .  .  .  Name: "Foo"
    16  .  .  .  .  .  .  Obj: *ast.Object {
    17  .  .  .  .  .  .  .  Kind: type
    18  .  .  .  .  .  .  .  Name: "Foo"
    19  .  .  .  .  .  .  .  Decl: *(obj @ 12)
    20  .  .  .  .  .  .  }
    21  .  .  .  .  .  }
    22  .  .  .  .  .  Type: *ast.StructType {
    23  .  .  .  .  .  .  Struct: testast.go:3:10
    24  .  .  .  .  .  .  Fields: *ast.FieldList {
    25  .  .  .  .  .  .  .  Opening: testast.go:3:17
    26  .  .  .  .  .  .  .  List: []*ast.Field (len = 1) {
    27  .  .  .  .  .  .  .  .  0: *ast.Field {
    28  .  .  .  .  .  .  .  .  .  Names: []*ast.Ident (len = 1) {
    29  .  .  .  .  .  .  .  .  .  .  0: *ast.Ident {
    30  .  .  .  .  .  .  .  .  .  .  .  NamePos: testast.go:4:2
    31  .  .  .  .  .  .  .  .  .  .  .  Name: "Bar"
    32  .  .  .  .  .  .  .  .  .  .  .  Obj: *ast.Object {
    33  .  .  .  .  .  .  .  .  .  .  .  .  Kind: var
    34  .  .  .  .  .  .  .  .  .  .  .  .  Name: "Bar"
    35  .  .  .  .  .  .  .  .  .  .  .  .  Decl: *(obj @ 27)
    36  .  .  .  .  .  .  .  .  .  .  .  }
    37  .  .  .  .  .  .  .  .  .  .  }
    38  .  .  .  .  .  .  .  .  .  }
    39  .  .  .  .  .  .  .  .  .  Type: *ast.Ident {
    40  .  .  .  .  .  .  .  .  .  .  NamePos: testast.go:4:6
    41  .  .  .  .  .  .  .  .  .  .  Name: "string"
    42  .  .  .  .  .  .  .  .  .  }
    43  .  .  .  .  .  .  .  .  }
    44  .  .  .  .  .  .  .  }
    45  .  .  .  .  .  .  .  Closing: testast.go:5:1
    46  .  .  .  .  .  .  }
    47  .  .  .  .  .  .  Incomplete: false
    48  .  .  .  .  .  }
    49  .  .  .  .  }
    50  .  .  .  }
    51  .  .  .  Rparen: -
    52  .  .  }
    53  .  }
    54  .  Scope: *ast.Scope {
    55  .  .  Objects: map[string]*ast.Object (len = 1) {
    56  .  .  .  "Foo": *(obj @ 16)
    57  .  .  }
    58  .  }
    59  .  Unresolved: []*ast.Ident (len = 1) {
    60  .  .  0: *(obj @ 39)
    61  .  }
    62  }

You can see that at the top level, an `*ast.File` has a few interesting fields. It has the position where the package starts, the package name, declarations, scope,
and unresolved objects. The struct we want to modify has the information in an `ast.Spec` field. Let's try printing our struct first.
We'll change the example code above to print some information about all the struct types it finds.

    ackage main

    import (
      "fmt"
      "go/ast"
      "go/parser"
      "go/token"
      "os"
    )

    func main() {
      fset := token.NewFileSet()
      f, err := parser.ParseFile(fset, os.Args[1], nil, 0)
      if err != nil {
        panic(err)
      }

      ast.Inspect(f, func(n ast.Node) bool {
        switch t := n.(type) {
        case *ast.TypeSpec:
          st, ok := t.Type.(*ast.StructType)
          if !ok {
            return true
          }

          fmt.Printf("Type name: %s\n", t.Name.Name)
          for _, field := range st.Fields.List {
            fmt.Printf("Field name: %s with type: %s\n", field.Names[0].Name, field.Type.(*ast.Ident).Name)
          }
        }
        return true
      })
    }

We've replaced our `ast.Print` call with `ast.Inspect`. This is a function that takes an `ast.Node` and a `func (ast.Node) bool`.
It will walk through the entire AST starting with `f` in depth-first order, calling the function on each node it finds. If the function returns `true`, it continues,
if the function returns `false`, it aborts. In this case, the function looks for nodes of type `*ast.TypeSpec`. If the node isn't of this type, it continues.
This way, we only find type declarations and nothing else.

If it finds a type declaration, it then checks if the `Type` it declares is an `*ast.StructType`. If not, it continues to the next node without doing anything.
If it is a struct type, it prints the name and iterates through the fields it declares. For each field, it prints the name and the type.
For the test file, this is the output:

    Type name: Foo
    Field name: Bar with type: string

If you add more structs or fields to the file, those will be picked up automatically. We can use this same principle to alter the AST. In this example, we'll start
by changing each string field to an int type.

    package main

    import (
      "go/ast"
      "go/parser"
      "go/printer"
      "go/token"
      "os"
    )

    func main() {
      fset := token.NewFileSet()
      f, err := parser.ParseFile(fset, os.Args[1], nil, 0)
      if err != nil {
        panic(err)
      }

      ast.Inspect(f, func(n ast.Node) bool {
        switch t := n.(type) {
        case *ast.TypeSpec:
          st, ok := t.Type.(*ast.StructType)
          if !ok {
            return true
          }

          for _, field := range st.Fields.List {
            if ident, ok := field.Type.(*ast.Ident); ok && ident.Name == "string" {
              ident.Name = "int"
            }
          }
        }
        return true
      })

      printer.Fprint(os.Stdout, fset, f)
    }

This code is quite similar to the previous code, with a few changes. We're no longer printing anything from the `Inspect` function,
because we want to print sourcecode instead. Therefore, at the end of the script we're calling `printer.Fprint` to print the entire AST to stdout.
Inside the Inspect function, we look for fields whose Type identifier is a string, and replace the identifier with `"int"`. The output now:

    package main

    type Foo struct {
        Bar int
    }

So far, this does exactly what we want. Now what if we want to add a field instead of changing an existing one? The code for this should be pretty apparent
from the above. We change the `Inspect` call.

    ast.Inspect(f, func(n ast.Node) bool {
      switch t := n.(type) {
      case *ast.TypeSpec:
        st, ok := t.Type.(*ast.StructType)
        if !ok {
          return true
        }

        newField := &ast.Field{
          Names: []*ast.Ident{
            {Name: "Baz"},
          },
          Type: &ast.Ident{
            Name: "string",
          },
        }

        st.Fields.List = append(st.Fields.List, newField)
      }
      return true
    })

We create a new field with a name of `Baz` and type `string`. We append that field to the fields list on the `StructType`. We then print the AST as before, which results in:

    package main

    type Foo struct {
      Bar     string
      Baz     string
    }

You will notice it doesn't exactly follow the gofmt style, instead placing spaces wherever it likes. To fix this, we use the `format` package instead of the `printer` package.
The last line in the main function becomes: `format.Node(os.Stdout, fset, f)`. This prints properly.

    package main

    type Foo struct {
      Bar string
      Baz string
    }
