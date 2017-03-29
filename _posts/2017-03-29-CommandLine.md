---
title: Using the Command Line
---
One thing I adore about `Swift` syntax is that the answer to questions like:

*How do I read the command line arguments?*

is quite often:

{% highlight swift linenos %}
CommandLine.arguments
{% endhighlight %}

In a world where I'm more used to stubborn grandfather languages like `C++` and `Python`, it's nice to see modern languages strive to be as brilliantly obvious as possible.

## ./Project Hello World
Open up a new Xcode Project, and make sure you've got `macOS` and `Command Line Tool` selected.
![Xcode Template](/assets/2017-03-29-CommandLine/template.png)

From there you can add the following code to the `main.swift` file.

{% highlight swift linenos %}
print("You have \(CommandLine.arguments.count) arguments")

for (i, arg) in CommandLine.arguments.enumerated()
{
    print("\(i) -> \(arg)")
}
{% endhighlight %}

Run that, and you'll see this in the console output:
{% highlight swift linenos %}
// You have 1 arguments
// 0 -> /Users/YOUR_NAME/Library/Developer/Xcode/DerivedData/PROJECT_NAME-GIBBERISH_CODE/Build/Products/Debug/PROJECT_NAME
// Program ended with exit code: 0

{% endhighlight %}
