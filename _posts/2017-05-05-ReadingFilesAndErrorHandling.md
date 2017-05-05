---
title: Reading Files, and Handling Errors
---
In my last post I showed you how to add command line arguments to your projects, and used it to demonstrate reading in a file. This is, of course, the least intuitive way to read a file, if you've never delved into the world of terminals where you can make your computer look like the matrix.

If you just want to drag a file into your project and read it, you can, provided you are using a View based macOS or iOS application.

![Drag in the file](/assets/2017-05-05-ReadingFilesAndErrorHandling/dragIn.png)

![Use these settings](/assets/2017-05-05-ReadingFilesAndErrorHandling/settings.png)

From here, we're going to be good programming citizens and create a new struct that handles all the logic. Add a file called `SecretMessage.swift` and write the following code:

{% highlight swift linenos %}
struct SecretMessage
{
    static func read() -> String
    {
        guard let path     = Bundle.main.path(forResource: "SecretMessage", ofType: "txt"),
              let contents = try? String(contentsOfFile: path)
            else { print("Cound not find file"); return "" }

        return contents
    }
}
{% endhighlight %}

Calling this code is then a just a case of:
{% highlight swift linenos %}
SecretMessage.read()
{% endhighlight %}

## Error Handling
As much as I would love to drop the mic and walk away after writing such lovely code, this `struct`is desperately crying out for error handling. One mantra it's good to abide by in programming: "If you don't scrutinize your code and deal every place it could fail, it is very likely to do so in the most public and humiliating of ways". In my head, this happens during a demo to Scarlett Johansson, while she's going through a very weird phase in her affections.

The two places this code could go wrong is in finding the file, and reading the file, so we need an `enum` of `Error`s.
{% highlight swift linenos %}
enum SecretMessageError: Error
{
    case FileNotFound
    case CouldNotRead
}
{% endhighlight %}

We then need to bulk up the `read()` function to throw when necessary.
{% highlight swift linenos %}
struct SecretMessage
{
    static func read() throws -> String
    {
        guard let path = Bundle.main.path(forResource: "SecretMessage", ofType: "txt")
            else { throw SecretMessageError.FileNotFound }

        guard let contents = try? String(contentsOfFile: path)
            else { throw SecretMessageError.CouldNotRead }

        return contents
    }
}
{% endhighlight %}

Doesn't take too much added enthusiasm or finger strength to write the code here, but it does make everything immensely more robust, and is definitely worthy of a mic drop.
