---
title: Fibonacci, and Doing it Fast
---
## Recursive
Ah Fibonacci, that beautiful sequence next door that gives us a glimpse into the elegance that mathematics, nature, and even programming can have. The recursive implementation in Swift is a fantastic example of clear, and readable code:

{% highlight swift linenos %}
func fibonacci (n: Int) -> Int
{
    precondition(n >= 0)

    if (n == 0 || n == 1) { return 1 }

    return fibonacci(n: n-1) + fibonacci(n: n-2)
}
{% endhighlight %}

That's it. Three little lines, and one of those most bad programmers don't even bother with.

When you stick this in an Xcode Playground, and start messing about with numbers, something becomes very apparent: this is one slow function. Stick a 100 into it, and everything stops being fun.

## Iterative
A good step at this point would be to do some benchmarking to see precisely how poorly the recursive approach scales, but before we get to that, we could play around with the iterative approach:

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

Another play around in Playground, and we see a huge speed improvement (admittedly at the cost of readability)

## Timing Execution
Eyeballing is one thing, but what if we wanted to know precisely how long each execution takes? Thanks to closures in Swift this can be done in a very reusable way, with a syntax that doesn't make you want to stab programming textbooks with blunt objects.

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
