---
layout: post
title: "Flattening Arrays in Ruby"
date:   2015-10-31
categories: algorithms
---

Ruby has the `Array.flatten` method which flattens any arbitrary multi-level nested array like so:

{% highlight ruby %}

array = [2, 3, [1, 6, [6, [7, 4, 0]], 9, 1], [4, 6, 3], 1]
array.flatten
=> [2, 3, 1, 6, 6, 7, 4, 0, 9, 1, 4, 6, 3, 1]

{% endhighlight %}

Here's one way of implementing `flatten` in Ruby.

{% highlight ruby %}

def flatten(array)
  array.reduce([]) do |result, item| 
    if item.kind_of? Array
      result + flatten(item)
    else
      result << item
    end
  end
end

test_cases = [
  [ 1, 1, 1, 1, 1, 1, 1, 1 ],
  [[3,4,[],5],5],
  [:apple, :banana, [1,[]], 'brocolli'],
  [],
  [[1,2,[3]],4],
  [1,2,3,"banana",[1,2,3],[]]
]

test_cases.each_with_index do |test, index|
  if test.flatten == flatten(test)
    puts "Test ##{index} passed :)"
  else
    puts "Test ##{index} failed :("
  end
end


{% endhighlight %}