---
title: Functions Returning Functions
---

"**Functions are first class citizens** in Swift", triumphantly announced by Apple following the introduction of their new language. On a linguistic level I have no idea what this means or what constitutes a "*second class citizen*". On a practical level, it simply means that functions can be treated like those variable things we all know and love.
{% highlight swift linenos %}
let someNumber: Int
let someBool: Bool
let someFunctionReturningInt: () -> Int
let someFunctionTakingIntReturningBool: (Int) -> Bool

let overkill: (Int) -> (Int) -> (Int) -> Bool)?
{% endhighlight %}

This is a bizarre thing to wrap your head around if you're more familiar with C++, Java, and the like, but it can be mighty useful.

I'll start with the pointless, but simple, code that everybody uses to demonstrate this:
{% highlight swift linenos %}
func adder (withIncrement increment: Int) -> (Int) -> Int
{
    return { $0 + increment }
}

let addFive = adder(withIncrement: 5)

print ("\(addFive(3))") // 8
print ("\(addFive(4))") // 9
print ("\(addFive(6))") // 11
{% endhighlight %}

NB: If you're having trouble with line 3, the version of the closure without any inference is `return { value in return value + increment }`.

We could have easily made a function of the form: `add(increment: to:)`, but if we know we're always going to add 5, or add a set of numbers, this saves a lot of time at the expense of very little mental capacity. Granted, it's highly unlikely you'll use the code above in any real code.

So here's an example that might actually do something for you 😊. I came across a [StackOverflow post](https://stackoverflow.com/questions/44252932/how-to-create-a-generator-in-swift/) looking for a generator-like structure in Swift. While Swift doesn't support python's `yield` keyword, we can muddle together this kind of behaviour with a function returning function.

{% highlight swift linenos %}
struct Countdown
{
    static func generator(startingAt: Int) -> () -> Int?
    {
        var start = startingAt
        return {
            start = start - 1
            return start > 0 ? start : nil
        }
    }
}

let countdown = Countdown.generator(withStart: 10)

while let i = b() { print ("\(i)") } // 10 9 8 7 6 .....
{% endhighlight %}

The ability to pass functions into other functions, and return functions from those functions is a strange construct to comprehend, and a delightful sentence to type. Use it well, and it allows for better structured code, as responsibilities can be further spread across your codebase to make things a lot [SMART](https://en.wikipedia.org/wiki/S.M.A.R.T.)er.

I've got a few posts coming up related to how I used this in a JSON parser, so sit tight for that.

In the meantime: thanks for Reading,
Chris
