---
layout: post
title: A JSON Parser
---

[1]: http://underscorejs.org/ "Underscore.js"

I recently built a JSON parser in JavaScript from scratch. I learned a lot regarding data and recursion and although there are many different (and perhaps better) ways to go about parsing JSON, here's a little walkthrough of my implementation. First, a little background...

## JSON ##

**JSON** (JavaScript Object Notation) is a data-interchange format commonly used over the web to transmit data.

What JSON is is basically a set of rules on how to format text to represent JS objects and arrays so that complex data can be transmitted using strings of characters. Let's say I have an array in JS `[9, 'cat']`. It has two elements, `9` (a number) and `'cat'` (a string). The same array in JSON format would be text looking like this **[9,&ldquo;cat"]**.

<pre><code class="javascript">var a = [9, 'cat'];
var json = JSON.stringify(a); // json = "[9,"cat"]"
</code></pre>

Even though JSON has the word _JavaScript_ in it, it is largely language-independent, as many programming languages have built in functions that can convert between JSON and useable data types ( For example, Objective-C has one called `NSJSONSerialization`).

## Grammar ##

JSON's **grammar** defines the correct format for JSON text. These are the rules we need to follow to properly parse the JSON text (or throw an error if the JSON text does not follow the grammar). If the text does not follow the correct grammar, then it is _unparseable_.

JSON grammar can be (partially) defined like this:

    <JSON>    ::= <value>
    <value>   ::= <object> | <array> | <boolean> | <string> | <number> | <null>
    <array>   ::= "[" [ <value> {"," <value>}* ] "]"
    <object>  ::= "{" [ {<kv>} {"," <kv>}* ] "}"
    <kv>      ::= <string> ":" <value>

This is **BNF** (Backus-Naur Form) notation to describe context-free grammars. I define the grammar for `<JSON>` to be a `<value>`. And on the next line, I define `<value>` to be any of the 6 data types - object, array, boolean, string, number, or null.

On the third line, I define an `<array>` as text that begins with an opening bracket, followed by `<value>`\'s separated by commas, ending with a closing bracket.

On the fourth line, I define an `<object>` in a similar way to arrays, except each element is a `<kv>`, standing for _key-value pair_. Each `<kv>`, defined on the last line, is a `<string>` and `<value>` separated by a colon.

Quick note on the notation:

- The square brackets _within_ quotation marks denotes that the JSON text should actually have the square bracket characters. The square brackets _without_ quotation marks denote that the text within it is _optional_.
- Similar to square brackets, the curly braces _within_ quotation marks denotes the curly brace characters. The curly braces _without_ quotation marks, followed by asterisks, denotes that the enclosed text _repeats 0 or more times_.

Note the use of **recursion** in the grammar definition. Each element of an `<array>` (which is one possible branch of `<value>`) is another `<value>`, which can itself be an `<array>` or `<object>` or anything defined by the `<value>` grammar rules.

The grammar for booleans, strings, numbers, and null can also be further defined using BNF notation, but these are primitive values, so I'd rather just describe them in words below. For this purpose, we can think of these primitive data types as the _terminals_ (that's where the recursions end).

## Recursive Descent Parser ##

A **Recursive Descent Parser** is a parser that uses one of the simplest parsing methods to construct a parse tree according to a grammar. A parse tree is the result of applying the grammar to the text stream. In our example, given the JSON text **[9,&ldquo;cat"]**, the parse tree would like like this:

    <JSON> --- <value> --- <array> --- "["
                                   \__ <value> --- <number>
                                   \__ ","
                                   \__ <value> --- <string>
                                   \__ "]"

Creating this parse tree involves testing the grammar of all possible `<values>` each time a `<value>` is reached until one of them fits. Note that the `<number>` and `<string>` are the terminals of the parse tree. From this parse tree, we can create a JS array by pushing the number and string into a newly created empty array, thereby parsing the JSON text into a JavaScript array.

<a class="bookmark" name="solution" id="solution"></a>
First, here are some useful variables to start our parser:

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

Based on our grammar definition, `<JSON>` is just a `<value>`, which can be one of six possible data types. Luckily, each data type can be identified by the first character of the `<value>` text (this is not always possible depending on the grammar). Here is a function that takes `ch`, the character at the current index, and uses a `switch` statement to determine which function to call to parse the text.

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

To make each function consistent, each function will be called with `ch` starting at the first character of that `value` (opening square bracket for an array, for example). It should traverse along the `json` until it reaches the closing character based on JSON grammar (closing square bracket for arrays). It should then parse the JSON text between the starting and closing character, returning the corresponding JS data (object, array, etc.), and then set `at` to the next index (i.e. the first character of the next `<value>`).

### Null ###

The function `nully` starts at the current character `ch` and checks if it and the next three characters correspond to **&ldquo;n&rdquo;**, **&ldquo;u&rdquo;**, **&ldquo;l&rdquo;**, and **&ldquo;l&rdquo;**. If so, it returns `null`, otherwise, the `<value>` does not follow proper JSON syntax since there is no JSON value that starts with **n** other than JSON null (strings start with quotation marks). The `_.times` function is from [Underscore.js] [1] and simply runs the function _n_ times.

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

`bool` is set up similar to `nully` to check for characters that match the word _true_ and _false_.

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
2. One or more consecutive digits 0-9. If first digit is 0, then must be followed by decimal.
3. Optional decimal point
4. One or more consecutive digits 0-9 after decimal point
5. "e" or "E" to signify exponents
6. Positive or negative sign for the exponent
7. Consecutive digits 0-9 (no decimal points)

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

If you declare a string in JS `var a = "backslash\\"`, `a` will hold the string `backslash\` since `\` is used as an escape. But the JSON string of the variable `a` will be **"backslash\\\\&rdquo;**. Similarly for the `\b`, `\n`, `\t`, `\f`, and `\"` escaped characters. So for strings, we need to pay attention to the escape character **&ldquo;\\&rdquo;**, so that if we encounter `backslash\\` in a JSON string, we know to parse it to a JS string by declaring it `var a = "backslash\\"` instead of `"backslash\\\\"`.

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

With the primitives out of the way. We only have the collections (arrays and objects) remaining. The `array` function actually leverages the `value` function to populate all its elements. If the end of `json` is reached without finding a closing square bracket, then it's a bad JSON.

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

For objects, it is essentially the same as arrays, except it must also parse the key to each property. The `key` is just a JSON string though, so simply calling `string` will suffice.

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

Now that all the pieces of `value` have been implemented, we just need to wrap all the code starting from <a href="#solution" id="go-to-solution">here</a> into a function `parseJSON` and add the following code to execute our parser.

    function parseJSON(json) {

      /*
      other functions go here
      */

      at = 0;
      ch = json.charAt(at);
      return value();
    }

<!-- markup clean* -->

## Other Considerations ##

### More on Strings ###

There is actually one more escape character for JSON strings that I did not take into account in my parser - the `\u` followed by four hexadecimal digits. This escape is used to form characters in the Basic Multilingual Plane. The idea is similar to the other escapes. Instead of adding the character **&ldquo;\\&rdquo;**, **&ldquo;u&rdquo;**, and the 4 hexadecimal digits into the string, you want to add whatever character the escape code represents. Something I should implement in the future when I get the chance.

### Room for Improvement ###

Like I mentioned above, there are other methods out there for parsing. What I implemented here is a **Recursive Descent Parser**, which is a top-down approach - parsing the highest level collection and then subsequently parsing the nested elements. This type of parser works relatively well for JSON because each data type can be determined by simply looking at the first character of the `<value>`. Imagine a grammar where we cannot determine the data type until we have examined the first _n_ characters of the `<value>`. This would require some backtracking of the variables `at` and `ch`. The performance of a recursive descent parser for such a grammar might be comparably poorer.
