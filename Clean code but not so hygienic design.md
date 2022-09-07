# Clean code but not so hygienic design?
_Originally posted 27/08/2018_

I really like the book [Clean Code: A Handbook of Agile Software Craftsmanship](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882/ref=sr_1_1_sspa) by Robert Martin (aka Uncle Bob) and have read it several times   I think it’s a valiant effort to describe the facets of clean code and a great guide for developers aiming to become better at their craft.  Having said that, there’s one part of the book that really annoys me – it gets my hackles up every time I read it.  The part I’m referring to is the Chapter 14 Successive Refinement “Case Study of a Command-Line Arguments Parser” (the actual code can be found at https://github.com/unclebob/javaargs).  It isn’t the actual code that bothers me – it’s the overall solution design that I think isn’t so great…

## Magical String
To utilise this code, the developer has to define a ‘schema’ – which is a string containing a comma separated list of definitions for the command line argument names and types.  For example:

```java
public class ArgsMain {
    public static void main(String[] args) {
        try {
            Args arg = new Args("l,p#,d*", args);
            boolean logging = arg.getBoolean('l');
            int port = arg.getInt('p');
            String directory = arg.getString('d');
            executeApplication(logging, port, directory);
        } catch (ArgsException e) {
            System.out.printf("Argument error: %s\n", e.errorMessage());
        }
    }

    private static void executeApplication(boolean logging, int port, String directory) {
        System.out.printf("logging is %s, port:%d, directory:%s\n",logging, port, directory);
    }
}
```
The string `"l,p#,d*"` passed into the constructor defines three command line arguments (named **l**, **p** and **d**) with types **boolean**, **integer** and **string** respectively.  How do I know these expected argument types by looking at the code?  Well, I can’t… I have to look at the documentation…
```
Schema:
- char    - Boolean arg.
- char*   - String arg.
- char#   - Integer arg.
- char##  - double arg.
- char[*] - one element of a string array.

Example schema: (f,s*,n#,a##,p[*])
Corresponding command line: "-f -s Bob -n 1 -a 3.2 -p e1 -p e2 -p e3
```
So that string is actually driving the behaviour of my code – but I have to decode that string to understand how.  Surely a better design starting point would have been to have the argument names and types defined as objects, for example something like…
```java
public class ArgsMain { 
    private static final ARG_NAME_LOGGING = "l";
    private static final ARG_NAME_PORT = "p";
    private static final ARG_NAME_DIRECTORY = "d";

    public static void main(String[] args) {
        try {
            ArgsParser argsParser = new ArgsParser(
                new BooleanArg(ARG_NAME_LOGGING),
                new IntegerArg(ARG_NAME_PORT),
                new StringArg(ARG_NAME_DIRECTORY)
            );

            argsParser.parse(args);

            boolean logging = argsParser.getBoolean(ARG_NAME_LOGGING);
            int port = argsParser.getInt(ARG_NAME_PORT);
            String directory = argsParser.getString(ARG_NAME_DIRECTORY);
            executeApplication(logging, port, directory);
        } catch (ArgsException e) {
            System.out.printf("Argument error: %s\n", e.errorMessage());
        }
    }

    private static void executeApplication(boolean logging, int port, String directory) {
        System.out.printf("logging is %s, port:%d, directory:%s\n",logging, port, directory);
    }
}
```

This would save having to parse the schema string and leave it with the single responsibility of parsing the actual command line arguments.

## Single character argument names?
There are several places In the book where it mentions that single character variable names are generally a bad practice – yet the design of this command line arguments parser inflicts single character argument names on the poor user.

## Hammock Driven Development
A year or so ago, I was reading the book again on a beach in Fuerteventura (yes, I get very bored on beaches!) and decided to jot down some ideas for a command line arguments parser of my own.  Upon arriving home, I converted those ideas into working code which can be found at https://github.com/Adeptions/CLArguments.

