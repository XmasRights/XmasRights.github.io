---
title: SearchTreeKit
---
*[SearchTreeKit on Github](https://github.com/XmasRights/SearchTreeKit)*

Last Christmas I spent a good amount of time watching the greatest series of Christmas films ever made. I am, of course, talking about Die Hards 1, 2, and 3 (it's a real shame they stopped making them after that).

In Die Hard 3 there is a fantastic "Water Jug" puzzle that pops up whereby Samuel L. Jackson and Bruce Willis need to produce exactly 4 litres of water using a fountain, a 3 litre jug and a 5 litre jug. There are plenty of examples of this problem in other languages
([Python](http://codereview.stackexchange.com/questions/78586/pouring-water-between-two-jugs-to-get-a-certain-amount-in-one-of-the-jugs), [C++](https://gist.github.com/douglas-vaz/5331386), [Haskell](http://hittaruki.info/post/water-jug-rewrite-with-haskell-part-I/), [Java](https://gist.github.com/oktapodi/5443952),...), but Google is rather vacant when it comes to doing it in `Swift`. Sounds like a blog post waiting to happen!

Now it's very easy to faff about in Playgrounds, whip up a solution in 30 mins, and get to work on your "Zuckerberg ain't got s--- on me hoodie", but this is not a hackathon. This problem is only interesting if all the numbers are variable, and all the logic is generic. Hence the birth of `SearchTreeKit`

## What are we doing here?
SearchTreeKit is a simple framework that allows for brute force iteration though a search space. You start by specifying a `SearchNode` to start with, creating a list of actions (`[(SearchNode)->SearchNode]`), and specifying some method that defines an end node (`(SearchNode)->Bool`). From there it's just a case of evaluating the current node with each of the valid actions, choosing one of the resulting nodes to explore, and repeat until you find what you're looking for. It's a very common paradigm that pops up in programming, though this implementation does not need to know the full search space before getting started.

## The Code
### SearchNode
Nodes just keep track of what your value is, and what you had to do to get here.
{% highlight swift linenos %}
public struct SearchNode<T: Equatable>
{
    public let value: T
    public let log:   String;

    public init(_ value: T)
    {
        self.value = value;
        self.log   = String()
    }

    public init(_ value: T, log: String)
    {
        self.value = value
        self.log   = log
    }
}
{% endhighlight %}

Few things to note here:
1. The use of `public` everywhere is an annoying consequence of writing a framework in `Swift` for the most part, leaving it blank (which will explicitly make everything `Internal`) will do you fine. To learn more, have a look at [Apple's Access Control documentation](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/AccessControl.html).
2. If you've never seen something like `<T: Equatable>`, it's nothing to panic over. It's called a [`Generic`](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Generics.html) and is simply a placeholder for some kind of Type, provided that type is `Equatable`. As a result, we can use `SearchTreeKit` to hold most data types (`Int`, `Float`, `Double`, `String`, or the soon to be created `Jug`).
3. We have a log to keep track of the journey taken to get to this node. I've opted for this over some kind of `Tuple` later down the line, because it's a much cleaner way of doing things in my opinion. Tuple aficionados, you do your thing :wink:

##SearchTreeKit
We have a Node  

## Future Developments
// TODO
