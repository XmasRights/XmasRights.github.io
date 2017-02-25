---
title: Third Party Frameworks
---
A good rule for programmers is to try to avoid writing the same piece of code twice. That's why frameworks are so important, even if you're working by yourself.

*Read if you're a developer*
Fewer lines of code == less to do -> more time to watch Firefly

*Read if you're in QA*
Fewer lines of code means fewer bugs, and fixes made in one place will apply to every project.

*Read if you're a in management*
Fewer lines of code means we don't have to pay developers as much to make this stuff happen. Let's push the deadline forward 😈

The best guide I know of when it comes to writing frameworks is from the fantastic folks at [Ray Wenderlich](https://www.raywenderlich.com/126365/ios-frameworks-tutorial), so head on over there for further reading. This post will be a [tl;dr](https://en.wikipedia.org/wiki/TL;DR) summary for those of you that just want to get it done quickly.

## Writing Code
Every function and class you write in Swift has an implicit level of access, defining which other things it can get its claws into. Each Xcode project you write can also be thought of as it's own "module", and other Xcode projects can import this code and use the stuff within depending on what restrictions you've applied. The various levels are:
- **`Open`**: ANYTHING can use me, subclass me, have their way with me. Look at me everyone!!
- **`Public`**: Hi everyone, feel free to use me and call my functions, but I'm pretty cool with who I am, so don't subclass me unless you're from my module.
- **`Internal`**: We are one module, and we should all get along. If you see any other module, hide! They're coming to take our variables!!
- **`Fileprivate`**: We need to build a wall, to keep all other files out of this class. Seriously, unless you're in the same `.Swift` file as me, you're a bad hombre.
- **`Private`**: IF YOU'RE NOT IN MY CLASS, GET THE F*** OFF ME!

If you want a more eloquently put version of that, have a look at [Apple's documentation](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/AccessControl.html).

Everything you write will implicitly be `Internal` unless you specify otherwise. This makes sense, as it means everything in every `.Swift` file you write will have access to everything else. However, if you take one Xcode project, and stick it in another, your new project won't actually see anything from the older one.

To change the access control of your classes, variables, and functions, just add one of the keywords specified above before it (in lower case):
{% highlight swift linenos %}
public class SomePublicClass {}
internal class SomeInternalClass {}
fileprivate class SomeFilePrivateClass {}
private class SomePrivateClass {}

public var somePublicVariable = 0
internal let someInternalConstant = 0
fileprivate func someFilePrivateFunction() {}
private func somePrivateFunction() {}
{% endhighlight %}

## 
