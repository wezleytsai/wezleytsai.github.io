---
layout: post
title: A JSON Parser
---

<!-- links -->
[underscorejs]: http://underscorejs.org/

[oreilly-json-parser]: http://archive.oreilly.com/pub/a/javascript/excerpts/javascript-good-parts/json.html

[progzoo-rdp-tutorial]: http://progzoo.net/wiki/Recursive_Descent_Parser_Tutorial

[mdn-json-parse]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse

[json.org]: http://json.org

<!-- post -->

I recently built a JSON parser in JavaScript from scratch. I learned a lot on data formats/parse trees/grammar/recursion that helped me break down and solve the problem. There are many different (and perhaps better) approaches to parsing JSON, but here's a little walkthrough of my implementation. First, a little background...

## JSON ##

**JSON** (JavaScript Object Notation) is a data-interchange format commonly used over the web to transmit data.

What JSON is is basically a set of rules on how to format text to represent JS objects and arrays so that complex data can be transmitted using strings of characters. An array in JS such as `[9, 'cat']` would be represented in JSON by this text **[9,&ldquo;cat"]**.

<pre><code class="javascript">var a = [9, 'cat'];
var json = JSON.stringify(a); // json = "[9,"cat"]"
</code></pre>

Even though JSON has the word _JavaScript_ in it, it is pretty much universal (a lot of APIs will provide data in JSON format). Almost all programming languages have built in functions that can convert between JSON and useable data types (For example, Objective-C has a method called `NSJSONSerialization` that can parse JSON into `NSArray` or `NSDictionary`).

## Grammar ##

JSON's **grammar** defines the correct format for JSON text. These are the rules we need to follow to properly parse the JSON text. If the text does not follow the correct grammar, then the parser should throw an error message to indicate that the JSON text is _unparseable_.

JSON grammar can be (partially) defined like this:

<pre><code class="nohighlight"><!--
--><JSON&gt;     ::= <value&gt;
<value&gt;    ::= <object&gt; | <array&gt; | <boolean&gt; | <string&gt; | <number&gt; | <null&gt;
<array&gt;    ::= "[" [<value&gt;] {"," <value&gt;}* "]"
<object&gt;   ::= "{" [<property&gt;] {"," <property&gt;}* "}"
<property&gt; ::= <string&gt; ":" <value&gt;
</code></pre>

This is **BNF** (Backus-Naur Form) notation to describe context-free grammars. I define the grammar for `<JSON>` to be a `<value>`. And on the next line, I define `<value>` to be any of the 6 data types - object, array, boolean, string, number, or null.

On the third line, I define an `<array>` as text that begins with an opening bracket, followed by `<value>`\'s separated by commas, ending with a closing bracket.

On the fourth line, I define an `<object>` in a similar way to arrays, except it contains a series of `<property>`'s separated by commas. A `<property>` is defined on the last line as a `<string>` (the property name) and `<value>` (property value), separated by a colon.

Quick note on the notation:

- The square brackets _within_ quotation marks indicate that the JSON text should actually have the square bracket characters. The square brackets _without_ quotation marks denote _optional_ text.
- Similarly, the curly braces _within_ quotation marks denote curly brace characters. The curly braces _without_ quotation marks, followed by asterisks, denote text that _repeats 0 or more times_.

Note the use of **recursion** in the grammar definition. Each element of an `<array>` (which is one possible branch of `<value>`) is another `<value>`, which can itself be an `<array>` or `<object>` or anything defined by the `<value>` grammar rules.

The grammar for booleans, strings, numbers, and null can also be further defined using BNF notation, but these are primitive values, so I'd rather just describe them in words below. For this purpose, we can think of these primitive data types as the _terminals_ (that's where the recursions end).

## Recursive Descent Parser ##

A parse tree, or expression tree, is the result of parsing the text into components that fit the grammar. In our example, given the JSON text **[9,&ldquo;cat"]**, the parse tree would look something like this:

<pre><code class="nohighlight"><!--
--><JSON&gt; --- <value&gt; --- <array&gt; --- "["
                               \__ <value&gt; --- <number&gt; --- 9
                               \__ ","
                               \__ <value&gt; --- <string&gt; --- "cat"
                               \__ "]"
</code></pre>

A **Recursive Descent Parser**, one of the simplest parsers, uses a top-down parsing approach, meaning we look at the highest level (the array in this case), and then work downwards (or rightwards in my graph) to parse its elements.

Creating this parse tree involves checking the grammar of all possible `<values>` each time a `<value>` to determine which one of them fits. Note that the `<number>` and `<string>` are the terminals of the parse tree. Once we have created the parse tree, we can create a JS array by pushing the number (9) and string ("cat") into a newly created empty array, thereby parsing the JSON text into a JavaScript array.

<a class="bookmark" name="solution" id="solution"></a>
The parser will work by looking at each character of the JSON text left to right. Here are some useful variables/functions to start our parser:

    var at, // current index of JSON text
        ch; // character at current index

    var next = function() {
      // increments at
      // updates ch
      at += 1;
      ch = json.charAt(at); // json is the JSON text passed into our parser
      return ch;
    };

    var error = function(message) { // throw error for bad syntax
      console.log(message);
      throw undefined;
    };

We'll use `next` to traverse our `json` text and `error` to throw errors for bad syntax.

### Parsing a Value ###

Based on our grammar definition, `<JSON>` is just a `<value>`, which can be one of six possible data types. Each data type can be identified by the first character of the `<value>` text, so no _backtracking_ is required (this is not always true depending on the grammar, but it is for JSON). Here is a function that takes `ch`, the character at the current index, and uses a `switch` statement to determine which function to call to parse the following text.

    var value = function () {
      switch(ch) {
        case '{':
          return object();
        case '[':
          return array();
        case '\"':
          return string();
        case 't':
        case 'f':
          return bool();
        case 'n':
          return nully();
        default:
          if(ch === '-' || (ch && ch >= 0 && ch <= 9)) { // number
            return number();
          } else {
            error('bad JSON');
          }
          break;
      }
    };

Now we just need to implement the functions corresponding to each data type in order to fully parse the JSON.

Recall how our grammar stated that objects and arrays also have `<value>`'s within their JSON syntax. This is the recursive aspect of our parser. Our parser will call `value` from within `object` and `array` based on that syntax. The recursion ends once a primitive value is reached (recall that they are the terminals of our parse tree).

To make each function consistent, each function will be called with `ch` starting at the first character of that `value` (**&ldquo;[&rdquo;** for an array, for example). It should traverse along the `json` until it reaches the closing character based on JSON grammar (**&ldquo;]&rdquo;** for arrays). It should then parse the JSON text between the starting and closing character, returning the corresponding JS data (an object, array, etc.), and then set `at` to the next index (i.e. the first character of the next `<value>`).

Here are the implementations of each possible data type:

### Null ###

The function `nully` starts at the current character `ch` and checks if it and the next three characters correspond to **&ldquo;n&rdquo;**, **&ldquo;u&rdquo;**, **&ldquo;l&rdquo;**, and **&ldquo;l&rdquo;**. If so, it returns `null`, otherwise, the `<value>` does not follow proper JSON syntax since there is no JSON value that starts with **n** other than JSON null (strings start with quotation marks, not letters). The `_.times` function is from [Underscore.js] [underscorejs] and simply runs the function _n_ times.

    var nully = function() {
      // ch is at 'n', verify and return null
      var nully = '';
      if(ch === 'n') {
        _.times(4, function() {
          nully += ch;
          next();
        });
        if(nully === 'null') {
          return null;
        } else {
          error('bad null');
        }
      }

      error('bad null');
    };

<!-- markup clean_ -->

### Booleans ###

`bool` is set up similar to `nully` to check for characters that match the words _true_ or _false_.

    var bool = function() {
      // ch is at 't' of 'f', verify & return the boolean
      var bool = '';
      if(ch === 't') {
        _.times(4, function() {
          bool += ch;
          next();
        });
        if(bool === 'true') {
          return true;
        } else {
          error('bad bool');
        }
      } else if(ch === 'f') {
        _.times(5, function() {
          bool += ch;
          next();
        });
        if(bool === 'false') {
          return false;
        } else {
          error('bad bool');
        }
      }

      error('bad bool');
    };

<!-- markup clean_ -->

### Numbers ###

The `number` function is a little more complicated because of the possible formats of a number. A JSON number consists of the following:

1. Optional negative sign
2. One or more consecutive digits 0-9 (0 can only be the first digit if followed by decimal point)
3. Optional decimal point
4. One or more consecutive digits 0-9 after decimal point
5. "e" or "E" to signify exponents
6. Positive or negative sign for the exponent
7. Consecutive digits 0-9 (only integers allowed here)

The `number` function constructs a string based on these components and then calls `Number` on it to convert it into a proper JS number. If the conversion fails, then the JSON number format was bad.

    var number = function() {
      // ch is at negative sign '-' or digit 0-9, create & return the number
      var number = ''; // create string and then use Number() to convert
      function getDigits() { // collect consecutive digits until non-digit is reached
        while(ch && ch >= 0 && ch <= 9) { // need to avoid empty strings
          number += ch;
          next();
        }
      }

      // optional - get neg sign
      if(ch === '-') {
        number += ch;
        next();
      }

      getDigits();

      // optional - get decimal point
      if(ch === '.') {
        number += ch;
        next();
        getDigits();
      }

      // optional - get exponential
      if(ch === 'e' || ch === 'E') {
        number += ch;
        next();
        // required - get sign of exponent
        if(ch === '-' || ch === '+') {
          number += ch;
          next();
        }
        getDigits(); // exponent
      }

      if(!isNaN(Number(number))) { // check if string can be converted to number
        return Number(number);
      } else { // string could not be converted to number
        error('bad number');
      }
    };

### Strings ###

If you declare a string in JS `var a = "backslash\\"`, `a` will hold the string `backslash\` since `\` is used as an escape. But the JSON string of the variable `a` is actually **"backslash\\\\&rdquo;**. Same goes for the escaped characters `\b`, `\n`, `\t`, `\f`, and `\"`. So for strings, we need to pay attention to the escape character **&ldquo;\\&rdquo;**, so that if we encounter `backslash\\` in a JSON string, we know it's a string with only one backslash character instead of two.

    var escapes = { // helper variable
      'b': '\b',
      'n': '\n',
      't': '\t',
      'r': '\r',
      'f': '\f',
      '\"': '\"',
      '\\': '\\'
    };

    var string = function() {
      // ch is at opening quote, create & return the string
      var string = '';
      if(ch !== '\"') error('string should start with \"');
      next();
      while(ch) {
        // watch for end of string
        if(ch === '\"') {
          next();
          return string;
        }

        // watch for escapes
        if(ch === '\\') {
          next();
          if(escapes.hasOwnProperty(ch)) {
            string += escapes[ch];
          } else {
            // if not a proper escape code, ignore escape and just add char
            // NOTE: this should never be called if proper stringified JSON provided
            string += ch;
          }
        } else {
          // anything other than \ and " => just add character to string
          string += ch;
        }
        next();
      }
      // reached end without closing quote => error
      error('bad string');
    };

### Arrays ###

With the primitives out of the way. We only have the collections (arrays and objects) remaining. The `array` function actually leverages the `value` function to populate all its elements, which are identified as the entire span of text between brackets and commas. If the end of `json` is reached without finding a closing square bracket, then it's a bad JSON.

    var array = function() {
      // ch is at opening bracket, create & return the array
      var array = [];
      if(ch !== '[') error('array should start with [');
      if(next() === ']') return array; // empty array

      do {
        array.push(value());
        if(ch === ']') { // array end reached
          next();
          return array;
        }
      } while(ch && ch === ',' && next()); // found ',' => more elements to go

      error('bad array');
    };

### Objects ###

`object` is implemented very similar to `array`, except it must also parse the key of each property. The `key` is just a JSON string though, so simply calling `string` will do.

    var object = function() {
      // ch is at opening curley brace, create & return the object
      var object = {};
      if(ch !== '{') error('object should start with {');
      if(next() === '}') return object; // empty object

      do {
        var key = string(); // get key
        if(ch !== ':') error('object property expecting ":"');
        next();
        object[key] = value(); // create property with whatever value is, perhaps another object/array
        if(ch === '}') {  // object end reached
          next();
          return object;
        }
      } while(ch && ch === ',' && next()); // found ',' => more properties to go

      error('bad object');
    };

### Execute ###

Now that all the pieces of `value` have been implemented, we just need to wrap all the code starting from <a href="#solution" id="go-to-solution">here</a> into a function - let's call it `parseJSON` - and add a couple of lines of code to execute the parser.

    function parseJSON(json) {

      /*
      insert code for:
      at, ch, next, error, value, nully, bool, number, escapes, string, array, object
      */

      at = 0;
      ch = json.charAt(at);
      return value();
    }

<!-- markup clean* -->

## Other Considerations ##

### More on Strings ###

There is actually one more escape for JSON strings that I did not take into account in my parser - the `\u` followed by four hexadecimal digits. This escape is used for special characters and symbols. The implementation would be similar to the other escapes. Instead of adding the characters **&ldquo;\\&rdquo;**, **&ldquo;u&rdquo;**, and the 4 hexadecimal digits into the string, you would instead add whatever character the escape code represents.

### Reviver ###

The documentation for `JSON.parse` on [MDN] [mdn-json-parse] shows that there's an optional second argument, `reviver`. The `reviver` function transforms the parsing result before returning it. Another feature that would be good practice to implement.

## Resources ##

- I found the Wikipedia entries for parse trees and BNF notation pretty informative.
- The railroad diagrams on [JSON.org] [json.org] are great for visualizing the JSON data syntax.
- For more on Recursive Descent Parsers, check out these links:
  - [Recursive Descent Parser Tutorial] [progzoo-rdp-tutorial]
  - [JSON Parser Example] [oreilly-json-parser]
