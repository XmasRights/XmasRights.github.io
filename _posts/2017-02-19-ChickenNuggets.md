---
title: Chicken Nuggets
---
In the interest of not waffling on for too long, the theme of this post is: "Just F*** Do It" (or JFDI).

In [my last post](http://www.funccode.com/SearchTreeKit) I introduced to you [`SearchTreeKit`](https://github.com/XmasRights/SearchTreeKit), a simple way to find answers a set of possible actions. If you care at all about the implementation go have a read of my [`SearchTreeKit` post](http://www.funccode.com/SearchTreeKit), if you ain't got time for that, and just want to see how to use it: you've come to the right place.

## Chicken Nuggets, and Frobenius
Once again, I'm writing a bit of Swift in response to something I saw on TV. On QI, Sandi Toksvig mentioned [Frobenius](https://en.wikipedia.org/wiki/Ferdinand_Georg_Frobenius), and his delightful method for solving specific types of infinite series. She explains it using McDonalds Chicken Nuggets, and the fact that you can only buy them in multiples of 6, 9 and 20. As a result, if you want to have your fill of exactly 43 nuggets without wasting a single one, you're out of luck; as there is no combination of orders that will give you that exact number. Anything higher than that, and you're golden.

## How to buy too many Chicken Nuggets
Let's put this to the test with `SearchTreeKit`, and see if we can figure out how to buy any number of Chicken Nuggets. As mentioned before, all you need is:
- A start (`SearchNode`)
- Some actions (`(SearchNode) -> SearchNode?`)
- An endpoint (`(SearchNode) -> SearchTreeKit.Result`)

### Start
One tool that will make everything look very sexy indeed is a `typealias`
{% highlight swift linenos %}
typealias Nuggets = SearchNode<Int>
let start  = Nuggets(0)
{% endhighlight %}
`typealias` essentially just tells swift to substitute one type name for another, meaning we can do much less writing and explain everything in a much more descriptive way. I could quite easily use Ints for this, but we're talking about food here, if we can call a Nugget a Nugget, why not do just that? 😉

### Actions
There are many ways to generate an Array of actions. I've opted for a function that creates the list and returns it, like so:
{% highlight swift linenos %}
func actions() -> [(Nuggets)->Nuggets?]
{
    func buy(_ nuggets: Int, _ n: Nuggets) -> Nuggets?
    {
        let description = "Buy \(nuggets) nuggets (\(n.value + nuggets))\n"
        return Nuggets(n.value + nuggets, log: n.log + description)
    }

    let buySix    = { (n: Nuggets) in return buy(6,  n) }
    let buyNine   = { (n: Nuggets) in return buy(9,  n) }
    let buyTwenty = { (n: Nuggets) in return buy(20, n) }

    return [buySix, buyNine, buyTwenty]
}
{% endhighlight %}

By day I'm a C++ programmer, and seeing code like this feels like I'm having an affair with someone who does far naughtier things that what I'm used to. There's two things going on:
1. A nested function is used to create a generic `buy()` action that can allow for any number of nuggets to be bought. It's good to do this because there will be a lot of repeated code if we decide just to write each three functions from scratch. Keeping this function nested is also good practice, because it's a method that should not be accessible externally, and only needs to exist in here.
2. The next few lines contain inline closures. Closures are mighty little tools, and inline closures allows you to be exceptionally lazy. Since return type can be inferred, you don't need to include it, so just calling an instance of the generic `buy()` function with hardcoded values is enough to create all three valid actions. Since functions are first class citizens in Swift, we can simple assign them to a `let` constant, and return them in an `Array` as if they were any old primitive variable.

### End
{% highlight swift linenos %}
func end (nuggets: Int) -> (Nuggets) -> SearchTreeKit.Result
{
    return { (n: Nuggets) -> SearchTreeKit.Result in
        switch n.value
        {
        case _ where n.value == nuggets: return .Pass
        case _ where n.value >  nuggets: return .Fail
        default:                         return .Continue
        }
    }
}
{% endhighlight %}

The end can be achieved with another inline closure. Here we just stick our desired number of nuggets into a Switch statement, and use `Swift`'s pattern matching syntax to compare this value of Nuggets with condition. Remember, when using pattern matching your switch statement needs to be exhaustive, so the `default:` case is required. You'll also notice we didn't use inferred return types in this one. By specifying the return `enum` explicitly, we only need to put `.Pass` in as a return value, as Swift will fill in the rest (otherwise, each return case will need the full `SearchTreeKit.Result.Pass`).

### Mixing them all together
To find the fewest steps necessary to buy a specific number of Chicken Nuggets:
{% highlight swift linenos %}
func buy(nuggets: Int) -> Nuggets?
{
    precondition(nuggets > 0)
    return SearchTreeKit.breadthFirstSearch(start: Nuggets(0), actions: actions(), end: end(nuggets: nuggets))
}

let restaurant = Restaurant()
guard let result = restaurant.buy(nuggets: 42) else
{
    print("Cannot buy exactly \(nuggets) nuggets\n")
    return
}

print("To buy \(nuggets) nuggets")
print(result.log)

// To buy 42 nuggets
// Buy 6 nuggets (6)
// Buy 9 nuggets (15)
// Buy 9 nuggets (24)
// Buy 9 nuggets (33)
// Buy 9 nuggets (42)
{% endhighlight %}

And there you have it. A few simple lines of code, and you can kick off your journey towards diabetes in the most precise way possible.

## Available on GitHub
All the code for this project is available in my Chicken-Nuggets-Frobenius repo. Find this, and many other projects with impossibly witty titles *[here](https://github.com/XmasRights/Chicken-Nuggets-Frobenius)*
