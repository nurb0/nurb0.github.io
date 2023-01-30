---
layout: post
title: Default Hash Value in Ruby
---

### tldr
If you plan to modify the default value for the new keys in your Hash, use the proc constructor `Hash.new {|hash, key| ...}` to set the default value instead of `Hash.new(default_value)`.


### We Are The Champions
So you have a list of of World Cup winners in a format like this:
```ruby
champions = [
  ["Argentina", 2022],
  ["France", 2018],
  ["Germany", 2014],
  ["Spain", 2010],
  ["Italy", 2006],
  ["Brazil", 2002],
  ["France", 1998],
  # ...
]
```
and you want to figure out which country won the Cup in what years. You write a quick loop:
```ruby
winning_years = {}  # initialize Ruby's Hash
champions.each do |country, year|
  winning_years[country] ||= [] # Create a new list if we encounter a `country` for the first time.
  winning_years[country].push(year)
end
```
and it works beautifully.
```ruby
> winning_years
 => {"Argentina"=>[2022], "France"=>[2018, 1998], "Germany"=>[2014], "Spain"=>[2010], "Italy"=>[2006], "Brazil"=>[2002]}
```
But wait. Wouldn't it be nicer if we just provided a default value for the Hash entries and got rid of that empty Array assignment, i.e. `winning_years[country] ||= []`? After quick googling you revise:
```ruby
winning_years = Hash.new([]) # provide default value in the constructor
champions.each { |country, year| winning_years[country].push(year) }
```
So you pat yourself on the back, rerun the code and check the value of winning_years:
```ruby
> winning_years
 => {}
```
Wait, what? Where are my champions? Let's do a sanity test:
```ruby
> winning_years["Argentina"].push 2022
 => [2022, 2018, 2014, 2010, 2006, 2002, 1998, 2022]
```
Now I'm confused even more!

You quickly re-read the docs and realize that the same empty Array you have provided in the constructor Hash.new([]) is used when any non-existing key is fetched.

Hmmm, alright... Why is the Hash then empty instead of mapping each country to the same mega-list of years like this?!
```ruby
> winning_years
 => {"Argentina"=>[2022, 2018, 2014, 2010, 2006, 2002, 1998], "France"=>[2022, 2018, 2014, 2010, 2006, 2002, 1998]...}
```
Well that's because you didn't really add anything to the Hash.

Huh?

When you write `winning_years[country].push(year)` you are just pushing elements to the list that's returned by `winning_years[country]`. You haven't added any elements to the Hash. To add a new pair to a Hash you must use either `[]=`, e.g. `winning_years[country] = [year]` or its alias `store`, e.g. `winning_years.store(country, [year])`.

Let's fix this with a block constructor instead:
```ruby
winning_years = Hash.new { |h, k| h[k] = [] } # provide default value
champions.each { |country, year| winning_years[country].push(year) }
```
But don't get your hopes high when you run:
```ruby
> winning_years['Canada'].nil?
 => false # when did Canada win the World Cup?!
```
That proc will run any time you try to fetch a non-existing key and it uses `[]=` operator so the hash itself actually gets populated with the key and an empty list.

The lesson is be careful with setting the default value for Hash. That's all for today!
