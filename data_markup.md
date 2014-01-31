Let's say this our template.

```
#- Aliases, can be either constant or take in arguments
@{nom = if $main.sex == 'male' then 'he' else 'she'}
@{acc = if $main.sex == 'male' then 'him' else 'her'}
@{pos = if $main.sex == 'male' then 'his' else 'her'}
@{Bd = [year, month, day] -> '$month $day, $year'
     | [year, month] -> 'some day in $month of $year'
     | [year] -> 'some day in $year'}

This is a story about $main.name. ${main.nickname or main.name} was born on ${Bd(main.birthday)}, which makes $acc ${if main.age > 50 then 'somewhat old' elif main.age < 20 'quite young' else 'an unremarkable age of $main.age'}. In any case, only just last year $nom was ${main.age - 1} years old, but $nom was already starting to ${if $main.age < 50 then 'fear' else 'not care about'} $pos birthday.
```

And we have this data dictionary:

```
{
  "main": {
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
  "main": {
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
