---
title: "Searching in a list using a set of choices"
excerpt: "How to search in a list using a set of choices as early return conditions"
classes: wide
tags: python, snippets
---
Based on the following code snippet, a colleague asked if there may be a better way to proceed with:
- Iterating over a list of things
- Returning the wanted thing only if is in another list of choices
- Execute an "early return"

{% highlight python %}
wanted_thing = None

for thing in things:
   correct_name = (
       "awesome" in thing.name
       and "great" in thing.name
   )
   if first_choice in thing.name and correct_name:
       wanted_thing = thing
       break
else:
   for thing in things:
       correct_name = (
           "awesome" in thing.name
           and "great" in thing.name
       )
       if second_choice in thing.name and correct_name:
           wanted_thing = thing
           break
{% endhighlight %}

The current implementation is quite good in explaining what the expected behaviour should be:
1. iterate over the main list of things
2. if the first choice is found, return the wanted element
3. else iterate over the list of things again but using the second condition as a check 
4. if element found, return
5. if element not found repeat using third, fourth, n-th exit conditions...

A very nahive approach could be:

{% highlight python %}
choices = ["first", "second", "third"]
things = ["thing", "second", "thing2", "first", "thing3", "thing4", "third"]

def return_on_condition(things, conditions):
    for condition in conditions:
        for thing in things:
            if condition in thing:
                return thing
    return False
{% endhighlight %}

<iframe src="https://trinket.io/embed/python/48196a3144?start=result" width="100%" height="300" frameborder="0" marginwidth="0" marginheight="0" allowfullscreen></iframe>

This will result in a working example that executes in $$O(n * m)$$ witn $$n$$ number of choices and $$m$$ number of things to check.

The return in the for-loop will act as an "early return" process, meaning that the whole search will stop as soon as there is a condition that is matched.



