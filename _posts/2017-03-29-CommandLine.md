---
title: Reading Command Line Arguments in Swift
---
One thing I adore about `Swift` is that the answer to questions like:

*How do I access the command line arguments?*

is quite often:

{% highlight swift linenos %}
CommandLine.arguments
{% endhighlight %}

In a world founded on grandfather languages like `C++`, `Java` and `Python` where things can often be unintuitive, it's nice to see modern languages strive to be as brilliantly obvious as possible.

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

From that we have learned that the first argument is always the name of the application itself. If you navigate to that address in finder, you will find the actual compiled build of your project, which you can run from the terminal yourself and look all Mr Robot.

## Adding Arguments
To add your own custom arguments every time Xcode builds, you simply need to click on your project name in the top bar, and select "Edit Scheme".
![Xcode Template](/assets/2017-03-29-CommandLine/editScheme.png)

From there there you'll see a tab called "Arguments", where you can add what you wish. Anything you add will be passed in, using the order they appear in that list.

## Reading from a File
In the interest of ending on something useful, let's read from a file. Create a text file on your desktop, call it `funcCode.txt`, and write whatever naughty secret message you wish inside.

Add the following as your one and only command line argument:
`~/Desktop/funccode.txt`

Then update all the code in `main.swift` with the following

{% highlight swift linenos %}
func read(_ file: String) throws -> String
{
    return try String (contentsOfFile: file, encoding: String.Encoding.utf8)
}

assert(CommandLine.arguments.count == 2, "Incorrect number of command line arguments")

let filenameWithTilde = CommandLine.arguments[1]
let filename = NSString(string: filenameWithTilde).expandingTildeInPath

do
{
    let contents = try read (filename)
    print(contents)
}
catch { print ("Could not read from your file") }
{% endhighlight %}

Run it, check your console, and your secret message is not a secret anymore 😊
