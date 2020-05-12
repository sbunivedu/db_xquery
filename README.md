# XQuery
XQuery is a language for finding and extracting elements and attributes from XML documents. XQuery is to XML what SQL is to relational databases.

This case study demonstrates the commonly used features of XQuery. You can test the solutions by downloading and using [BaseX](http://basex.org/download/).

# Books
List all book elements.
```xml
let $books := doc("books.xml")
for $book in $books/bookstore/book
return $book

expected output:
<book category="cooking">
  <title lang="en">Everyday Italian</title>
  <author>Giada De Laurentiis</author>
  <year>2005</year>
  <price discount="promo">20.00</price>
  <format>hardcover</format>
  <price>30.00</price>
</book>
<book category="children">
  <title lang="en">Harry Potter</title>
  <author>J K. Rowling</author>
  <year>2005</year>
  <price>29.99</price>
</book>
...
```

List all book elements in a `Books` element.
```xml
let $books := (
  for $book in doc("books.xml")/bookstore/book
  return $book
)
return <Books>{$books}</Books>

let $books := doc("books.xml")/bookstore/book
return <Books>{$books}</Books>

expected output:
<Books>
  <book category="cooking">
    <title lang="en">Everyday Italian</title>
    <author>Giada De Laurentiis</author>
    <year>2005</year>
    <price discount="promo">20.00</price>
    <format>hardcover</format>
    <price>30.00</price>
  </book>
  <book category="children">
    <title lang="en">Harry Potter</title>
    <author>J K. Rowling</author>
    <year>2005</year>
    <price>29.99</price>
  </book>
...
</Books>
```
This examples shows that `$books` is a sequence of `book` elements.

Find the titles of books that cost more than 30.
```
let $books := doc("books.xml")
for $book in $books/bookstore/book
where $book/price>30
return data($book/title)

data(doc("books.xml")/bookstore/book[price>30]/title)

expected output:
XQuery Kick Start
Lernen XML
```
Run it at http://www.xpathtester.com/xquery/64dd16907c13f3dd3f3b7796f23385fb

Find the titles of books that cost more than 30 and sort titles.
```xml
let $books := doc("books.xml")
for $book in $books/bookstore/book
where $book/price>30
order by $book/title
return data($book/title)

expected output:
Lernen XML
XQuery Kick Start
```

## Movies
List all the star elements found among the versions of all movies.
```xml
let $movies := doc("movies.xml")
for $m in $movies/Movies/Movie
return $m/Version/Star

expected output:
<Star>Fay Wray</Star>
<Star>Jeff Bridges</Star>
<Star>Jessica Lange</Star>
<Star>Kevin Bacon</Star>
<Star>John Lithgow</Star>
<Star>Sarah Jessica Parker</Star>
```

Produce a sequence of Movie elements, each containing all the stars
of movies with a given title, regardless of which version they starred
in. The title will be an attribute of the Movie element. Note the `""` and `{}` around `$m/@title`.
```xml
let $movies := doc("movies.xml")
for $m in $movies/Movies/Movie
return <Movie title="{$m/@title}">{$m/Version/Star}</Movie>

expected output:
<Movie title="King Kong">
  <Star>Fay Wray</Star>
  <Star>Jeff Bridges</Star>
  <Star>Jessica Lange</Star>
</Movie>
<Movie title="Footloose">
  <Star>Kevin Bacon</Star>
  <Star>John Lithgow</Star>
  <Star>Sarah Jessica Parker</Star>
</Movie>
```

List all the star elements found among the versions of all movies. Put the entire sequence of stars in a `Stars` element:
```xml
let $starSeq := (
  let $movies := doc("movies.xml")
  for $m in $movies/Movies/Movie
  return $m/Version/Star
)
return <Stars>{$starSeq}</Stars>

expected output:
<Stars>
  <Star>Fay Wray</Star>
  <Star>Jeff Bridges</Star>
  <Star>Jessica Lange</Star>
  <Star>Kevin Bacon</Star>
  <Star>John Lithgow</Star>
  <Star>Sarah Jessica Parker</Star>
</Stars>
```

Find the cities in which stars (in `stars.xml`) mentioned in the `movies.xml` file live.
```xml
let $movies := doc("movies.xml"),
    $stars := doc("stars.xml")
for $s1 in $movies/Movies/Movie/Version/Star,
    $s2 in $stars/Stars/Star
where data($s1) = data($s2/Name)
return $s2/Address/City

expected output:
<City>Hollywood</City>
<City>Malibu</City>
```

Find movies that have more than one version.
```xml
let $movies := doc("movies.xml")
for $m in $movies/Movies/Movie
where count($m/Version) > 1
return $m

expected output:
<Movie title="King Kong">
  <Version year="1933">
    <Star>Fay Wray</Star>
  </Version>
  <Version year="1976">
    <Star>Jeff Bridges</Star>
    <Star>Jessica Lange</Star>
  </Version>
  <Version year="2005"/>
</Movie>
```

Produce each of the versions of "King Kong", tagging the most
recent version `Latest` and the earlier versions `Old`.
```xml
let $kk :=
  doc("movies.xml")/Movies/Movie[@title="King Kong"]
for $v in $kk/Version
return
  if ($v/@year = max($kk/Version/@year))
  then <Latest>{$v}</Latest>
  else <Old>{$v}</Old>

expected output:
<Old>
  <Version year="1933">
    <Star>Fay Wray</Star>
  </Version>
</Old>
<Old>
  <Version year="1976">
    <Star>Jeff Bridges</Star>
    <Star>Jessica Lange</Star>
  </Version>
</Old>
<Latest>
  <Version year="2005"/>
</Latest>
```

Consider all versions of all movies, order them by year, and produce
a sequence of `Movie` elements with the title and year as attributes.
```xml
let $movies := doc("movies.xml")
for $m in $movies/Movies/Movie,
  $v in $m/Version
order $v/@year
return <Movie title="{$m/@title}" year="{$v/@year}" />

expected output:
<Movie title="King Kong" year="1933"/>
<Movie title="King Kong" year="1976"/>
<Movie title="Footloose" year="1984"/>
<Movie title="King Kong" year="2005"/>
```

## Ships
Find the names of classes that had at least 10 guns.
```xml
let $ships := doc("ships.xml")
return data($ships/Ships/Class[@numGuns>=10]/@name)

let $ships := doc("ships.xml")
for $class in $ships/Ships/Class
where $class/@numGuns >= 10
return data($class/@name)

expected output:
Tennessee
King George V
```

Find the names of ships that had a least 10 guns.
```xml
let $ships := doc("ships.xml")
return data($ships/Ships/Class[@numGuns>=10]/Ship/@name)

let $ships := doc("ships.xml")
for $class in $ships/Ships/Class
where $class/@numGuns >= 10
return data($class/Ship/@name)

expected output:
Tennessee
California
King George V
Prince of Wales
Duke of York
Howe
Anson
```

Find the names of the ships that were sunk.
```xml
let $ships := doc("ships.xml")
for $ship in $ships/Ships/Class/Ship
where $ship/Battle/@outcome = "sunk"
return data($ship/@name)

let $ships := doc("ships.xml")
for $ship in $ships//Ship[Battle]
where $ship/Battle/@outcome = "sunk"
return data($ship/@name)

expected output:
Kirishima
Prince of Wales
```

Find the names of classes with at least 3 ships.
```xml
let $ships := doc("ships.xml")
for $class in $ships/Ships/Class
where count($class/Ship) >= 3
return data($class/@name)

expected output:
Kongo
King George V
```

Find the names of the classes such that no ship of that class was in a battle.
```xml
let $ships := doc("ships.xml")
for $class in $ships/Ships/Class
where count($class/Ship/Battle) = 0
return $class/@name

let $ships := doc("Ships.xml")
for $class in $ships//Class
where every $ship in $class/Ship satisfies count($ship/Battle) = 0
return data($class/@name)

expected output:
0 results
```

Find the names of the classes that had at least two ships launched in the same year.
```xml
let $ships := doc("ships.xml")
for $class in $ships/Ships/Class
where (
  for $s1 in $class/Ship,
      $s2 in $class/Ship
  where $s1/@launched = $s2/@launched and
    $s1/@name != $s2/@name
  return $class/@name)
return data($class/@name)

let $ships := doc("Ships.xml")
for $class in $ships//Class
where some $ship in $class/Ship satisfies count($class/Ship[@launched = $ship/@launched]) >= 2
return data($class/@name)

expected output:
Kongo
North Carolina
King George V
```

Produce a sequence of items of the form `<Battle name=x><Ship name=y />...</Battle>` where `x` is the name of a battle and `y` is the name of a ship in the battle. There may be more than one `Ship` element in the sequence.
```xml
let $ships := doc("ships.xml")
let $battles := distinct-values(data($ships//Battle))
for $battle in $battles
return <Battle name = "{$battle}">
{
  for $ship in $ships//Ship[Battle]
  where data($ship/Battle) = $battle
  return <Ship name = "{data($ship/@name)}" />
}</Battle>

let $ships := doc("ships.xml")
let $battles := distinct-values(data($ships//Battle))
for $battle in $battles
return <Battle name = "{$battle}">
{
  for $ship in $ships//Ship[data(Battle) = $battle]
  return <Ship name = "{data($ship/@name)}" />
}</Battle>

expected output:
<Battle name="Guadalcanal">
  <Ship name="Kirishima"/>
  <Ship name="Washington"/>
</Battle>
<Battle name="Surigao Strait">
  <Ship name="Tennessee"/>
  <Ship name="California"/>
</Battle>
<Battle name="Den.mark Strait">
  <Ship name="Prince of Wales"/>
</Battle>
<Battle name="Malaya">
  <Ship name="Prince of Wales"/>
</Battle>
<Battle name="North Cape">
  <Ship name="Duke of York"/>
</Battle>
```
## Cars

List cars with "id" attribute from 2 to 3.
```xml
let $cars := doc("cars.xml")
let $range := 2 to 3
return $cars//car[@id = $range]

expected output:
<car id="2">Mazda</car>
<car id="3">Toyota</car>
```

## References
* https://www.w3schools.com/xml/xquery_intro.asp
* https://www.tutorialspoint.com/xquery/xquery_quick_guide.htm
* https://en.wikibooks.org/wiki/XQuery
