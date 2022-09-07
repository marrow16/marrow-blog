# Unicode Unicorns
_Originally posted 26/08/2018_

Unicode has been around for 27 years, the first standard being published in 1991.  The intention of Unicode was to allow computer systems to exchange information containing characters from the many languages around the world.

With the rise of the World Wide Web and our desire to develop systems that can be used by anyone in the world you’d have thought that Unicode would have solved any multi-lingual problems our applications might encounter.  Unfortunately this isn’t quite the case – our tools and languages can deal with Unicode but there are still some developer myths that catch us out.

_WARNING: the rest of this blog post may contain nuts and Java code._

## Unicorn Myth #1 _“The 8 in UTF-8 means each character is 8-bits”_
No.  Unicode is a codepoint system rather than a character system.  Each codepoint in UTF-8 may well be 8-bits (aka a byte!) but each codepoint does not necessarily represent one character.

(Besides, if the ‘8’ meant 8-bits per character then we’d still be limited to 256 different characters – whereas Unicode supports over 1 million characters).

Take the Euro symbol for example, whose Unicode codepoint is U+20AC.  Create a string in Java containing that character and see how many bytes you get…

```java
package com.adeptions.code;

import java.io.UnsupportedEncodingException;

public class Application {
  public static void main(String[] args) throws UnsupportedEncodingException {
    String euroSymbol = "\u20ac";
    String price = euroSymbol + "10";
    System.out.println(price.getBytes("UTF-8").length);
  }
}
```
You will see 5 being output – which means that the Euro symbol was encoded with 3 bytes.

For more info on why – see https://en.wikipedia.org/wiki/UTF-8#Examples

## Unicorn Myth #2 _“The 16 in UTF-16 means each character is 16-bits”_
No, and same reason as #1 is incorrect.  The 16 means each codepoint is 16-bits.

Try the following code…

```java
package com.adeptions.code;

import java.io.UnsupportedEncodingException;

public class Application {
  public static void main(String[] args) throws UnsupportedEncodingException {
    String euroSymbol = "\u20ac";
    String price = euroSymbol + "10";
    System.out.println(price.getBytes("UTF-16").length);
  }
}
```
The output is 8 – which means the Euro symbol was 4 bytes!


## Unicorn Myth #3 _“UTF-16, UTF-16, there’s only one UTF-16!”_
Unfortunately not, let me introduce you to ‘endians’ (no, we’re not talking about Cowboys and Indians here!).  See https://en.wikipedia.org/wiki/Endianness

Java supports UTF-16 plus UTF-16BE (Big-endian) and UTF-16LE (Little-endian) – try the following to see the difference…

```java
package com.adeptions.code;

import java.io.UnsupportedEncodingException;

public class Application {
  public static void main(String[] args) throws UnsupportedEncodingException {
    String euroSymbol = "\u20ac";
    String price = euroSymbol + "10";

    System.out.println("Length of UTF-16 is: " + price.getBytes("UTF-16").length);
    System.out.println("Length of UTF-16BE is: " + price.getBytes("UTF-16BE").length);
    System.out.println("Length of UTF-16LE is: " + price.getBytes("UTF-16LE").length);
 
    System.out.println("First byte of UTF-16 is: " + (price.getBytes("UTF-16")[0] & 0xff));
    System.out.println("First byte of UTF-16BE is: " + (price.getBytes("UTF-16BE")[0] & 0xff));
    System.out.println("First byte of UTF-16LE is: " + (price.getBytes("UTF-16LE")[0] & 0xff));
  }
}
```
You should get an output of…
```
Length of UTF-16 is: 8
Length of UTF-16BE is: 6
Length of UTF-16LE is: 6
First byte of UTF-16 is: 254
First byte of UTF-16BE is: 32
First byte of UTF-16LE is: 172
```
which might not be what you’d expect!


## Unicorn Myth #4 _“In Java, I can easily iterate over the characters in a String”_
Only if you are absolutely, 100% sure that all the characters in the string are in the BMP (Basic Multilingual Pane) of Unicode (codepoints 0x0000 to 0xFFFF).  This is because Java uses UTF-16 to represent strings internally – which makes it easy to index into characters of the string most of the time.

The Euro symbol, U+20AC, is conveniently in the BMP – therefore the following code works…

```java
package com.adeptions.code;

public class Application {
  public static void main(String[] args) {
    String euroSymbol = "\u20ac";
    String price = euroSymbol + "10";

    for (int i = 0; i < price.length(); i++) {
      System.out.println("Character at [" + i + "] is... " + price.charAt(i));
    }
  }
}
```
And you should get…
```
Character at [0] is... €
Character at [1] is... 1
Character at [2] is... 0
```
However, things start to go wrong when the characters don’t sit in the BMP.  Take some music for example…
```java
package com.adeptions.code;

public class Application {
  public static void main(String[] args) {
    char[] musicalSymbolFClef = Character.toChars(0x1D122);
    String someMusic = new String(musicalSymbolFClef);

    for (int i = 0; i < someMusic.length(); i++) {
      System.out.println("Character at [" + i + "] is... " + someMusic.charAt(i));
    }
  }
}
```
The output looks very wrong…
```
Character at [0] is... ?
Character at [1] is... ?
```
There was just one Unicode codepoint – let’s check that…
```java
package com.adeptions.code;

import java.io.UnsupportedEncodingException;

public class Application {
  public static void main(String[] args) throws UnsupportedEncodingException {
    char[] musicalSymbolFClef = Character.toChars(0x1D122);
    String someMusic = new String(musicalSymbolFClef);

    System.out.println("Actual number of Unicode codepoints is : " + (someMusic.getBytes("UTF-32").length / 4));
    System.out.println("But Java thinks there are " + someMusic.length() + " characters");
  }
}
```
which should output…
```
Actual number of Unicode codepoints is : 1
But Java thinks there are 2 characters
```
Yes, definitely just one codepoint but Java is convinced there are two characters.  So how can we reliably dissect that string? Try this…
```java
package com.adeptions.code;

public class Application {
  public static void main(String[] args) {
    char[] musicalSymbolFClef = Character.toChars(0x1D122);
    String someMusic = new String(musicalSymbolFClef);

    for (int i = 0; i < someMusic.length();) {
      System.out.println("Codepoint at [" + i + "] is... 0x" + Integer.toHexString(someMusic.codePointAt(i)));
      System.out.println("Character at [" + i + "] is... " + new String(Character.toChars(someMusic.codePointAt(i))));
      i += Character.charCount(someMusic.codePointAt(i));
    }
  }
}
```
Which should correctly output…
```
Codepoint at [0] is... 0x1d122
Character at [0] is... 𝄢
```

## Unicorn Myth #5 _“Unicode is always input in exactly the same way”_
If only that were true – but it’s not.  When Unicode was created, it was realised that even with a potential of 1,114,112 code points there would still be occasions where, in some languages, people might add accents to characters that hadn’t been done before.  Welcome to Unicode normalization! (see https://en.wikipedia.org/wiki/Unicode_equivalence).

The French call themselves “Française” – the ‘c’ has a strange little hook underneath it… it’s called a cedilla.

Unicode has a codepoint for a ‘c’ with a cedilla (U+00E7).  Unicode also has a codepoint for just a ‘c’ (U+0063) aswell as a codepoint for a combining cedilla (U+0327).  So whoever input the word “Française” may have used the ‘c’ with cedilla or they may have used a ‘c’ followed by a combining cedilla.  But those two strings won’t be the same…
```java
package com.adeptions.code;

public class Application {
  public static void main(String[] args) {
    String french1 = "Fran\u00E7aise";
    String french2 = "Fran\u0063\u0327aise";

    System.out.println(french1);
    System.out.println(french2);
 
    System.out.println("Are they equal? " + french1.equals(french2));
  }
}
```
Which will output…
```
Française
Française
Are they equal? false
```
They looked the same in the output but they weren’t equal!  This is why Java offers you the chance to normalize the Unicode – to eliminate the different ways that a string may have been input….
```java
package com.adeptions.code;

import java.text.Normalizer;

public class Application {
  public static void main(String[] args) {
    String french1 = "Fran\u00E7aise";
    String french2 = "Fran\u0063\u0327aise";

    String french1Normalized = Normalizer.normalize(french1, Normalizer.Form.NFD);
    String french2Normalized = Normalizer.normalize(french2, Normalizer.Form.NFD);
 
    System.out.println("Are they equal? " + french1Normalized.equals(french2Normalized));
  }
}
```
That’s better!


## Unicorn Myth #6 _“When people search for stuff, they always input words in the same Unicode form”_
See also #5 – no they don’t!

So when persisting text in a database that you later intend to search – try normalising it into something consistent and then normalizing the search string.


## Unicorn Myth #7 _“Consumers of my API will always POST/PUT with a request body encoded in UTF-8”_
Good luck with that!

