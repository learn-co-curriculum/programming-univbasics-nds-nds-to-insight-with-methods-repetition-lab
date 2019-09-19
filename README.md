# Derive Insight from NDS Using Methods...Repeated

## Learning Goals

* Create a "First-Order" method

## Introduction

In the previous lab we saw how wrapping "raw" use of Ruby primitive
instructions (REEPETITION, `Array` methods, etc.) into methods makes it easier
for humans to reason about code.

However, we can create code that is easier to maintain, clearer and more
beautiful if we wrap "First-Order" methods inside of _other methods_. In the
same way that we used "raw" `[]` and `Hash` and `Array` methods _inside_
"First-Order" methods, we can use "First-Order" methods _inside_ other methods.

When code is structured this way there are many benefits:

* It's easier to debug
* It's more obvious to read
* It's more easy to get help with online

## Starting Point

First, let's start with some common "starter" code.

```ruby
vm = [[[{:name=>"Vanilla Cookies", :pieces=>3}, {:name=>"Pistachio Cookies",
:pieces=>3}, {:name=>"Chocolate Cookies", :pieces=>3}, {:name=>"Chocolate Chip
Cookies", :pieces=>3}], [{:name=>"Tooth-Melters", :pieces=>12},
{:name=>"Tooth-Destroyers", :pieces=>12}, {:name=>"Enamel Eaters",
:pieces=>12}, {:name=>"Dentist's Nighmare", :pieces=>20}], [{:name=>"Gummy Sour
Apple", :pieces=>3}, {:name=>"Gummy Apple", :pieces=>5}, {:name=>"Gummy Moldy
Apple", :pieces=>1}]], [[{:name=>"Grape Drink", :pieces=>1}, {:name=>"Orange
Drink", :pieces=>1}, {:name=>"Pineapple Drink", :pieces=>1}], [{:name=>"Mints",
:pieces=>13}, {:name=>"Curiously Toxic Mints", :pieces=>1000}, {:name=>"US
Mints", :pieces=>99}]]]

def total_snack_pieces_on_spinner(nds, row_index, column_index)
  coordinate_total = 0
  inner_len = nds[row_index][column_index].length
  inner_index = 0
  while inner_index < inner_len do
    coordinate_total += nds[row_index][column_index][inner_index][:pieces]
    inner_index += 1
  end
  coordinate_total
end

grand_piece_total = 0
row_index = 0
while row_index < vm.length do
  column_index = 0
  while column_index < vm[row_index].length do
    grand_piece_total += total_snack_pieces_on_spinner(vm, row_index,
column_index)
    column_index += 1
  end
  row_index += 1
end

p grand_piece_total #=> 1192
```

## Wrap a First-Order Method

Looking at the starter code, the inner-most part is _still_ not so easy to
reason about. Let's zoom in to it:

```ruby
# Non-Runnable!
while row_index < vm.length do
  column_index = 0
  while column_index < vm[row_index].length do
    grand_piece_total += total_snack_pieces_on_spinner(vm, row_index, column_index)
    column_index += 1
  end
  row_index += 1
end
```

That inner-most `while` _means_ "go to each spinner on the row, and total it
up".  That is, "**total up the row based on its spinners**." Wouldn't it be
GREAT if there were a method called `total_snack_pieces_in_row`? Again, there's
nothing stopping us from making it!

***But here's the crucial insight about how we'll write that code***: a row is
made up of spinners. So all we have to do to total a row's snack count is add
together the snack counts of its spinners. Let's update the code:

```ruby
vm = [[[{:name=>"Vanilla Cookies", :pieces=>3}, {:name=>"Pistachio Cookies", :pieces=>3}, {:name=>"Chocolate Cookies", :pieces=>3}, {:name=>"Chocolate Chip Cookies", :pieces=>3}], [{:name=>"Tooth-Melters", :pieces=>12}, {:name=>"Tooth-Destroyers", :pieces=>12}, {:name=>"Enamel Eaters", :pieces=>12}, {:name=>"Dentist's Nighmare", :pieces=>20}], [{:name=>"Gummy Sour Apple", :pieces=>3}, {:name=>"Gummy Apple", :pieces=>5}, {:name=>"Gummy Moldy Apple", :pieces=>1}]], [[{:name=>"Grape Drink", :pieces=>1}, {:name=>"Orange Drink", :pieces=>1}, {:name=>"Pineapple Drink", :pieces=>1}], [{:name=>"Mints", :pieces=>13}, {:name=>"Curiously Toxic Mints", :pieces=>1000}, {:name=>"US Mints", :pieces=>99}]]]

def total_snack_pieces_on_spinner(nds, row_index, column_index)
  coordinate_total = 0
  inner_len = nds[row_index][column_index].length
  inner_index = 0
  while inner_index < inner_len do
    coordinate_total += nds[row_index][column_index][inner_index][:pieces]
    inner_index += 1
  end
  coordinate_total
end

def total_snack_pieces_in_row(nds, row_index)
  grand_row_total = 0
  column_index = 0
  while column_index < nds[row_index].length do
    grand_row_total += total_snack_pieces_on_spinner(nds, row_index, column_index)
    column_index += 1
  end
  grand_row_total
end

grand_piece_total = 0
row_index = 0
while row_index < vm.length do
  grand_piece_total += total_snack_pieces_in_row(vm, row_index)
  row_index += 1
end

p grand_piece_total #=> 1192
```

## Wrap an Additional Method

Suddenly all that's left in our code that's not "inside" a method is the
following:

```ruby
# Non-Runnable

grand_piece_total = 0
row_index = 0
while row_index < vm.length do
  grand_piece_total += total_snack_pieces_in_row(vm, row_index)
  row_index += 1
end
```

But this bit of code has a purpose. What does this code _mean_. What is it
trying to accomplish?

Isn't it "`total_snack_pieces_in_machine`?" And, just as before, isn't the
total number of pieces in the machine the sum of all the pieces in the rows
(which we just conveniently gained the ability to calculate)? We're now going
to "wrap" `total_snack_pieces_in_row` into a new method:
`total_snack_pieces_in_machine`.

> It might be tempting to call `total_snack_pieces_on_spinner` a "Second-Order"
> method. And it certainly is! Most programmers would recognize this idea. But
> programmers don't generally go around applying labels like this to methods.
> To do so would be, frankly, silly at some point. 
>
> If we wrap multiple Second-Order methods do we get a Third-Order? A
> Fourth-Order? A HundredAndFirst Order?  The main idea here is that methods
> can **nest** just like data structures.

```ruby
vm = [[[{:name=>"Vanilla Cookies", :pieces=>3}, {:name=>"Pistachio Cookies", :pieces=>3}, {:name=>"Chocolate Cookies", :pieces=>3}, {:name=>"Chocolate Chip Cookies", :pieces=>3}], [{:name=>"Tooth-Melters", :pieces=>12}, {:name=>"Tooth-Destroyers", :pieces=>12}, {:name=>"Enamel Eaters", :pieces=>12}, {:name=>"Dentist's Nighmare", :pieces=>20}], [{:name=>"Gummy Sour Apple", :pieces=>3}, {:name=>"Gummy Apple", :pieces=>5}, {:name=>"Gummy Moldy Apple", :pieces=>1}]], [[{:name=>"Grape Drink", :pieces=>1}, {:name=>"Orange Drink", :pieces=>1}, {:name=>"Pineapple Drink", :pieces=>1}], [{:name=>"Mints", :pieces=>13}, {:name=>"Curiously Toxic Mints", :pieces=>1000}, {:name=>"US Mints", :pieces=>99}]]]

def total_snack_pieces_on_spinner(nds, row_index, column_index)
  coordinate_total = 0
  inner_len = nds[row_index][column_index].length
  inner_index = 0
  while inner_index < inner_len do
    coordinate_total += nds[row_index][column_index][inner_index][:pieces]
    inner_index += 1
  end
  coordinate_total
end

def total_snack_pieces_in_row(nds, row_index)
  grand_row_total = 0
  column_index = 0
  while column_index < nds[row_index].length do
    grand_row_total += total_snack_pieces_on_spinner(nds, row_index, column_index)
    column_index += 1
  end
  grand_row_total
end

def total_snack_pieces_in_machine(nds)
  grand_piece_total = 0
  row_index = 0
  while row_index < nds.length do
    grand_piece_total += total_snack_pieces_in_row(nds, row_index)
    row_index += 1
  end
  grand_piece_total
end

p total_snack_pieces_in_machine(vm) #=> 1192
```

Wow! That's really handy! We have **three** helpful methods that will help hide
away complexity and keep our code readable and maintainable.

From the perspective of generating _insight_, our code knows what _insight_ we
want and makes it easy to find by calling a method whose name immediately
suggest the _insight_ we want: `total_snack_pieces_in_machine`. The method
then, internally, uses _other_ methods to help keep itself clean and simple:

```text
total_snack_pieces_in_machine
 \-> total_snack_pieces_in_row
    \-> total_snack_pieces_on_spinner
```

## Lab

In the lab you're going to write a method that uses First-Order methods.

## Conclusion

Whenever you have complex data that needs to be "transformed," "massaged," or
"changed," let _methods_ help your human brain keep track of the constituent
actions that need to happen to create the _insights_ you need. Compare this
last bit of code with the first "raw `[]` and `Array` methods" implementation.
Which version would _you_ rather debug? Which is easier to talk about and ask
for help with?

What's even more interesting, is that as your collection of "First-Order"
methods grows, you'll find yourself writing methods that mix their outputs
together to create new _insights_. Eventually, you might have so many
well-written, tiny, clear "First-Order" methods that reasoning about,
explaining, and extending your code is simple. That's the goal.
