---
title: Using Whitelisting to control file access in Apache Webserver
date: 2016-06-28
description: You want to control which files on your webserver can be accessed by whom, but you don’t want to use blacklisting. You want to say, i.e., external users can access files whose names end with .php, .jpg and .png. All other files must not be accessible, no matter which name they have.
draft: false
tags:
  - HowTo
  - Apache Webserver
  - Whitelisting
toc: false
authors: [Fuseki]
---


# Using Whitelisting to control file access in Apache Webserver

## The aim

You want to control which files on your webserver can be accessed by
whom, but you don’t want to use blacklisting. You want to say, i.e.,
external users can access files whose names end with .php, .jpg and
.png. All other files must not be accessible, no matter which name they
have.

You can use `LocationMatch` and `FilesMatch` to control access to files.
In this example, we will use FilesMatch, because we care about files
which are stored on the filesystem. But the same approach also applies
to files, which are generated when requested.

## The problem

When apache finds a FilesMatch entry in its configuration which matches
the requested filename, the corresponding rules are applied. There is no
problem with that as long as you do not try to create a catch-all rule,
such as

`<FilesMatch "^.*$">`  
`Require all denied`  
`</FilesMatch>`

`<FilesMatch "^.+\.(php|jpg|png)$">`  
`Require all allowed`  
`</FilesMatch>`

Clearly, the first rule applies to all files. It also does not make a
difference if you change the order of both entries. So, the above
configuration always returns 403.

## The solution

Unfortunately, regular expressions do not support a special syntax for
negative matches. You cannot say “everything that doesn’t matches XYZ”.
But you can do something similar: Negative lookahead. Apache uses *Perl
Compatible Regular Expressions (PCRE)* for `FilesMatch` and
`LocationMatch`.

Consider the String `index.php` and the regex `index.(?:php)`.
Obviously, `index.` matches the first part of the filename. Then,
something special happens. Apache tries to match the substring “php” to
the regex `(?:php)` and succeeds. Lookahead means that no characters of
the string are consumed and thus do not belong to the matched string. In
the example, the matched string would be `index.`; as long as it is
followed by `php`.

We now use *negative lookahead* to *not deny* access to all file
extensions we want to be accessible:

`^.*[^.](?!\.(php|jpg|png))`

This regular expression applies to all files whose name does not end
with php, jpg, png. But be aware, this also matches to `index.php`
itself, because the first part of the expression, `^.*[^.]`, matches to
`index.php` and the negative lookahead part matches to every zero-length
string. To prevent this, we enforce the negative lookahead to consume
those three characters that did not match the expression, by appending
`.{3,3}` to it. The resulting setting is:

`<FilesMatch "^.*[^.](?!\.(php|jpg|png)).{3,3}$">`  
`Require all denied`  
`</FilesMatch>`
