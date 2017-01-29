---
title: Fibonacci, and Doing it Fast
---
As a programmer, there are often moments when you have to question who is truly in charge in the computer-developer relationship. When a boring and repetitive task presents itself and you diligently proceed without question, I have to imagine your computer's laughter circuits are chuckling away with the satisfaction that they've managed to avoid another irritating endeavour.

One of those tasks that I find myself doing over and over and over again when optimising code (or more likely figuring out why my code is so Goddamn slow) is calculating execution time. It's simple enough to write, copy, paste, print to console, delete, and repeat over and over and over and over again, but it's not absurd to think that spending the 4 minutes to make something a bit more elegant might be in order.

Thanks to closures in Swift this can be done in a very reusable way, with a syntax that doesn't make you want to stab programming textbooks with blunt objects.

## Working out Execution Time
{% highlight swift linenos %}
func executionTime(function: ()->()) -> Measurement<UnitDuration>
{
    let start = DispatchTime.now()
    function()
    let end = DispatchTime.now()

    let nanoseconds = end.uptimeNanoseconds - start.uptimeNanoseconds
    let seconds     = Double(nanoseconds) / 1_000_000_000

    return Measurement(value: seconds, unit: UnitDuration.seconds)
}
{% endhighlight %}

With any luck, you'll never have to write that code again. From then on it's simply a case of:
{% highlight swift linenos %}
let time = executionTime {
    print("Hello World")
}

let seconds = time.converted(to: .seconds)
print("Printing Hello World takes \(seconds) seconds")
{% endhighlight %}

So now, let's put this code to the test, with my favourite number sequence: Fibonacci.

## Fibonacci
### Recursive
Ah Fibonacci, that beautiful sequence next door that gives us a glimpse into the elegance that mathematics, nature, and even programming can have. The recursive implementation in Swift is a fantastic example of clear, and readable code:
{% highlight swift linenos %}
func fibonacci (n: Int) -> Int
{
    precondition(n >= 0)

    if (n == 0 || n == 1) { return 1 }
    return fibonacci(n: n-1) + fibonacci(n: n-2)
}
{% endhighlight %}

That's it. Three little lines, one of which most bad programmers don't even bother with.

### Iterative
There's also the iterative approach, which is far less attractive:
{% highlight swift linenos %}
func fibonacci (n: Int) -> Int
{
    precondition(n >= 0)

    if (n == 0 || n == 1) { return 1 }

    var a = 1, b = 1
    for _ in 1...(n - 1)
    {
        let c = b

        b = b + a
        a = c
    }
    return b
}
{% endhighlight %}

### Memoized
There is one final implementation I'm going to throw onto the table: memoization. Memoization is a process by which the result of a function for a given input is stored locally. The next time you wish to call the function with that same input, the answer has already been worked out, and we can take a quick glance at our previous homework rather than starting out from scratch. This comes at the cost of memory usage, but depending on the nature of your app, it's a pretty great option if your functions are running rather sluggishly.

If you want to do memoization properly there is a generic function that will power up any of your existing code. You can learn about that in a [great blog post on Scott Logic](http://blog.scottlogic.com/2014/09/22/swift-memoization.html). For the purposes of this experiment, I'm going to be naughty and knock out a quick method, specific to this codebase:

{% highlight swift linenos %}
static func memoized (n: Int) -> Int
{
    precondition(n >= 0)

    if (n == 0 || n == 1) { return 1 }

    if let result = memoizedResults[n]
    {
        return result
    }
    else
    {
        let result = memoized(n: n-1) + memoized(n: n-2)
        memoizedResults[n] = result
        return result
    }
}

private static var memoizedResults = [Int:Int]()
{% endhighlight %}

In order to properly test memoization, I will run it twice in any experiment.

### The Fibonacci Struct
Before proceeding, I'll neaten everything up by sticking it in a structure:
{% highlight swift linenos %}
struct Fibonacci
{
    @discardableResult static func recursive (n: Int) -> Int
    {
        precondition(n >= 0)

        if (n == 0 || n == 1) { return 1 }
        return recursive(n: n-1) + recursive(n: n-2)
    }

    @discardableResult static func iterative (n: Int) -> Int
    {
        precondition(n >= 0)

        if (n == 0 || n == 1) { return 1 }

        var a = 1, b = 1
        for _ in 1...(n - 1)
        {
            let c = b

            b = b + a
            a = c
        }
        return b
    }

    @discardableResult static func memoized (n: Int) -> Int
    {
        precondition(n >= 0)

        if (n == 0 || n == 1) { return 1 }

        if let result = memoizedResults[n]
        {
            return result
        }
        else
        {
            let result = memoized(n: n-1) + memoized(n: n-2)
            memoizedResults[n] = result
            return result
        }
    }

    private static var memoizedResults = [Int:Int]()
}
{% endhighlight %}

## How Fast is it?
So now it's time to put all this to the test. When running each Fibonacci implementations on my modest 2012 iMac, we get the following results:

![Fibonacci Execution Time Graph](/assets/ExecutionTime/fibGraph1.png)

Dismissing the leaps my computer makes as other processes cry out for attention, what we can take away from these results is that iterative slaughters the recursive approach, even with some memoized magic sprinkled in. This is exponentially more impressive once you join the keen eyed among you that noticed the logarithmic scale on the Y-Axis.

We also see, as expected, the memoized approach significantly outperforming recursive. This is because this experiment started at zero and work its way up. The previous two values used to calculate the new one will already be in our `Dictionary`, and it's just two lookups and an addition to get the new one. When we run the memoized approach a second time, each result is essentially just the time it takes to read from a `Dictionary`.

While it would be easy to pack our bags and accept that the most readable code isn't always the fastest, it's worth taking one final glance at the iterative vs memoized battle.

![Fibonacci Execution Time Graph](/assets/ExecutionTime/fibGraph2.png)

Here we are looking at fractions of milliseconds difference, hence why the anomalies are more pronounced. All three test runs performed well, but for sufficiently large values of n, the memoized approach finally beats out iterative. (Note: I stopped the test at 90, because any higher and the Fibonacci value becomes larger than `Double` can support)

#So What Does This Mean?
These results are unsurprising. Recursion is a great structure that typically produces beautiful, readable code, but that comes at the expense of a huge call stack, and a lot of overhead for your CPU to deal with. Iterative is incredibly fast for this particular operation, and is probably the way to go for most applications that will use Fibonacci in some way. However, if you're going to be repeatedly calling this function, and you don't have to think about memory usage in kilobytes, memoization is a great thing to look into for your project.

*[Download the ExecutionTime project used to run the experiment here](/assets/ExecutionTime/ExecutionTime.zip)*
