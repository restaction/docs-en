# YAML and Schema syntax

JSON is a subset of YAML, so most JSON data can be parsed by YAML parser.

Since YAML use indent to control data struct, and needn't quote strings,
it's more suitable for writing schema in doc string.

[Validr](https://github.com/guyskk/validr) use JSON to represent schema, but
this web framework use YAML for fitting different scene.

To learn schema syntax, you should learn basic JSON and YAML syntax first.
See http://json.org/ ,http://yaml.org/ and http://pyyaml.org/.

Then you can learn [schema syntax](https://github.com/guyskk/validr/blob/master/Isomorph-JSON-Schema.md)
in Validr, it is easy to be master of.

Finally, migrate from JSON to YAML, beginers may encounter some problems, 
I summarized some problems for easy to start.


## YAML parser

There are some special chars in YAML, eg: `@`,`&`,`*`. `&` is used for [Anchors](http://pyyaml.org/wiki/PyYAMLDocumentation#Aliases) but it almost won't
used in schema. Those chars are conflict with schema syntax, so I modified
YAML parser, treat `@`, `&` as plain text and disable Anchor syntax of YAML.

## list

Note: `'-'` should followed with a space or new line.

simple list:

    # tags
    - &unique&minlen=1
    - str

nested list:

    # time_table
    # Monday
    - - morning write BUG
      - afternoon fix BUG
      - evening write BUG
    # Tuesday
    - - morning fix BUG
      - afternoon write BUG
      - evening fix BUG
    # ...

    # schema_of_time_table
    - &minlen=7&maxlen=7 # 7 days a week
    - - &minlen=3&maxlen=3 # 3 period per day
      - str&optional # todos


## dict

Note: `':'` should followed with a space or new line.

simple dict:

    user:
        id?int: user id
        name?str: user name

nested dict:

    friends:
        best:
            $self: best friend
            id?int: user id
            name?str: user name
        bad:
            $self: bad friend
            id?int: user id
            name?str: user name

    friends:
        best@user: best friend
        bad@user: bad friend


## complicated nested

dict in list:

    - my friends
    - id?int: user id
      name?str: user name

    - my friends
    - @user

list in dict:

    friends:
        - my friends
        - id?int: user id
          name?str: user name

    friends:
        - my friends
        - @user


## refer

    $shared:
        userid: int
        tags:
            - &unique&minlen=1
            - str
        user:
            id?int: user id
            name?str: user name

refer to the back of the front:

    $shared:
        userid: int
        user:
            id@userid: user id
            name?str: user name

## mixin

    $shared:
        paging:
            page_num?int&min=1&default=1: page number
            page_size?int&min=1&default=10: page size
        query:
            $self@paging: query params
            tag?str: tag
            date?date: date
