# Objective-C Coding Conventions & Best Practices

Most of these guidelines are to match Apple's documentation and community-accepted best practices. Some are derived some personal preference. This document aims to set a standard way of doing things so everyone can do things the same way. If there is something you are not particularly fond of, it is encouraged to do it anyway to be consistent with everyone else.

This document is mainly targeted toward iOS development, but definitely applies to Mac as well.

## Reading this Document

I'd recommend installing the Mac App [Marked](https://itunes.apple.com/us/app/marked/id448925439?mt=12) and setting it up for Github syntax highlighting. See [this post](http://support.markedapp.com/kb/how-to-tips-and-tricks/using-marked-with-github-flavored-markdown-and-syntax-highlighting) for setup in Marked. Then just open this document and any other .md documents in Marked.

## Indenting

Indenting should be performed with spaces (configure your editor to emit spaces when using the tab key). 4 spaces should be used to indent a line.

The `+` or `-` defining the beginning of a method should always be hard-aligned to the left. The contents of the method should be indented by one level (4 spaces). One indent level should be added for each level of control structure within the method.

Comments should be at the same level of indent as the code that they are commenting on.

## Operators

```objective-c
NSString *foo = @"bar";
NSInteger answer = 42;
answer += 9;
answer++;
answer = 40 + 2;
```

The `++`, `--`, etc are preferred to be after the variable instead of before to be consistent with other operators. Operators separated should **always** be surrounded by spaces unless there is only one operand.


## Types

`NSInteger` and `NSUInteger` should be used instead of `int`, `long`, etc per Apple's best practices and 64-bit safety. `CGFloat` is preferred over `float` for the same reasons. This future proofs code for 64-bit platforms.

All Apple types should be used over primitive ones. For example, if you are working with time intervals, use `NSTimeInterval` instead of `double` even though it is synonymous. This is considered best practice and makes for clearer code.


## Methods

```objective-c
- (void)someMethod {
    // Do stuff
}


- (NSString *)stringByReplacingOccurrencesOfString:(NSString *)target withString:(NSString *)replacement {
    return nil;
}
```

There should **always** be a space between the `-` or `+` and the return type (`(void)` in this example). There should **never** be a space between the return type and the method name.

There should **never** be a space before or after colons. If the parameter type is a pointer, there should **always** be a space between the class and the `*`.

There should **always** be a space between the end of the method and the opening bracket. The opening bracket should **never** be on the following line.

There should **always** be two new lines between methods. This matches some Xcode templates (although they change a lot) and increase readability.


## Pragma Mark and Implementation Organization

An excerpt of a UIView:

```objective-c
#pragma mark - NSObject

- (void)dealloc {
    // Release
    [super dealloc];
}


#pragma mark - UIView

- (id)layoutSubviews {
    // Stuff
}


- (void)drawRect:(CGRect)rect {
    // Drawing code
}
```

Methods should be grouped by inheritance. In the above example, if some `UIResponder` methods were used, they should go between the `NSObject` and `UIView` methods since that's where they fall in the inheritance chain.

Pragma marks that are not separating the class or superclass methods should conform but are not limited to the following:

```objective-c
#pragma mark - Actions
#pragma mark - Getters
#pragma mark - Setters
#pragma mark - Notifications
#pragma mark - ScrollView delegate // replace ScrollView with any delegate
```


## Control Structures

There should **always** be a space after the control structure (i.e. `if`, `else`, etc).


### If/Else

```objective-c
if (button.enabled) {
    // Stuff
} 
else if (otherButton.enabled) {
    // Other stuff
} 
else {
    // More stuf
}
```

`else` statements should begin on next line as their preceding `if` statement.
    
```objective-c
// Comment explaining the conditional
if (something) {
    // Do stuff
}

// Comment explaining the alternative
else {
    // Do other stuff
}
```

If comments are desired around the `if` and `else` statement, they should be formatted like the example above.


### Switch

```objective-c
switch (something.state) {
    case 0: {
        // Something
        break;
    }
    case 1: {
        // Something
        break;
    }
    case 2:
    case 3: {
        // Something
        break;
    }
    default: {
        // Something
        break;
    }
}
```

Brackets are desired around each case. If multiple cases are used, they should be on separate lines. `default` should **always** be the last case and should **always** be included.


### For

Avoid C-style for loops and instead try to stick to Objective-C enumeration of sets. Doing so keeps syntax and conventions along line with Apple's standards.

In order to manipulate an object inside the enumeration, flag the object with <code>__block</code>.

```objective-c
__block NSObject *object = nil;
[someArray enumerateObjectsUsingBlock:^(id obj, NSUInteger idx, BOOL *stop) {
    object = obj;
    // do something
}];

[someDictionary enumerateKeysAndObjectsUsingBlock:^(NSString *key, id obj, BOOL *stop) {
    // do something
}];
```

If, for some reason, C-style for loops must be used, use the following conventions.

```objective-c
for (NSInteger i = 0; i < 10; i++) {
    // Do something
}


for (NSString *key in dictionary) {
    // Do something
}
```

When iterating using integers, it is preferred to start at `0` and use `<` rather than starting at `1` and using `<=`. Fast enumeration is generally preferred.


### While

```objective-c
while (something < somethingElse) {
    // Do something
}
```

## Import

**Always** use `@class` whenever possible in header files instead of `#import` since it has a slight compile time performance boost.

From the [Objective-C Programming Guide](http://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjectiveC/ObjC.pdf) (Page 38):

> The @class directive minimizes the amount of code seen by the compiler and linker, and is therefore the simplest way to give a forward declaration of a class name. Being simple, it avoids potential problems that may come with importing files that import still other files. For example, if one class declares a statically typed instance variable of another class, and their two interface files import each other, neither class may compile correctly.

### Header Prefix

Adding frameworks that are used in the majority of a project to a header prefix is preferred. If these frameworks are in the header prefix, they should **never** be imported in source files in the project.

For example, if a header prefix looks like the following:

```objective-c
#ifdef __OBJC__
    #import <Foundation/Foundation.h>
    #import <UIKit/UIKit.h>
#endif
```

`#import <Foundation/Foundation.h>` should never occur in the project outside of the header prefix.

## Properties

```objective-c
@property (nonatomic, retain) UIColor *topColor;
@property (nonatomic, assign) CGSize shadowOffset;
@property (nonatomic, retain, readonly) UIActivityIndicatorView *activityIndicator;
@property (nonatomic, assign, getter=isLoading) BOOL loading;
```

If the property is `nonatomic`, it should be first. The next option should **always** be `retain` or `assign` (or `strong` with ARC) since if it is omitted, there is a warning. `readonly` should be the next option if it is specified. `readwrite` should never be specified in header file. `readwrite` should only be used in class extensions. `getter` or `setter` should be last. `setter` should rarely be used.

See an example of `readwrite` in the *Private Methods* section.


## Dot Syntax

Dot syntax should **always** be used when getting or setting properties (public or private). It should **never** be used when calling a method. Even methods such as <code>-count</code>, though they seem to be properties, should be called with brackets. If ever unsure about dot syntax, unless a property is defined in a header file or Apple's documentation, it is best ot use brackets.


## Private Methods and Properties

MyShoeTier.h

```objective-c
@interface MyShoeTier : NSObject

@property (noatomic, retain, readonly) MyShoe *shoe;

//...

@end
```

MyShoeTier.m

```objective-c
#import "MyShoeTier.h"

@interface MyShoeTier ()

@property (nonatomic, retain, readwrite) MyShoe *shoe;
@property (nonaomic, retain) NSMutableArray *laces;

@end

@implementation MyShoeTier

//...

@end
```

Note: The above example provides an example for an acceptable use of a `readwrite` property.


## Extern, Const, and Static

```objective-c
extern NSString *const kMyConstant;
extern NSString *MyExternString;
static NSString *const kMyStaticConstant;
static NSString *staticString;
```

## Naming

In general, everything should be prefixed with a 2-3 letter prefix. Longer prefixes are acceptable, but not recommended.

It is a good idea to prefix classes with an application specific application if it is application specific code. If there is code you plan on using in other applications or open sourcing, it is a good idea to do something specific to your or your company for the prefix.

If you company name is *Awesome Buckets* and you have an application named *Bucket Hunter*, here are a few examples:

```objective-c
ABLoadingView // Simple view that can be used in other applications
BHAppDelegate // Application specific code
BHLoadingView // `ABLoadingView` customized for the Bucket Hunter application
```

## Enums

```objective-c
// Rarely useful
enum {
    Foo,
    Bar
};

// Acceptable but complicated
typedef NS_ENUM(NSInteger, TQOptionType) {
    TQOptionTypeA = 0,
    TQOptionTypeB = 1,
    TQOptionTypeC = 2,
    TQOptionTypeD = 3
};

// Preferred
typedef enum {
    SSLoadingViewStyleLight,
    SSLoadingViewStyleDark
} SSLoadingViewStyle;
```
