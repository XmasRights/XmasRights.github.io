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

Before proceeding, I'll neaten everything up by sticking it in a structure:
{% highlight swift linenos %}
struct Fibonacci
{
    public func recursive (n: Int) -> Int
    {
        precondition(n >= 0)

        if (n == 0 || n == 1) { return 1 }
        return recursive(n: n-1) + recursive(n: n-2)
    }

    public func iterative (n: Int) -> Int
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
}
{% endhighlight %}

## How Fast is it?
So now it's time to put all this to the test. When running both Fibonacci implementations on my modest 2011 iMac, I get the following results:
