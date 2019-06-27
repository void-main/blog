---
comments: true
title:  "What I have learned from terminal-notifier"
date:   2013-07-15 14:34:57
categories: 
  - learning
---

The [terminal-notifier](https://github.com/alloy/terminal-notifier) is a cute command-line tool written by [alloy](https://github.com/alloy) that sends *User Notifications* for OS X systems running 10.8 or later.

There's not so much code down there, but still I've learned a lot, and here's what I've got:

### Architecture Overview

There are two major parts in this project. To deliver `NSUserNotification` or respond to user click event, we need a Cocoa App even though there will never be a display window or something. According to the builder, it is currently packaged as an application bundle, because NSUserNotification does not work from a ‘Foundation tool’.

To provide a easier way to install and use terminal-notifier, the project also provides a ruby gem, whose source is located at `Ruby/` folder of the project.

As for the communication, I've managed to draw a small graph below:

![Overview](/assets/terminal-notifier-overview.png)

The ruby gem is in charge of providing a command-line interface. Once the arguments are collected, it will call the Cocoa App's bin file with extra arguments. By then, the Cocoa App will read the arguments and deliver the notification to user.

That's quite straight-forward, yet the result is amazingly powerful. Let me tell you the technique details I've learned below.

### Lesson #1 - Are we running this app on OS X 10.8 or later?
Easy, but you need to know two commands, namely `uname` and `sw_vers`. You can refer to `man` to see what exactly they are doing, briefly, `uname` tells the name of the operating system, which is `Darwin` for OS X, and `sw_vers` returns the version info for OS X.

And here's the code from terminal-notifier:

```
@available = `uname`.strip == 'Darwin' && `sw_vers -productVersion`.strip >= '10.8'
```

This is ruby code, and it's easy to understand I believe.

### Lession #2 - Don't forget some useful variables
Just a quick flash back, `$0` refers to the name of current running script, `$:` is the same as `$LOAD_PATH` and `$?` is the status code of the last process terminated. For a detailed list, please refer [this article](http://www.tutorialspoint.com/ruby/ruby_predefined_variables.htm).

### Lession #3 - NSUserDefaults will help you with the command-line arguments
Parsing command-line arguments is not an unusual job, there are surely a lot of ways to do this in all programming languages. As I'm new to Cocoa Programming, I DID NOT know that one can read command-line arguments using `NSUserDefaults`. Imagine how I felt lost while reading the following code:

```
NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];

NSString *subtitle = defaults[@"subtitle"];
NSString *message  = defaults[@"message"];
NSString *remove   = defaults[@"remove"];
NSString *list     = defaults[@"list"];
NSString *sound    = defaults[@"sound"];
```

Who the heck fills the `standardUserDefaults` for us with the command-line arguments? It's impossible. 

But, indeed, it is possible. I've found this article that explains the defaults domain and their precedence. As it turns out, it is the `NSArgumentDomain` that does the heavy-lift job of parsing the arguments and store them into the `standardUserDefaults`. Note that this is quite a good way, espically overriding the system wide defaults with command line arguments. It is a way of hacking something, I guess.

### Lession $4 - Easier API with `objectForKeyedSubscript:`
What if you want to grab some object using `[]` operator, by that I mean instead of retrieving object like `dict[@"key"]` rather than `[dict objectForKey: @"key"]`. And again, it's simple, just implement the `objectForKeyedSubscript:` method for the class or add category with some predefined classes. Here's the example from terminal-notifier:

```
@interface NSUserDefaults (Subscript)
@end

@implementation NSUserDefaults (Subscript)
- (id)objectForKeyedSubscript:(id)key;
{
  return [self objectForKey:key];
}
```

With this code snippet, we add the power of subscription to NSUserDefaults, and it's fun to use.

### Conclusion
I've learned the 4 major lessons from terminal-notifier, but there are many other lessions I've not mentioned. And to write better code, we should all read more *f**king* code.
