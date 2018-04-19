---
title: Clean Swift - Embracing Types
---

Swift is a strongly typed language. This means seeing a particular type allows you to make some reassuring assumptions that you just don't get from your previous partners.

## Why NSArray is the worst

Consider the following `Objective-C` snippet:
{% highlight objc linenos %}
NSArray* data = [self getDataArray];
{% endhighlight %}

This looks simple enough, but there's a very important question to ask: "What format is the data?". There's no way to know without anxiously hunting around the codebase (ideally an hour before the deadline, in a cold sweat). What's worse is there aren't any real protections if you handle the data type incorrectly or if the type changes in future. There's also a the horrible chance that the array is `nil`, because `Objective-C` is the worst.

In Swift, it's much simpler; you will immediately know if you're dealing with `[Int]`, `[Float]`, or `[MyFunkyStruct]`, the moment you have your fingers on the array. If the data model changes, the code will fail at compile time, and you can easily deal with it. You can also guarantee the result will not be `nil`, becuase if it could be it'd be an `Optional`, and the compiler would order you to deal with that.

## Strings are the weakest type

Now let's have a look at some beautiful `Swift` code:

{% highlight swift linenos %}

func createItem(identifier: String) -> Item
{
  // ...
}

{% endhighlight %}

The code looks good, and you look good. The trouble is, everything below does NOT look good, and is perfectly valid:

{% highlight swift linenos %}

let a = createItem(identifier: "abc123")
let b = createItem(identifier: "helloWorld")
let c = createItem(identifier: "troll@funccode.com")
let d = createItem(identifier: "NicksRulez42")
let e = createItem(identifier: "120323102")
let f = createItem(identifier: "@,£)$!@@£@$@")
let g = createItem(identifier: "IWonderIfThereIsACharacterLimitToTheStringType.....")

{% endhighlight %}

Everything above is perfectly acceptable to the compiler, but it might not be acceptable to your data model. If you're encoding this identifier as a `json`, `csv`, or to use with a server, there may be formatting restrictions.

The worst way to let your team know how this should work is to use comments (slightly worse than doing nothing at all), as these can be misinterpreted, or are often not updated as the code evolves. The best way to do it is to make sure your compiler goes red when you enter the wrong thing.

Now, why on Earth am I writing a blog post about this? There's a trivial solution:

{% highlight swift linenos %}

func createItem(identifier: String) -> Item?
{
  guard isValid(identifier) else { return nil }
  // ...
  // *drops mic*
}

{% endhighlight %}

Well thank you for falling into my trap, my foolish inner monologue. This works fine if you're after a quick an easy hack, but it's not a long term solution. There are likely going to be other places where identifiers are created and validated, and adding `isValid` to every instance can quickly become annoying, and is incredibly easy to forget if someone else is working with your code.

## You get a type, and YOU get a type

Swift is a strongly typed language. Embrace it! 🤗

The best way to ensure that only valid data is passed into a function is to create a type that can only exist in a valid form.

{% highlight swift linenos %}

struct Identifier
{
    private let raw: String

    init?(string: String)
    {
        guard isValid(string) else { return nil }
        self.raw = string
    }

    private func isValid(_ string: String) -> Bool
    {
        // ...
    }
}

func createItem(identifier: Identifier) -> Item
{
    // ...
}

{% endhighlight %}

With this, those reassuring assumptions I mentioned earlier hold true. It's impossible to pass in an invalid `Identifier`, as it literally can't be created in an invalid format. Plus, unlike our `Objective-C` counterparts, you can't pass in a `nil` and expect to get away with it. This code will not compile if you pass in a random string. The cherry on top: it's super easy to extend the `Identifier` type with methods that ONLY affect identifiers, and not ALL Strings.

## Why is this Important?

String used as identifiers do have their advantages, and occur a lot as a result. However, remember that when you work as part of a team **the time others will spend reading your code is far greater than the time you spend writing it**. Developers need to be able to comprehend what's happening as quickly as possible, and the best way to do that is to have the compiler do your bidding.

If you have any questions or comments, feel free to leave a comment down below, or grab me on twitter at [@XmasRights](https://twitter.com/XmasRights).

In the meantime: thanks for reading,

Chris
