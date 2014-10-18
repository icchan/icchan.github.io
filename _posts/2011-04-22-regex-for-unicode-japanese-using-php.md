---
title: Regex for Unicode Japanese using PHP
layout: posts
---

Recently I've had to deal with Japanese character validation a fair bit, and its a fair bit tricker than dealing with the usual checking that an input only contains letters, numbers and underscores.

A pretty standard white list validation would be something like:

```php
preg_match("/^[a-zA-Z0-9]+$/",$input) 
```

Which says all the characters from the start to the end are either letters or numbers. Of course this doesn't work in Japanese.

First thing you need to do is use the /u option at the end to enable unicode matching. Next you need to put the Japanese characters inside the square brackets [].

To do this we use something like this \x{####} where #### is the unicode for the character. You can look up the codes on the internet somewhere like <a href="http://www.rikai.com/library/kanjitables/kanji_codes.unicode.shtml">this</a>.

So if you want hiragana you would use something like: \x{3041}-\x{3096}
for katakana you would use: \x{30a1}-\x{30fc}
For common kanji use: \x{4e00}-\x{9faf}
For uncommon kanji use: \x{3400}-\x{4dbf}

So finally, if we want to white list letters, numbers, hiragana,katakana and common kanji we could do something like:

```php
preg_match("/^[a-zA-Z0-9\x{3041}-\x{3096}\x{30a1}-\x{30fc}\x{4e00}-\x{9faf}]+$/u",$input)
```

You can also add underscores or dashes or whatever suits your needs.
