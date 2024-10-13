# Numbers Are Numbers, Not Strings
A common practice when serializing to JSON is to encode floating point numbers as strings. This is done any time high precision is required, such as in the financial or scientific sectors. This approach is designed to overcome a flaw in many JSON parsers across multiple platforms, and, in my opinion, it's an anti-pattern.

## Numbers in JSON
The JSON specification (the latest being RFC 8259 as of this writing) does not place limits on the size or precision of numbers encoded into the format. Nor does it distinguish between integers or floating point.

That means that if you were to encode the first million digits of π as a JSON number, that precision would be preserved.

Similarly, if you were to encode 85070591730234615847396907784232501249, the square of the 64-bit integer limit, it would also be preserved.

They are preserved because JSON, by its nature as a text format, encodes numeric values as decimal strings. The trouble starts when you try to get those numbers out via parsing.

## The problem with parsers
Mostly, parsers are pretty good, except when it comes to numbers.

An informal, ad-hoc survey conducted by the engineers at a former employer of mine found that the vast majority of parsers in various languages automatically parse numbers into their corresponding double-precision (IEEE754) floating point representation. If the user of that parsed data wants the value in a more precise data type (e.g. a decimal or bigint), that floating point value is converted into the requested type afterward.

But at that point, all of the precision stored in the JSON has already been lost!

In order to properly get these types out of the JSON, they must be parsed directly from the text.

## My sad attempt at repeating the survey
### Pearl
Perl will at least give you the JSON text for the number if it can't parse the number into a common numeric type. This lets the consumer handle those cases. It also appears to have some built-in support for bignum.

A JSON number becomes either an integer, numeric (floating point) or string scalar in perl, depending on its range and any fractional parts.

### Javascript
Javascript actually recommends the anti-pattern for high-precision needs.

… numbers in JSON text will have already been converted to JavaScript numbers, and may lose precision in the process. To transfer large numbers without loss of precision, serialize them as strings, and revive them to BigInts, or other appropriate arbitrary precision formats.

### Go
Go parses a bigint number as floating point and truncates high-precision decimals. There’s even an alternative parser that behaves the same way.

### Ruby
Ruby only supports integers and floating point numbers.

### PHP
PHP (search for “Example #5 json_decode() of large integers”) appears to operate similarly to Perl in that it can give output as a string for the consumer to deal with.

### .Net
.Net actually stores the tokenized value (_parsedData) and then parses it upon request. So when you ask for a decimal (via .GetDecimal()) it actually parses that data type from the source text and gives you what you want. 10pts for .Net!

It appears that many languages support dynamically returning an appropriate data type based on what's in the JSON text (integer vs floating point), which is neat, but then they only go half-way: they only support basic integer and floating point types without any support for high-precision values.

## Developers invent a workaround
As is always the case, the developers who use these parsers need to have a solution, and they don't want to have to build their own parser to get the functionality they need. So what do they do? They create a convention where numbers are serialized as JSON strings any time high precision is required. This way the parser gives them a string, and they can parse that back into a number of the appropriate type however they want.

However, this has led to a multitude of support requests and StackOverflow questions.
* How do I configure the serializer to read string-encoded numbers?
* How do I validate string-encoded numbers?
* When is it appropriate or unnecessary to serialize numbers as strings?

And, as we saw with the Javascript documentation, this practice is actually being recommended now!

This is wrong! Serializing numbers as strings is a workaround that came about because parsers don't do something they should.

## Where to go from here
Root-cause analysis gives us the answer: the parsers need to be fixed. They should support extracting any numeric type we want from JSON numbers and at any precision.

A tool should make a job easier. However, in this case, we're trying to drive a screw with a pair of pliers. It works, but it's not what was intended.


## Source
https://blog.json-everything.net/posts/numbers-are-numbers-not-strings/
