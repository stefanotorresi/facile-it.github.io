---
authors: ["elviro"]
comments: true
date: "2015-11-09"
draft: false
share: true
categories: [English, Swift, Objective-C, Functional programming, Monad]
title: "Optionals in Objective-C"
ita: "optionals-in-objective-c"

languageCode: "en-US"
type: "post"
aliases:
  - "/optionals-in-objective-c"
---

**Objective-C is not going anywhere**. While Swift is most certainly the new hotness for iOS and OS X programming, there are some concrete reasons to stick with Objective-C for a while:

- Objective-C based projects still need maintenance and new features to be added, and mixing Swift and Objective-C, while possible, **can be tricky** and possibly unconvenient, due to the dynamic nature of the latter;
- Swift is changing rapidly, has still some bugs and **performance problems**, and still lacks some features that professionals need, while Objective-C is mature and has a strong community;
- some may **prefer a more dynamic language**, and Objective-C support from Apple [is still strong](https://netguru.co/blog/objective-c-generics);

Personally, while I naturally lean towards a more static, [type-first](http://engineering.facile.it/type-first-development-in-swift/) approach to programming, from time to time I like to work in a more dynamic environment, so both for preference and for business needs, I still didn't put Objective-C completely away. But just after a few weeks of Swift I found myself missing one of the most powerful features of the language: [Optionals](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/TheBasics.html#//apple_ref/doc/uid/TP40014097-CH5-ID330).

## Table of contents
- [The `Optional` type](#the-optional-type)
- [Optionals in Swift](#optionals-in-swift)
- [Back to Objective-C](#back-to-objective-c)
- [Example: a JSON parser](#example-a-json-parser)
- [Conclusion](#conclusion)

<a name="the-optional-type"></a>
## The `Optional` type

A Optional is a *generic* type, that is, a type that's dependent  (actually, *parametric*) on another **subtype**, so whe can have, for example, a `Optional<String>` or a `Optional<Int>`: thanks to some syntactic sugar, those types are written in Swift as `String?` and `Int?`. So, what is an `Optional`? It's a representation   of a value that could be there, and be of a particular subtype, or could not, therefore being **nil**: by *wrapping* the subtype into an `Optional`, the Swift compiler knows that the value could be nil, and complain in the cases where we are using an `Optional` where a value that's **never** nil is expected. By itself, this simple feature grants us a lot of type-safety for free: for example, in Swift we can't initialize a non-optional value with nil, and considering that, for the Swift's compiler, a value cannot be used before being initialized, by saying that a value's type is `String` the compiler will assure us that the value will **always** be a string, no matter what.

In contrast, Objective-C objects can always be nil, and if we expect something not to be nil, we are forced to check for the actual existence of everything at any time. But actually, that's not necessarily a big deal: it's a classic **tradeoff** of dynamic languages, and it can be even considered a *feature* by some people: in using a dynamically typed language, we are always expected to design our APIs with [late binding](https://en.wikipedia.org/wiki/Late_binding) in mind. Also, the new [Objective-C nullability specifiers](https://developer.apple.com/swift/blog/?id=25) will help the compiler trigger warnings when nil is passed to a property or method parameter when `nonnull` is expected. But the problem is still there: we need a lot of boilerplate to always check for *nullness*, and that can produce error-prone, less readable code. The need for this boilerplate is missing in Swift, and that's thanks to the nature of the `Optional` type: in fact, it works in the same way as the `Maybe` type in Haskell, or the `Option` type in Scala, that is, `Optional` is actually a **monad**.

The concept of *monad* is inherited by functional programming from [category theory](https://en.wikipedia.org/wiki/Category_theory) and there are plenty of [introductions](https://wiki.haskell.org/Monad) [about it](https://www.haskell.org/tutorial/monads.html) [on the web](http://learnyouahaskell.com/a-fistful-of-monads), so I'm not going into the specifics here: let me just say that a monad is a *computational context*, that is, a specification for the way some *computations* need to be resolved. Applying a certain transformation to a monad will result in another instance of the same monad, different from the first, based on the specific kind of monad. For example, applying some transformation to an `Optional` monad will result in applying the same transformation to the contained value if it's there, or absolutely nothing if the `Optional` contains nil: in both cases, the transformation's output will be another `Optional`, possibly of different subtype.

<a name="optionals-in-swift"></a>
## Optionals in Swift

Consider the following Swift code:

```Swift
func makeOptionalIntFrom(value: Int, ifTrue: Bool) -> Int?
{
  return ifTrue ? value : nil
}

let optionalInt = makeOptionalIntFrom(3, ifTrue: true)
print(optionalInt.dynamicType) /// prints 'Optional<Int>'

let toString: Int -> String = { "\($0)" }

let optionalString = optionalInt.map(toString)
print(optionalString.dynamicType) /// prints 'Optional<String>'
```

In the example, `optionalInt` is a `Int` wrapped into an `Optional`, thus having type `Optional<Int>` (Swift's syntactic sugar allows us to write `Int?`), and by applying the `toString` function to that `Optional` we get an instance of `Optional<String>`. We can see that, to actually apply the `toString` function to the `Optional`, we passed the function to the `map` method: this operation is usually called [*lifting* a function](https://wiki.haskell.org/Lifting), because the function `toString`, of type `Int -> String`, is *lifted* into the type `Optional<Int> -> Optional<String>`.

As we can see, no conditional statements were used in manipulating the optional integer: by *lifting* our transformations with the `map` method, we can apply them directly to the `Optional` instances, and we can also chain them pretty easily, for example:

```Swift
func makeOptionalIntFrom(value: Int, ifTrue: Bool) -> Int?
{
  return ifTrue ? value : nil
}

let optionalInt = makeOptionalIntFrom(3, ifTrue: true)
print(optionalInt.dynamicType) /// prints 'Optional<Int>'

let doubled: Int -> Int = { $0*2 }

let toString: Int -> String = { "\($0)" }

let optionalDoubledString = optionalInt.map(doubled).map(toString)
print(optionalDoubledString.dynamicType) /// prints 'Optional<String>'
```

If we want to express the opposite operation, we run into a problem: transforming a `String` into a `Int` is not always considered possible by Swift (for example, no numbers in the string), therefore the operation is optional. In fact:

```Swift
func makeOptionalStringFrom(value: String, ifTrue: Bool) -> String?
{
  return ifTrue ? value : nil
}

let toInt: String -> Int? = { Int($0) }

let anotherOptionalString = makeOptionalStringFrom("3", ifTrue: true)

let anotherOptionalInt = anotherOptionalString.map(toInt).map(doubled)
/// this won't compile!
```

The problem here is that `toInt` is a function of type `String -> Optional<Int>`, and lifting the function to the `Optional` "world" will turn its type into something like `Optional<String> -> Optional<Optional<Int>>`, that is, an optional integer wrapped into another optional: in *monadic* terminology, to lift functions that transforms the wrapped type into another instance of the same monad, we need a `flatMap` operation:

```Swift
let anotherOptionalInt = anotherOptionalString.flatMap(toInt).map(doubled)
/// this is fine
```

So, for the `Optional` type, `flatMap` works like `map` but for functions that transform the wrapped type into another `Optional`. In reading the code, `map` and `flatMap` can be basically treated as the same: they both indicate a transformation, and the fact that computations passed to `flatMap` are of slightly different type can be considered an implementation detail.

Actually, Swift treats these types **a little differently** than in other languages: for example, `map` and `flatMap` are usually *free functions* in most *functional* languages, while in Swift they are **methods**. In general, Swift encourages the use of methods and **method chaining** instead of free functions composed with special operators: it's mostly a matter of philosophy, but in bringing these features back to Objective-C, the method-based approach is going to be really useful because there's no easy syntax for free functions in the language, while methods have the signature clean and readable syntax that we're used to.

<a name="back-to-objective-c"></a>
## Back to Objective-C

Let's implement the `Optional` class in Objective-C:

```ObjectiveC
///Optional.h

#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface Optional : NSObject

+ (Optional*)with:(id _Nullable)value;

- (Optional*)map:(id(^)(id))mapBlock;
- (Optional*)flatMap:(Optional*(^)(id))flatMapBlock;

@end

NS_ASSUME_NONNULL_END
```

```ObjectiveC
///Optional.m

#import "Optional.h"

@interface Optional ()

@property (strong, nonatomic, nullable) id value;

@end

@implementation Optional

+ (Optional*)with:(id _Nullable)value
{
  Optional* optional = [Optional new];
  optional.value = value;
  return optional;
}

- (Optional *)map:(id  _Nonnull (^)(id _Nonnull))mapBlock
{
  if (self.value != nil)
  {
    return [Optional with:mapBlock(self.value)];
  }
  return self;
}

- (Optional *)flatMap:(Optional* _Nonnull (^)(id _Nonnull))flatMapBlock
{
  if (self.value != nil)
  {
    return flatMapBlock(self.value);
  }
  return self;
}

@end
```

As we can see, the `with:` class method will create an `Optional` object by wrapping some other object, of unspecified class: because there's no true *generics* in Objective-C, we won't be able to make our Optional type as type-safe as in Swift, but as we'll see, it won't be a big deal and we'll still be able to get some really **powerful features**. The `map:` method will check if the wrapped object is not nil, and apply the transformation expressed by the `mapBlock` block only if the object is actually there: in this case, Objective-C nullability specifiers really help us in designing our APIs, because we clearly specified that `mapBlock` will accept and will return non-nil objects only. The `flatMapBlock` block will return instead another `Optional`.

We can test this by essentially recreating the same Swift example:

```ObjectiveC
///Test.h

#import <Foundation/Foundation.h>

@interface Test : NSObject

+ (void)testOptional;

@end
```

```ObjectiveC
///Test.m

#import "Test.h"
#import "Optional.h"

@implementation Test

+ (void)testOptional
{
  Optional* optionalInt = [self makeOptionalIntFrom:@3 ifTrue:YES];
  Optional* optionalDoubledString = [[optionalInt
                                      map:[self doubled]]
                                     map:[self toString]];
  NSLog(@"%@", optionalDoubledString);
}

+ (Optional*)makeOptionalIntFrom:(NSNumber*)fromInt ifTrue:(BOOL)ifTrue
{
  return [Optional with:ifTrue ? fromInt : nil];
}


+ (NSNumber*(^)(NSNumber*))doubled
{
  return ^NSNumber*(NSNumber* value)  {
    return @(value.integerValue*2);
  };
}

+ (NSString*(^)(NSNumber*))toString
{
  return ^NSString*(NSNumber* value)  {
    return [NSString stringWithFormat:@"%@", value];
  };
}

@end
```

Once we applied our transformations, we need a way to *unwrap* the object inside our `Optional`: in Swift this is done at language level, with some syntactic sugar, while in Objective-C we can follow the standard conventions of the functional programming community, that is, we can *get* the wrapped object via a `get` method.

```ObjectiveC
///Optional.h

- (id _Nullable)get;
```

```ObjectiveC
///Optional.m

- (id)get
{
  return self.value;
}
```

Of course the `get` method will return an `id _Nullable`, which means that the returned object can (obviously) be nil. A frequent pattern when dealing with `nil` objects is replacing them with "default" versions: that can be useful when we don't really care about the data contained in an object, but we don't want to be exposed to the risk of possibly null references. Once again, with the `Optional` class we can avoid conditional statements and express the "defaulting" mechanism in a more declarative way: we just need to add a `getOrElse:` method, that will return the wrapped object if it's there, or will return a default object, that we'll pass to the method:

```ObjectiveC
///Optional.h

- (id)getOrElse:(id(^)())elseBlock;
```

```ObjectiveC
///Optional.m

- (id)getOrElse:(id  _Nonnull (^)())elseBlock
{
  if (self.value != nil)
  {
    return self.value;
  }
  return elseBlock();
}
```

As we can see, `getOrElse:` will **not** return a `_Nullable` object, and that's thanks to `elseBlock`: we're not passing the actual default object here, but a block that will produce one; this way, we can get the default object **lazily**, because if the wrapped object is not nil, the default object doesn't need to be allocated.

For the rest of the article we'll work on a more real-world example, and in doing so we'll add more features to the `Optional` class, making it more and more powerful.

<a name="example-a-json-parser"></a>
## Example: a JSON parser

Suppose we want to create an app that shows movie informations: we are retrieving those informations in JSON format from a website like [myapifilms](http://api.myapifilms.com). Each movie has a JSON representation like the following:

```JSON
{
    "countries": [
        "USA"
    ],
    "directors": [
        {
            "name": "Frank Darabont",
            "nameId": "nm0001104"
        }
    ],
    "filmingLocations": [
        "St. Croix",
        "U.S. Virgin Islands"
    ],
    "genres": [
        "Crime",
        "Drama"
    ],
    "idIMDB": "tt0111161",
    "languages": [
        "English"
    ],
    "metascore": "80/100",
    "originalTitle": "",
    "plot": "Andy Dufresne is a young and successful banker whose life changes drastically when he is convicted and sentenced to life imprisonment for the murder of his wife and her lover. Set in the 1940s, the film shows how Andy, with the help of his friend Red, the prison entrepreneur, turns out to be a most unconventional prisoner.",
    "ranking": 1,
    "rated": "R",
    "rating": "9.3",
    "releaseDate": "19941014",
    "runtime": [
        "142 min"
    ],
    "simplePlot": "Two imprisoned men bond over a number of years, finding solace and eventual redemption through acts of common decency.",
    "title": "The Shawshank Redemption",
    "type": "Movie",
    "urlIMDB": "http://www.imdb.com/title/tt0111161",
    "urlPoster": "http://ia.media-imdb.com/images/M/MV5BODU4MjU4NjIwNl5BMl5BanBnXkFtZTgwMDU2MjEyMDE@._V1_SX214_AL_.jpg",
    "votes": "1,533,914",
    "writers": [
        {
            "name": "Stephen King",
            "nameId": "nm0000175"
        },
        {
            "name": "Frank Darabont",
            "nameId": "nm0001104"
        }
    ],
    "year": "1994"
}
```

Of course we want to define our own internal representation with a value object, change some structure and ignore the parts that we consider not relevant. Here's a possible interface for the `Movie` class:

```ObjectiveC
NS_ASSUME_NONNULL_BEGIN

@interface Movie : NSObject

@property (copy, nonatomic, readonly) NSString* title;
@property (copy, nonatomic, readonly) NSNumber* rating;
@property (copy, nonatomic, readonly) NSNumber* year;
@property (copy, nonatomic, readonly) NSNumber* lengthInMinutes;

+ (Movie*)withJSONDict:(NSDictionary*)dict;

@end

NS_ASSUME_NONNULL_END
```

The class method `withJSONDict:` will create an instance of `Movie` from a JSON dictionary like the one showed before: as we can see from the interface declaration, we decided to not allow nil values for any of the properties; but when dealing with JSON, we must consider **every possibility**, including missing keys, null values or wrong types. We're going to use the `Optional` type to parse the JSON dict in a clean, declarative and error-proof way.

Let's start with the title. From the JSON, the title should be given by the "title" key, and should be a string (`NSString` in Objective-C), so we can get the title via the following:

```ObjectiveC
 NSString* title = [[[[Optional
                        with:[dict objectForKey:@"title"]]
                       
                       flatMap:^Optional*(id title) {
                         return [Optional with:[title isKindOfClass:[NSString class]] ? title : nil];
                       }]
                      
                      flatMap:^Optional*(NSString* title) {
                        return [Optional with:title.length > 0 ? title : nil];
                      }]
                     
                     getOrElse:^NSString*{
                       return @"NO TITLE";
                     }];
```

The first `flatMap:` step means that we want the title to be of `NSString` class. It's convenient to abstract this operation directly into the `Optional` constructor, because we're going to do it all the time:

```ObjectiveC
+ (Optional *)with:(id)value as:(Class)valueClass
{
  if ([value isKindOfClass:valueClass])
  {
    return [Optional with:value];
  }
  return [Optional with:nil];
}
```

The second `flatMap:` step means that if the found string is of length 0 (that is, empty), we are going to treat it as unknown. Actually, this step will act as a **filter**, because it will only *let pass* the strings that are not empty; with that in mind, let's add a `filter:` method to the `Optional` class:

```ObjectiveC
///Optional.h

- (Optional*)filter:(BOOL(^)(id))filterBlock;
```

```ObjectiveC
///Optional.m

- (Optional*)filter:(BOOL (^)(id _Nonnull))filterBlock
{
  return [self flatMap:^Optional*(id value) {
    if (filterBlock(value))
    {
      return self;
    }
    else
    {
      return [Optional with:nil];
    }
  }];
}
```

The `filter:` method takes as parameter a block that returns a `BOOL` based on the wrapped value: internally, `filter:` will actually call `flatMap:`, with a conditional expression to check if the `filterBlock` succeeds or fails.

Finally:

```ObjectiveC
NSString* title = [[[Optional
                       with:[dict objectForKey:@"title"]
                       as:[NSString class]]
                      
                      filter:^BOOL(NSString* string) {
                        return string.length > 0;
                      }]
                     
                     getOrElse:^NSString*{ return @"NO TITLE"; }];
```

So, no conditional statements, and very few lines of code, completely declarative.

For the `rating` and `year` parameters we need to add a `map` to convert the `NSString` into an `NSNumber`:

```ObjectiveC
NSNumber* rating = [[[[Optional
                         with:[dict objectForKey:@"rating"]
                         as:[NSString class]]
                        
                        filter:^BOOL(NSString* string) {
                          return string.length > 0;
                        }]
                       
                       map:^NSNumber*(NSString* stringValue) {
                         return [NSDecimalNumber decimalNumberWithString:stringValue];
                       }]
                      
                      getOrElse:^NSNumber*{ return @0; }];
  movie.rating = rating;
  
  NSNumber* year = [[[[Optional
                       with:[dict objectForKey:@"year"]
                       as:[NSString class]]
                      
                      filter:^BOOL(NSString* string) {
                        return string.length > 0;
                      }]
                     
                     map:^NSNumber*(NSString* stringValue) {
                       return [NSDecimalNumber decimalNumberWithString:stringValue];
                     }]
                    
                    getOrElse:^NSNumber*{ return @0; }];
```

The `lenghtInMinutes` parameter is a little tricky; from the JSON we can see that the value is represented like this:

```JSON
"runtime": [ 
	"142 min" 
]
```

So we expect an array, of which we're only interested in the first element (thus, the array must not be empty), that has to be a string; of this string we only need the first part, removing the ` min` portion. Luckily, we already have all the tools for the job:

```ObjectiveC
  NSNumber* lengthInMinutes =
  [[[[[[Optional
        with:[dict objectForKey:@"runtime"]
        as:[NSArray class]]
       
       flatMap:^Optional*(NSArray* array) {
         return [Optional with:[array firstObject]];
       }]
      
      map:^NSString*(NSString* string) {
        return [string
                stringByReplacingOccurrencesOfString:@" min"
                withString:@""];
      }]
     
     filter:^BOOL(NSString* string) {
       return string.length > 0;
     }]
    
    map:^NSNumber*(NSString* stringValue) {
      return @([stringValue integerValue]);
    }]
   
   getOrElse:^NSNumber*{ return @0; }];
```

Actually, something is missing: the method `[array firstObject]` returns the first object of the array if the array has at least one element, or nil if the array is empty: there's no indication that the first object is actually a `NSString`, so we need to add a `filter` step to insure that the object is an instance of the correct class. Let's do it:

```ObjectiveC
 NSNumber* lengthInMinutes =
  [[[[[[[Optional
         with:[dict objectForKey:@"runtime"]
         as:[NSArray class]]
        
        flatMap:^id(NSArray* array) {
          return [Optional with:[array firstObject]];
        }]
       
       filter:^BOOL(id value) {
         return [value isKindOfClass:[NSString class]];
       }]
      
      map:^NSString*(NSString* string) {
        return [string
                stringByReplacingOccurrencesOfString:@" min"
                withString:@""];
      }]
     
     filter:^BOOL(NSString* string) {
       return string.length > 0;
     }]
    
    map:^NSNumber*(NSString* stringValue) {
      return @([stringValue integerValue]);
    }]
   
   getOrElse:^NSNumber*{ return @0; }];
```

As we can see, when we work with the `Optional` class, adding a condition simply means **adding a step** in the linear **flow of expressions**.

<a name="conclusion"></a>
## Conclusion

Bringing back the `Optional` type from Swift to Objective-C allowed us to build a JSON parsing routine, often something hard to maintain and error-prone, in a simple, linear and readable way. Of course there's **plenty of cases** where working with optionals can give us the same advantages: by composing `map`, `flatMap` and `filter` operations we can express our intent in a clean and readable way. But this is not just about the `Optional` type: some readers may have probably noticed that many languages (including Swift) allow operations with the same names on lists and arrays, and the meanings are also the same: `map` and `flatMap`, in particular, are **generic concepts** related to monads (in fact, the array type is also a monad) and allow declarative compositions of transformations. This concept is at the core of **functional programming**, and I think there's plenty of reasons to apply similar concepts to a more object-oriented environment: they will help the programmer in reasoning about the code and express the intent in a more declarative way.

The code for the `Optional` class used throughout the article can be found on [GitHub Gist](https://gist.github.com/broomburgo/e318228a5f7d6a605e82).
