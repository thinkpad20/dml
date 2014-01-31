## DML

DML is a language for generating text files from static data. It is somewhat similar to moustache; however, it is able to operate on (though not modify) the data and make decisions about what it should write based on what is presented.

### Example

Here's a sample template file.

```
#begin Aliases {sex = protagonist.sex}
${nom = if sex == 'male' then 'he' else 'she';
  acc = if sex == 'male' then 'him' else 'her';
  pos = if sex == 'male' then 'his' else 'her';
  RenderBd = [year, month, day] -> '$month $day, $year'
           | [year, month] -> 'some day in $month of $year'
           | [year] -> 'some day in $year'}
#end Aliases {export [nom, acc, pos]}

#begin 'Main story', {@context protagonist}
This is a story about $name. ${nickname or name} was born on ${RenderBd(birthday)}, which makes $acc ${if age > 50 then 'somewhat old' elif age < 20 'quite young' else 'an unremarkable age of $age'}. In any case, only just last year $nom was ${age - 1} years old, but $nom was already starting to ${if age < 50 then 'fear' else 'not care about'} $pos birthday.
#end 'Main story'
```

And we have this data dictionary:

```
{
  "protagonist": {
    "name": "Allen"
    "sex": "male",
    "age": "28",
    "birthday": ["1985", "April", "20"]
  }
}
```

Then we'd render to this output:

```

This is a story about Allen. Allen was born on April 20, 1985, which makes him an unremarkable age of 28. In any case, only just last year he was 27, but he was already starting to fear his birthday.
```

And with this input:

```
{
  "protagonist": {
    "name": "Gabrielle",
    "nickname": "Gabby",
    "sex": "female",
    "age": "56",
    "birthday": ["1958"]
  }
}
```

Then we'd get this output:

```

This is a story about Gabrielle. Gabby was born on some day in 1958, which makes her somewhat old. In any case, only just last year she was 55, but she was already starting to not care about her birthday.
```

### Syntax

At a high level, the DML syntax is as follows. At the outermost level is the string. This can contain any sequence of characters, but the following characters have special meaning:

* Hash tag directives (`#`), all of which can be escaped by putting a `\` in front of the hash:
  * `#` starts a single-line comment IF followed by a space.
  * `#note` starts a block comment ending in `#endnote`.
  * `#begin <name> <initializer>` and `#end <name> <exports>` introduce and end lexical scopes for aliases.
    * The name is optional but if present will be checked for matched pairs.
    * The "initializer" is an arbitrary DML expression (see below), which won't be printed (so it's useful for variable assignments).
    * Any assignments made in between two of these will not reprotagonist unless included in the export list.
    * Assignments to make right at the beginning can be put in a pair of curly braces after the `#begin` (and after any name given).
* A `$` initializes a DML expression. This can be either a single variable, in which case the dollar sign alone is sufficient (`$foo`), or a more complicated expression, in which case we need curly braces: `${Max(foo, bar.baz * 3)}`. If only a single variable is needed, no curly braces are required: `$foo.bar` is equivalent to `${foo.bar}`. If a literal dollar sign is needed, write it with `\$`.
* DML expressions can be:
  * constants; for example: variables (which are constant because they are read from a data file), numbers, strings, lists, dictionaries, etc.
  * `if ... then ... else` statements
  * case statements
  * function calls
  * definitions, which can be functions if they contain a `->`.
* Strings in DML can themselves contain DML: for example, `I think ${if foo then 'the answer is ${foo + 3}' else 'nobody knows the answer'}.`
* They can also contain a `@context <identifier>` directive, which means that any variable read should first be looked for as an attribute of `<identifier>`.
  * For example, if the data file contains `{"foo": {"bar": "Tom", "baz": "Dick"}, "qux": "Harry"}` and we have `@context foo`, then `$foo.bar introduced $baz to $qux` would render as "Tom introduced Dick to Harry".
  * If it were desired that *only* `foo` be looked in, and otherwise not found, then `@strictContext` can be used. In that case, we would throw an error when we tried to reference `foo.bar` and `qux` (or we would render blank text in 'forgiving' mode).
* DML expressions can deal with collections of objects as well. They can iterate over collections as well: if the data contains `"mylist": [1, 2, 3]` then we can write

    ```
    My list contains ${mylist.Filter(n -> n % 2 == 1).Length} odd numbers: \_
    ${mylist.Filter(n -> n % 2 == 0).Map(RenderNumber).Enumerate}
    ```
  which will render as "My list contains 2 odd numbers: one and three". The `\_` is a whitespace skipper which will remove all whitespace (in this case, the line break) until the first non-whitespace character.
* Function calls can be written prefix with `Foo(bar, baz)` or postfix with `bar.Foo(baz)`
* Typing is dynamic and somewhat loosey goosey. Don't go too crazy with it.
