# Markdown Style Guidelines

## General

* Section titles should follow the [Chicago Title Capitalization](https://en.wikipedia.org/wiki/Title_case#Chicago_Manual_of_Style) standard.
* Lists should use the `*` character rather than the `-` character.
* Documents should start with a level one heading and should ideally be the same as the file name.
* For long files that are not served within this repository, it is best to have a table of contents at the end of the introduction of the level one heading section.
* Keep extra newlines between paragraphs and sections to make the source easier to read.
* Always specify the language for code blocks so that neither the syntax highlighter nor the text editor must guess.
  If no specific type makes sense, just use `text`.
* Use informative link titles. For example, instead of naming your links "link" or "here", wrap part of the sentence that is meant to be linked as a title.
* Do not use gendered pronouns when talking about users/consumers/whatever but always `they/their` instead.
* Do not use the future tense but use present simple for expressing general truths instead.
* Use active voice when there is no specific need to use passive.
* Abbreviations and acronyms should be spelled out the first time they appear in any technical document with the shortened form appearing in parentheses immediately after the term.
  The abbreviation or acronym can then be used throughout the document.
* Avoid ambiguous and abstract language (i.e. really, quite, very), imprecise or subjective terms (i.e. fast, slow, tall, small) and words that have no precise meaning (i.e. bit, thing, stuff).
* Avoid contractions (i.e. don't, you'll, etc.) as they are meant for informal contexts.
* Avoid generalized statements, because they are difficult to substantiate and too broad to be supported.
* Paragraphs that include multiple sentences should have the sentences on separate lines, so that updating one sentence results in a clear diff where one line changes.

## Specific to this Repository

In this repository, Markdown is extended by some mkdocs plugins which allow you to include external files within a markdown files, and to insert admonitions.

[Admonitions](https://squidfunk.github.io/mkdocs-material/reference/admonitions/#usage) are a way to include side content into a page without significantly interrupting the document flow.
There are different types of admonitions, and they allow for the inclusion and nesting of arbitrary content.

Including external files can be done by writing the name of a file present in the `include` directory surrounded by `{`/`}` and exclamation marks on an empty line.

## External Resources

* [Technical Writing Standards](https://engineering.usu.edu/students/ewc/writing-resources/technical-writing-standards)
* [Google Markdown Style Guide](https://google.github.io/styleguide/docguide/style.html)
* [Mastering GitHub Markdown](https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)
