---
layout: "en"
published: true
title: "Spelling correction with the Levensthein Distance algorithm"
author: Josephine Wright
author_github_username: jozr
---

# Spelling

How similar are the words 'hear' and 'here'?

There are many answers to that question. First and foremost, 'hear' and 'here' are indistinguishable in _sound_. Meaning, a fluent English speaker will pronounce 'here' just as they pronounce 'hear'. However, 'here' and 'hear' are different semantically or, rather, their definitions are distinct. 'Here' and 'hear' are also different in spelling. 

Spelling is a particularly important issue for our users, here, at Babbel. It's common for language learners to make spelling mistakes in a new language. It's also common for users to make typo mistakes while using a mobile or web application. As we're constantly looking for ways to improve our language learners' experience, these issues were addressed in a recent project by using a popular algorithm called Levenshtein Distance.

---

# Levenshtein Distance

There's no need to reinvent the wheel this time. Many years of computer science research have resulted in a vast library of algorithms to solve problems like spelling. 

In this post, I'll explore the Levenshtein Distance algorithm. It was discovered and documented by a scientist named Vladmir Levenshtein in 1965. Click [here](https://en.wikipedia.org/wiki/Vladimir_Levenshtein) to read more about him. 

At its core, the purpose of Levenshtein Distance is to quantify the differences between 2 strings.

## Edit Operations
According to the algorithm, there are 3 distinct edit operations that account for distance between 2 strings. They are as follows.
1. Insertion
1. Deletion
1. Substitution

Each operation has an edit distance of 1. For example, I will transform the word 'competers' to 'computer' in operational steps.
1. 'competers' to 'competer' (delete the 's')
1. 'competer' to 'computer' (substitute the first 'e' with 'u')

Above, there are 2 operations needed to convert 'competers' into 'computer'. Therefore the Levenshtein Distance between 'competers' and 'computer' is 2.

## Formula

For programmatic reasons, it's necessary to define the mathematical specifications. The Levenshtein Distance Wikipedia [article](https://en.wikipedia.org/wiki/Levenshtein_distance) includes the following mathematical formula.

![Levenshtein Formula](/images/levenshtein-formula.png "Levenshtein Formula")

It can look intimidating, so I'll walk through it step-by-step with an example.

## Example
As stated before, the words 'here' and 'hear' sound the same in English, but are spelled differently. I would like to know _how_ differently they are spelled or, rather, the edit distance in a numeric value.

First, I'll create a grid with the word 'hear' on the horizontal axis (`x` coordinates) and 'here' on the vertical axis (`y` coordinates). I'll also fill in the first corresponding row and column with an increasing count starting from 0 to represent the distance from the start of the word.

![Levenshtein Grid, Step 1](/images/levenshtein-grid-1.png "Levenshtein Grid, Step 1")

Then, I'll approach the first position `(1, 1)`. Because the first character of 'h' in 'here' matches the first character of 'h' in 'hear'. Then I'll look to the diagonally inferior coordinate `(x - 1, y - 1)` and copy its value, which is 0.

![Levenshtein Grid, Step 2](/images/levenshtein-grid-2.png "Levenshtein Grid, Step 2")

From there, I'll approach the next coordinate either horizontally or vertically adjacent to the `(1, 1)`. I'll choose the above box `(1, 2)`. Because the character 'e' from 'here' does not match the character 'h' from 'hear', I have to choose the minimum value from a neighboring box and then add 1. The minimum value would be the box below `(1, 1)` and its value is 0. And 0 + 1 is 1.

![Levenshtein Grid, Step 3](/images/levenshtein-grid-3.png "Levenshtein Grid, Step 3")

If I continue to do this for all the boxes, I end up with the following grid. The top right box `(4, 4)` becomes the final edit distance between the strings. Therefore, the edit distance between 'here' and 'hear' is 2.

![Levenshtein Grid, Step 4](/images/levenshtein-grid-4.png "Levenshtein Grid, Step 4")

It's also possible to arrive at this conclusion by noticing that there are 2 edits necessary to turn 'here' into 'hear'. I would have to substitute 2 characters 're' to 'ar', therefore the edit distance is 2. However, this grid example is necessary to introduce the logic for the algorithm.

## Implementation in Ruby
The best way, in my opinion, to understand an algorithm is to implement it in a programming language you know well. Here, at Babbel, we use lots of Ruby. I'll use Ruby for implementation, but it's possble to translate the code below into any imperative programming language.

### Matrices
Above, I used a grid to explain the algorithm step-by-step. It's essential to do the same for a programmatic solution as well. Grids are referred to as matrices in Ruby (and generally in Mathematics). The [Matrix class](https://ruby-doc.org/stdlib-2.5.1/libdoc/matrix/rdoc/Matrix.html) in Ruby is basically a 2D (2 dimensional) array with added mathematical functionality. I can access this class by requiring it.

```ruby
require 'matrix'
```

Positions in a Ruby's `Matrix` class are immutable. For the purpose of this algorithm, I need to mutate positions. To do this, I'll edit the class and expose a normally private method.

```ruby
class Matrix
  def []=(a, b, value)
    @rows[a][b] = value
  end
end
```

### Algorithm

Now that I have a mutable matrix, I can use it to populate the boxes like above. Matrices in Ruby define vertical positions increasingly from top to bottom, so position `(0, 0)` will refer to the upper left position (or boxes). Because of the deviation from normal algebraic order, I'll refer to the positions using `a` and `b`, not `x` and `y`.

```ruby
def mutable_matrix(string_a, string_b)
  Matrix.build(
    string_a.length + 1,
    string_b.length + 1
  ) do |row, col|
    if row == 0
      col
    elsif col == 0
      row
    else
      0
    end
  end
end

# mutable_matrix('aaa', 'bbbb')
# => Matrix[[0, 1, 2, 3, 4],
#           [1, 0, 0, 0, 0],
#           [2, 0, 0, 0, 0],
#           [3, 0, 0, 0, 0]]
```

Given that I now have a mutable matrix, I will build the first conditional. In the example of 'here' and 'hear' above, I first checked to see if the first characters were equal.

```ruby
def equal_character?(a, b)
  string_a[a - 1] == string_b[b - 1]
end
```

If the characters do not match, I need to find the minimum distance to the neighboring positions.

```ruby
def min_operation_value(a, b)
  [
    matrix[a, b - 1],    # Insertion:    Look directly above the current position
    matrix[a - 1, b],    # Deletion:     Look left to the current position
    matrix[a - 1, b - 1] # Substitution: Look diagonally lower from the current position
  ].min                  # Conclusion:   Return the minimum value from these 3 options
end
```

Now, I'll wrap these 2 methods in another method that will return a final cost for each position.

```ruby
def cost(a, b)
  if equal_character?(a, b)
    matrix[a - 1, b - 1] # Reference the value from the diagonally inferior position
  else
    min_operation_value(a, b) + 1 # Edit operations have an added cost of 1
  end
end
```

Now, I need to fill in every position in the matrix. This requires... looping! It's fine to loop over either string first.

```ruby
(1..string_b.length).each do |b|
  (1..string_a.length).each do |a|
    matrix[a, b] = cost(a, b)
  end
end
```

Finally, I need to grab the last value from the matrix to determine the overall edit distance between 2 strings.

```ruby
matrix[a_length, b_length]
```

Here are all of the above methods in a nice Ruby class.

```ruby
require 'matrix'

class Matrix
  def []=(a, b, value)
    @rows[a][b] = value
  end
end

class Levenshtein
  def initialize(string_a, string_b)
    @string_a = string_a
    @string_b = string_b
    @a_length = string_a.length
    @b_length = string_b.length
    @matrix   = mutable_matrix(a_length, b_length)
  end

  def distance
    return a_length if b_length == 0
    return b_length if a_length == 0

    (1..b_length).each do |b|
      (1..a_length).each do |a|
        matrix[a, b] = cost(a, b)
      end
    end

    matrix[a_length, b_length]
  end

  private

  def cost(a, b)
    if equal_character?(a, b)
      matrix[a - 1, b - 1]
    else
      min_operation_value(a, b) + 1
    end
  end

  def equal_character?(a, b)
    string_a[a - 1] == string_b[b - 1]
  end

  def min_operation_value(a, b)
    [
      matrix[a, b - 1],
      matrix[a - 1, b],
      matrix[a - 1, b - 1]
    ].min
  end

  def mutable_matrix(a_length, b_length)
    Matrix.build(a_length + 1, b_length + 1) do |row, col|
      if row == 0
        col
      elsif col == 0
        row
      else
        0
      end
    end
  end

  attr_reader :string_a, :string_b, :a_length, :b_length, :matrix
end

# Levenshtein.new('hear', 'here').distance
# => 2
```

# Damerau-Levenshtein Distance

Like most algorithms, Levenshtein Distance has some shortcomings. 

For example, the word 'receive' in English is easy to misspell. Often the 'i' and 'e' are switched in typing mistakes or otherwise. Using the Levenshtein Distance algorithm, the strings 'recieve' and 'receipt' are both 2 substitutional edit operations away from 'receive'. This is particularly problematic because it's much more likely for someone to spell 'receive' as 'recieve' while the word 'receipt' is semantically very different.

Another scientist named Frederick Damerau proposed an improvement to the Levenshtein Distance algorithm to account for character switches like the 'i' and 'e' in the case of 'receive' vs. 'recieve'.

## Transposition

Damerau called this new edit operation transposition. Transpositional edits add much more logic to the original Levenshtein Distance algorithm. I'll explore them in my next post.
