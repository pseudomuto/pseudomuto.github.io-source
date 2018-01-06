+++
aliases = ["/development/2011/07/29/nsdata-from-odata/"]
date    = "2011-07-29T20:50:09-05:00"
tags    = ["ios", "objective-c", "odata"]
title   = "NSDate From OData"
+++

Recently I've been working on an iOS project that loads data that we publish via the [Open Data
Protocol](http://www.odata.org/).

At first everything was working perfectly. I was using [JSONKit](https://github.com/johnezang/JSONKit) and
[NSOperation](http://developer.apple.com/library/ios/#documentation/cocoa/reference/NSOperation_class/Reference/Reference.html)
to download and process the information. That was until I had to get a `DateTime` object...which resulted in an epic
fail!

It turns out that WCF Data Services publish DateTime objects as a new Date object represented by the time in
milliseconds (not seconds) since January 1, 1970. Anyone familiar with [EPOCH
dates](http://en.wikipedia.org/wiki/Epoch_(reference_date)) knows about January 1, 1970 right?

The problem isn't so much in that it publishes dates from a known time, rather that it does it in the least accessible
way! The value is rendered out (in JSON) as `"\/Date(1311836400000)\/"` - yeah..I know, right?

So now I had to figure out a nice way to parse this very custom date format in Objective-C. This led me to the shocking
(_and borderline unacceptable_) realization that objective-c (lower case to drive the point home) _**has no support for
regular expressions!!!**_. There's no punch-line to this joke...for real. No support for regex. A Unix-based OS has a
development language that has no support for something that is natively supported (see the * below)!

To start, I decided to add a category to the `NSDate` object that would allow me to work with these ridiculous strings.

{{< gist pseudomuto 6335411 >}}

The code above allows me to get an `NSDate` from the crazy OData DateTime string and also convert an `NSDate` to another
(quite different) format used by OData to query based on a DateTime value.

To get an `NSDate` from the OData string, I am substring'ing the value between the two brackets. This value is then
converted to a long long value (twice on purpose for you .NET developers out there). The `NSDate` has a static method
called `dateWithTimeIntervalSince1970:` that will return an (autoreleased) NSDate object with the value of the time (in
seconds) since January 1, 1970.

To get the date to the query format used by OData (see [OData URI
Conventions](http://www.odata.org/developers/protocols/uri-conventions)), we need to take an `NSDate` and get it into
the format `yyyy-MM-dd'T'HH:mm:ss` for example, `2011-07-29T18:15:00`.

I hope this information helps someone, as I was unable to find a simple example online anywhere (although I will concede
that my googling skills could use some work).

+ *Please don't take this as a complete knock to Objective-C. The language is quite easy to work with and definitely
    robust...just lame on the regex front!*
