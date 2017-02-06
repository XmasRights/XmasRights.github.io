---
title: SearchTreeKit
---
During the holidays I spent many hours watching the greatest series of Christmas films ever made. I am, of course, talking about Die Hards 1, 2, and 3 (it's a real shame they never went any further than that 🙄).

In Die Hard 3 there is a fantastic "Water Jug" puzzle that pops up in which Samuel L. Jackson and Bruce Willis need to produce exactly 4 litres of water using a fountain, a 3 litre jug, and a 5 litre jug. There are plenty of examples of solutions to this problem in other languages
([Python](http://codereview.stackexchange.com/questions/78586/pouring-water-between-two-jugs-to-get-a-certain-amount-in-one-of-the-jugs), [C++](https://gist.github.com/douglas-vaz/5331386), [Haskell](http://hittaruki.info/post/water-jug-rewrite-with-haskell-part-I/), [Java](https://gist.github.com/oktapodi/5443952),...), but Google is rather vacant when it comes to doing it in `Swift`. Sounds like a blog post waiting to happen!

Now it's very easy to faff about in Playgrounds, whip up a solution in 30 mins, and get to work on your "Zuckerberg ain't got s--- on me hoodie", but this is not a hackathon. This problem is only interesting if all the numbers are variable, and all the logic is generic. Hence the birth of `SearchTreeKit`

## What are we doing here?
SearchTreeKit is a simple framework that allows for brute force iteration though a search space. You start by specifying a `SearchNode`, create a list of actions (`[(SearchNode)->SearchNode]`), and then define a function that evaluates a particular `SearchNode` to identify the end (`(SearchNode)->Result`). From there it's just a case of branching off from the start node using all available actions, and then working your way down the tree till you find the delicious solution. It's a very common paradigm that pops up in programming, though this implementation does not need to know the full search space before getting started.

## SearchTreeKit
*[SearchTreeKit on Github](https://github.com/XmasRights/SearchTreeKit)*

### SearchNode
Nodes keep track of what your value is, and what you had to do to get here.
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
1. `public` everywhere is an irritating consequence of writing a framework in `Swift`. For the most part, leaving it blank (which will implicitly make everything `Internal`) will do you fine. To learn more, have a look at [Apple's Access Control documentation](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/AccessControl.html).
2. If you've never seen something like `<T: Equatable>`, it's nothing to panic over. It's called a [`Generic`](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Generics.html) and is simply a placeholder for some kind of Type, provided that type is `Equatable`. As a result, we can use `SearchTreeKit` to hold most data types (`Int`, `Float`, `Double`, `String`, or the soon to be created `Jug`).
3. We have a log to keep track of the journey taken to get to this node. I've opted for this over some kind of `Tuple` later down the line, because it's a much cleaner way of doing things in my opinion. Tuple aficionados, you do your thing, no judgement here 😉

### Actions
We now need to define what a valid operation is for our tree. This is housed in a list, defined as `[(SearchNode<T>)->SearchNode<T>?]`. Simply put, you need a series of functions that take a particular node, and return a new node with an altered value, and an updated log of what was done to it.

I've chosen to go for `Optionals` here becuase (in addition to it making everything look 1337 by using `Generics` and `Optionals` in the same function), it makes it super simple to cut out actions that don't do anything and avoid pointless loops. For example, attempting to fill a jug that is already full is stupid, and we should just return nothing. When we get down to the search algorithms, this will prevent many headaches as all `for-in` loops we use while just silent fizzle out to nothing and we can carry on with our lives.

### SearchTreeKit
We have a Node, have our toys to play with it, now we just need some technique. `SearchTreeKit` at the time of writing has two methods to work our way to an answer:
1. Depth First Search
2. Breadth First Search

Depth first search constantly works its way down one branch of the tree until it hits a dead end or an the desired answer. If it needs to, it then backtracks and keeps plowing down the tree until there's nothing left to explore. For example, with the Water Jug problem it will keep filling the first jug until it can't do it anymore, and will then work it's way backwards to see what else it can do. For certain types of problems it is wonderfully fast, and it's great if you just want a solution of any kind.

Breadth first search will apply all available actions to the start node, and create an `Array` of results. It will then do the same for each item in that `Array` to form the next level, and carry on until it finds the end. This is predictably slower, but is best at finding the minimum number of moves necessary to get the result. Since what we're doing is far from rocket science, it's the perfect algorithm for us.

Now for the code. The first thing we need is a little helper function that can apply an `Array` of actions to a `SearchNode`

{% highlight swift linenos %}
private static func apply<T: Equatable> (actions: [(SearchNode<T>)->SearchNode<T>?], toNode node: SearchNode<T>) -> [SearchNode<T>]
{
  var output = [SearchNode<T>]()

  for action in actions
  {
      if let result = action(node)
      {
          output.append(result)
      }
  }
  return output
}
{% endhighlight %}

### Depth First Search
If you read my previous post, you know that there are many situations where you should avoid recursion if you want lightning fast performance. Well....let's use recursion!!
{% highlight swift linenos %}
public enum Result { case Pass, Fail, Continue }

public static func depthFirstSearch<T: Equatable> (start: SearchNode<T>, actions: [(SearchNode<T>)->SearchNode<T>?], end isEnd: (SearchNode<T>) -> Result) -> SearchNode<T>?
{
    let nextNodes = apply(actions: actions, toNode: start)

    for next in nextNodes
    {
        switch isEnd (next)
        {
        case .Pass:
            return next

        case .Fail:
            return nil

        case .Continue:
            if let result = depthFirstSearch(start: next, actions: actions, end: isEnd) { return result }
        }
    }
    return nil
}
{% endhighlight %}

Besides the rather nasty function declaration, there isn't too much complexity here. We use our handy `apply()` function to create an `Array` of nodes for the next level of the tree. For each node we check to see if we `Pass` or `Fail` to end the execution there, and if we need to continue we just recursively call the same function on each node in that list.

### Breadth First Search
{% highlight swift linenos %}
public static func breadthFirstSearch<T: Equatable> (start: SearchNode<T>, actions: [(SearchNode<T>)->SearchNode<T>?], end isEnd: (SearchNode<T>) -> Result) -> SearchNode<T>?
{
    var visited: Set  <SearchNode<T>> = [start]
    var queue:   Array<SearchNode<T>> = [start]

    repeat
    {
        let node = queue.remove(at: 0)

        switch isEnd(node)
        {
        case .Pass:
            return node

        case .Fail:
            continue

        case .Continue:
            let nextNodes = apply(actions: actions, toNode: node)

            for next in nextNodes
            {
                if (!visited.contains(next))
                {
                    visited.insert(next)
                    queue  .append(next)
                }
            }

        }
    } while (queue.count > 0)
    return nil
}
{% endhighlight %}

Breath First Search requires keeping a queue of `SearchNode`s and working along it in an orderly fashion. We begin by adding the start node to the head of the queue and jumping into a `do-while` (or `repeat-while` in `Swift`) loop until we find the end. With each loop, we see if we have ended or failed, as before, but if we encounter a `.Continue`, we call the `apply()` function and add each of the results to the queue. It then follows that each level of the tree will be exhaustively searched before the next level moves to the front of the line. The only additional treat I've given this function is a `Set` of visited nodes. This just boosts performance as we won't bother evaluating values we've already seen.

For both of these algorithms, the result of each execution will be a `SearchNode` with the desired result, and a log of the journey taken or a simple `nil` if nothing could be found (oh how I love `Optionals`).

## Using SearchTreeKit
Since this post has already gotten pretty long, and uncomfortably expositional, I'll cut it short here and save the practical usage of `SearchTreeKit` to more interesting posts in future. Links coming soon.

## Future Developments
I wrote this framework to be simple to use, and great at doing very basic tasks. There are many additional bells and whistles that I'd like to add to it, but many programmers better than myself have fallen down similar rabbit holes and been lost forever. For now it's best to enjoy it while it's young, shiny, and reliable, and we can think about heuristics for another lazy Saturday night.
