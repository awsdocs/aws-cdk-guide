# Guidelines for contributing to the AWS CDK Developer Guide

Thank you for your interest in contributing to AWS documentation! We greatly value feedback and contributions from our community.

**Table of Contents**

- [Introduction and welcome](#introduction-and-welcome)
- [Contribution workflow](#contribution-workflow)
- [Getting started](#getting-started)
- [Pull Request guidelines](#pull-request-guidelines)
- [What to expect when you contribute](#what-to-expect-when-you-contribute)
- [Finding contributions to work on](#finding-contributions-to-work-on)
- [Code of conduct](#code-of-conduct)
- [Security issue notifications](#security-issue-notifications)
- [Licensing](#licensing)
- [AsciiDoc syntax reference](#asciidoc-syntax-reference)

## Introduction and welcome

The AWS CDK Developer Guide is an important resource for AWS CDK users. Your contributions help improve the documentation, making it more accurate, comprehensive, and useful for everyone. Whether you're fixing a typo, clarifying explanations, or adding new examples, your contributions are welcome and appreciated.

This document outlines the process for contributing to the AWS CDK Developer Guide to help ensure a smooth experience for both contributors and maintainers.

## Contribution workflow

The contribution process follows these general steps:

### Step 1: Find something to work on

1. **Find something from the CDK v2 Developer Guide website**
   - On any page in the [AWS CDK v2 Developer Guide](https://docs.aws.amazon.com/cdk/v2/guide/home.html), you can select the **Edit this page on GitHub** link to open the source file in GitHub.

2. **Find an Issue to work on**
   - Find an open [Issue](https://github.com/awsdocs/aws-cdk-guide/issues) that you'd like to contribute to.

### Step 2: Work on your contribution

1. **Create an Issue (optional)**
   - For substantial changes, check if an issue already exists or create a new one.
   - For typos, minor clarifications, or small improvements, you can skip directly to creating a pull request.

2. **Fork and clone the repository**
   - Work on your changes in your forked repository.

3. **Make your changes**
   - Author locally or on GitHub.
   - Follow the existing document style and formatting.
   - Test your changes using GitHub's preview.

4. **Submit a pull request**
   - Link to any related issues in your PR description.
   - Fill out the PR template with necessary information.

### When to create an issue

We recommend creating or finding an issue first for:

- New content you'd like to contribute (such as new code samples or tutorials).
- Information gaps that need substantial new content.
- Significant reorganization of existing content.
- Any change that might benefit from discussion before implementation.

For simple changes like typos, grammatical errors, or minor clarifications, you can directly submit a pull request without an issue.

## Getting started

There are two main ways to contribute: editing directly in GitHub (easier for small changes) or forking and cloning the repository (better for substantial changes).

### Option 1: Editing directly in GitHub

For simple changes like fixing typos or small improvements:

1. Navigate to the file you want to edit in the GitHub repository.
2. Click the pencil icon to edit the file.
3. If this is your first contribution, GitHub will automatically prompt you to fork the repository. Click **Fork this repository** to continue.
4. Make your changes in the editor. Refer to the [AsciiDoc format](#asciidoc-syntax-reference) for your AsciiDoc syntax guidance.
5. Preview your changes using GitHub's **Preview** tab to ensure they look correct.
6. Select **Commit changes** to commit your changes to your fork.
7. Provide a **commit message** and clear description of your changes.
8. Select **Propose changes** and verify that your changes look correct in the diff preview.
9. Select **Create pull request**.
10. Fill out the PR and select **Create pull request**.

### Option 2: Editing locally using your preferred IDE

For more substantial changes or when you need to make multiple edits:

1. [Fork the repository](https://help.github.com/articles/fork-a-repo/) by clicking the **Fork** button in the upper right corner of the GitHub interface.

2. Clone your fork to your local machine:

   ```bash
   git clone https://github.com/YOUR-USERNAME/aws-cdk-guide.git
   cd aws-cdk-guide
   ```

3. Create a new branch for your changes:

   ```bash
   git checkout -b your-branch-name
   ```

4. Make your changes using your preferred IDE.

5. Commit your changes with a descriptive message:

   ```bash
   git add .
   git commit -m "Description of your changes"
   ```

6. Push your changes to your fork:

   ```bash
   git push origin your-branch-name
   ```

7. When your changes are ready, [create a pull request](https://help.github.com/articles/creating-a-pull-request-from-a-fork/) from your fork to the main repository.

### Making changes

When making changes to the documentation:

1. Make your changes using your preferred IDE (if you cloned locally) or directly in GitHub's editor.
2. Follow the existing document structure and formatting.
3. Use the [AsciiDoc format](#asciidoc-syntax-reference) for your documentation.
4. Preview your changes using GitHub's **Preview** tab or your IDE's preview feature to ensure proper formatting.

## Pull Request guidelines

### What to include in your Pull Request

- **Clear description**: Explain what your changes accomplish and why they're necessary.
- **Related issues**: Reference any related issues (e.g., "Fixes #123" or "Addresses #456").
- **Scope of changes**: Mention which files were modified and what kind of changes were made.

### Pull Request review process

After submitting your PR:

1. Maintainers will review your changes.
2. You may receive feedback requesting changes.
3. Once approved, your changes will be merged.

We try to review pull requests within a few business days, but response times may vary depending on the current workload of the documentation team.

### Tips for a successful Pull Request

- Keep PRs focused on a single topic to make them easier to review.
- Follow the existing document structure and formatting.
- Test your changes using GitHub's preview functionality.
- Respond to feedback and make requested changes promptly.

## What to expect when you contribute

When you submit a pull request, our team is notified and will respond as quickly as we can. We'll do our best to work with you to ensure that your pull request adheres to our style and standards. If we merge your pull request, we might make additional edits later for style or clarity.

The AWS documentation source files on GitHub aren't published directly to the official documentation website. If we merge your pull request, we'll publish your changes to the documentation website as soon as we can, but they won't appear immediately or automatically.

## Finding contributions to work on

If you'd like to contribute, but don't have a project in mind, look at the [open issues](https://github.com/awsdocs/aws-cdk-guide/issues) in this repository for some ideas.

In addition to written content, we really appreciate new examples and code samples for our documentation, such as examples for different platforms or environments, and code samples in additional languages.

## Code of conduct

This project has adopted the [Amazon Open Source Code of Conduct](https://aws.github.io/code-of-conduct). For more information, see the [Code of Conduct FAQ](https://aws.github.io/code-of-conduct-faq) or contact [opensource-codeofconduct@amazon.com](mailto:opensource-codeofconduct@amazon.com) with any additional questions or comments.

## Security issue notifications

If you discover a potential security issue, please notify AWS Security via our [vulnerability reporting page](http://aws.amazon.com/security/vulnerability-reporting/). Please do **not** create a public issue on GitHub.

## Licensing

See the [LICENSE](https://github.com/awsdocs/aws-cdk-guide/blob/main/LICENSE) file for this project's licensing. We will ask you to confirm the licensing of your contribution. We may ask you to sign a [Contributor License Agreement (CLA)](http://en.wikipedia.org/wiki/Contributor_License_Agreement) for larger changes.

## AsciiDoc syntax reference

This section provides a reference for writing and formatting AsciiDoc content for the AWS CDK v2 Developer Guide. Following these conventions ensures consistency throughout the documentation and simplifies maintenance.

> **Tip**: You don't need to memorize all these formats. You can use existing documentation files as templates and refer to this guide when needed.

### Basic File Structure

AWS CDK Developer Guide AsciiDoc files follow a common structure:

```asciidoc
include::attributes.txt[]

// Attributes
[.topic]
[#page-id]
= Page Title
:info_titleabbrev: Short Title
:info_abstract: Abstract text that describes the page content in 1-2 sentences.
:keywords: comma-separated, list, of, keywords

[abstract]
--
Abstract paragraph that appears at the top of the page. This is typically 
identical to the info_abstract attribute above but formatted as a paragraph.
--

// Content start
Main content begins here...
```

Key components:

1. **Attributes include**: Every file starts by including common attributes from `attributes.txt`.
2. **Page properties**: Topic class, page ID anchor, and title.
3. **Metadata attributes**: Short title, abstract, and keywords for SEO.
4. **Abstract block**: Formatted version of the abstract that appears on the page.
5. **Main content**: Starts after a comment indicating content start.

### Headings

AsciiDoc uses `=` symbols for headings with more symbols indicating lower levels:

```asciidoc
= Level 1 (Page Title)
== Level 2 Section
=== Level 3 Section
==== Level 4 Section
```

In AWS CDK documentation, use the following pattern:

- `=` for page title (only one per document)
- `==` for major sections
- `===` for subsections
- `====` for sub-subsections (use sparingly)

**Special rule for nested pages**: For pages that have nested pages (like `chapter-deploy.adoc`), you can only use two levels of headers:

- `=` for the page title
- `==` for sections

For additional heading levels in nested pages, use variable lists instead:

```asciidoc
Topic Name::
+
This serves as a third-level heading and can contain paragraphs, code blocks, and other content.
```

Section IDs are crucial for linking and navigation. Add them using the following syntax:

```asciidoc
[#section-id]
== Section Title

[#section-id,section-id.title]
== Section Title with Custom Label
```

### Text Formatting

Use the following syntax for text formatting:

- **Bold**: `*bold text*`
- _Italic_: `_italic text_`
- `Monospace`: ```monospace text```
- **_Bold and italic_**: `*_bold and italic text_*`
- [.underline]#Underlined#: `[.underline]#underlined text#`
- Link text: `link:url[link text]` or `https://example.com[link text]`

### Lists and Term Definitions

AsciiDoc provides multiple ways to structure content using different list types. These are essential for organizing information in the AWS CDK documentation.

#### Standard Lists

##### Unordered Lists

Unordered lists use asterisks, with more asterisks for deeper nesting levels:

```asciidoc
* First-level item
* Another first-level item
** Second-level item
*** Third-level item
* Back to first-level
```

This renders as:

- First-level item
- Another first-level item
  - Second-level item
    - Third-level item
- Back to first-level

##### Ordered Lists

Ordered lists use periods, with more periods for deeper nesting levels:

```asciidoc
. First item
. Second item
.. Nested item A
.. Nested item B
. Third item
```

This renders as:

1. First item
2. Second item
   a. Nested item A
   b. Nested item B
3. Third item

##### Mixing List Types

You can mix list types:

```asciidoc
. First ordered item
* Unordered sub-item
* Another unordered sub-item
. Second ordered item
```

#### Variable Lists (Term Definitions)

Variable lists use a term followed by a double colon (`::`), which is particularly useful for describing terms or creating pseudo-headings, especially in nested pages:

```asciidoc
Term or Heading::
+
Content associated with the term. The plus sign after the double colon ensures 
that this content is treated as a new block, allowing paragraphs, code blocks, 
and other elements.

Another Term::
+
More content here.
```

In nested pages, variable lists serve as third-level headings:

```asciidoc
[#my-section]
== Section Heading

Topic Name::
+
This serves as a pseudo third-level heading and can contain full paragraphs, 
code examples, lists, and more.

Another Topic::
+
Another topic's content here.
```

#### Continuation and Complex Content in Lists

To include paragraphs, code blocks, or other elements within any list item, use the plus sign continuation:

```asciidoc
* List item
+
Additional paragraph within the list item.
+
[source,javascript]
----
console.log('Code block within a list item');
----
+
Another paragraph.

* Next list item
```

#### Grouping Lists to Fix Formatting Issues

If you encounter formatting problems with lists, especially when they appear within paragraphs, you can group list items using block delimiters (`--`):

```asciidoc
This is a sentence with a list:
+
--
* Bullet point 1
* Bullet point 2
--
+
This is the next sentence after the list.
```

This technique ensures proper rendering and prevents common formatting issues when lists are embedded within other content.

### Links and References

The CDK documentation uses several link types:

1. **External links:**

```asciidoc
https://aws.amazon.com[AWS]
```

1. **Internal document cross-references (xref):**

```asciidoc
xref:page-id[Optional Link Text]
xref:page-id#section-id[Optional Link Text]
```

1. **API references link example:**

```asciidoc
link:https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html[Bucket]
```

### Code Blocks and Examples

For syntax-highlighted code blocks, use:

```asciidoc
[source,language]
----
code here
----
```

Where `language` is the programming language (e.g., `javascript`, `python`, `bash`).

For console output or plain text:

```asciidoc
[source,bash,subs="verbatim,attributes"]
----
$ command example
output example
----
```

For language-specific tabbed examples (a common pattern in CDK docs):

```asciidoc
====
[role="tablist"]
TypeScript::
+
[source,javascript,subs="verbatim,attributes"]
----
// TypeScript code here
----

JavaScript::
+
[source,javascript,subs="verbatim,attributes"]
----
// JavaScript code here
----

Python::
+
[source,python,subs="verbatim,attributes"]
----
# Python code here
----

Java::
+
[source,java,subs="verbatim,attributes"]
----
// Java code here
----

C#::
+
[source,csharp,subs="verbatim,attributes"]
----
// C# code here
----

Go::
+
[source,go,subs="verbatim,attributes"]
----
// Go code here
----
====
```

### Notes, Tips, and Admonitions

Admonitions add highlighted notes, warnings, or tips:

```asciidoc
[NOTE]
====
Note content here. Use the block form (====) for multi-paragraph notes.
====

[TIP]
====
Tip content here.
====

[IMPORTANT]
====
Important information here.
====

[WARNING]
====
Warning information here.
====
```

For single-paragraph admonitions, you can use the inline form:

```asciidoc
[NOTE]
A simple, single-paragraph note.
```


### Variable Replacement and Attributes

AWS CDK documentation uses attribute references for common terms. This ensures consistency and simplifies updates. For example:

- `{aws}` expands to "AWS"

Use attributes for any term that:

- Appears frequently
- Might need to be updated across the entire doc set
- Has specific styling or formatting requirements

### Tables

Simple tables use this format:

```asciidoc
|===
|Header 1 |Header 2 |Header 3

|Row 1, Col 1
|Row 1, Col 2
|Row 1, Col 3

|Row 2, Col 1
|Row 2, Col 2
|Row 2, Col 3
|===
```

### Collapsible Sections

For large code examples or verbose output, use collapsible sections:

```asciidoc
[#example-id]
.Collapsible section title
[%collapsible]
====
Content that can be collapsed/expanded
====
```

### Common Patterns in CDK Documentation

1. **Service Names**: Always use the full service name on first mention, followed by the abbreviation in parentheses.
2. **Cross-Service Links**: When referencing another AWS service, provide a link to its documentation on first mention.
3. **Language Examples**: Always provide code examples for all supported languages (TypeScript, JavaScript, Python, Java, C#, and Go).
4. **Prerequisites**: List prerequisites at the beginning of tutorial-style content.
5. **Step Numbering**: For sequential steps, use the AsciiDoc ordered list or explicitly number sections.

### Best Practices

1. **File Organization**: Keep files focused on a single topic or concept.
2. **Language**: Use simple, clear language and active voice.
3. **SEO**: Include relevant keywords in page titles, section headings, and the `:keywords:` attribute.
4. **Links**: Cross-reference related topics to help users navigate the documentation.
5. **Images**: Include diagrams for complex concepts (store in the `/images/` directory).
6. **Examples**: Include practical, working examples whenever possible.

### Common Issues and Troubleshooting

1. **Missing Attributes**: If attributes aren't resolving (appearing as `{attribute-name}`), check that `attributes.txt` is properly included.
2. **Broken Cross-References**: Ensure the referenced page and section IDs exist.
3. **Build Errors**: The most common causes are:

    - Invalid AsciiDoc syntax
    - Missing closing tags for blocks
    - Invalid attributes or includes

4. **Preview Differences**: The local preview may differ slightly from the published version. Always verify important changes in the staging environment.
