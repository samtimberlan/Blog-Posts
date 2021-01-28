Regular expressions are powerful patterns written to check the adherence of input to expected requirements. For example, regular expressions provide a way to validate email addresses to ensure they meet the expected requirement of a name, an `@` sign and a domain.
Though powerful, regular expressions (regex), can cause performance bottlenecks and security breaches through denial of service (DOS), if not used correctly. What are some important points to consider when using regex in .Net?

1.	Input source: An input source can be reliable or unreliable. They usually contain data that:
 - Matches the pattern
 - Nearly matches the pattern
 - Does not match the pattern
Unreliable sources will contain more input that nearly matches or does not match a given pattern. Input that nearly matches the regex pattern, typically require more processing time. Depending on the input length, this time could be days, or even years.
2.	Always use a timeout: The regex engine in .Net uses an infinite timeout by default when matching. This is meant to be changed according to the developer’s need and the resulting timeout exception handled accordingly. Using a regex timeout secures an app from denial of service.
3.	Object instantiation: Use static pattern-matching method, such as `Regex.Match(String, String)` instead of instantiating the regex class `new Regex(pattern)` for methods that will be frequently called, as this, will probably use a cached, compiled version
4.	Assembly Compilation: When developing a regular expression library, you can significantly improve the startup time and execution time by compiling to a library. 
5.	 Testing: When testing, do not restrict input to only ones that match. Test input that nearly matches and observe the performance, especially in scenarios where the regex will handle unreliable input

Additional reading:
[Microsoft doc: Best practices for regular expressions in .NET](https://docs.microsoft.com/en-us/dotnet/standard/base-types/best-practices)

[Microsoft Doc: Backtracking in Regular Expressions](https://docs.microsoft.com/en-us/dotnet/standard/base-types/backtracking-in-regular-expressions)

There you have it. Thanks for reading.

Did you spot a typo, an error or want to contribute? [Here's the repo on GitHub](https://github.com/samtimberlan/Blog-Posts/blob/drafts/)