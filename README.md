This repository contains the content of [scratchapixel.com](https://www.scratchapixel.com) in Markdown format.

What is Scratchapixel? It is a website that attempts to be a central repository of various computer graphics techniques. It gives readers a place where most vital information is stored and organized in a single location.

- Free. 
- Written in plain English. 
- Every equation is derived.
- Focus on simplicity & accessibility.
- Books like PBRT (Physically Based Rendering) provide a complete software solution, and chapters from the book describe different parts of the program. Scratchapixel does not provide the source code of a large software solution to which the lessons would refer. Instead, each lesson focuses on one particular technique, and each lesson comes with its code. The goal is to show how the studied technique translates into code in a program that can easily be compiled from the command line and doesn't rely on a complex build system to be tested.

What Scratchapixel is not:

- Not a place where you can learn about using 3D software such as Maya or Blender.
- Doesn't teach you about 3D real-time APIs such as Direct X or Vulkan. However, it will help you understand how these APIs work under the hood.

## Licence and Copyright

All content of Scratchapixel falls under the Creative Commons “BY-NC-ND” license (Attribution-NonCommercial-NoDerivatives license). It allows people to use the unadapted work for noncommercial purposes only as long as they give credit to the creator. They may also adapt the work for personal use but may not publicly share any adaptations.

## Editing Content

### Why and Who Can Edit the Content, and What For?

Anybody can edit the content. So you are more than welcome to do so. However, please check the guidelines below.

### Workflow for Contributors

If you want to help, first, thank you). We want to avoid conflicts in the process of reviewing lessons. For this reason, as we are experimenting with this review process, we thought that to start with, we would only have one reviewer per lesson. To do so, we made a Google Docs document listing all the review tasks' lessons. If you see a lesson that has yet to be a reviewer/contributor and that you would be willing to review this lesson, add your name next to the lesson title and follow the steps below.

[https://docs.google.com/spreadsheets/d/19ljS9fyrRFchaIn_TF8L2YaZw5p89kFqo6QfTad9Av0/edit?usp=sharing](https://docs.google.com/spreadsheets/d/19ljS9fyrRFchaIn_TF8L2YaZw5p89kFqo6QfTad9Av0/edit?usp=sharing)

- You need a GitHub account. Once logged in to your account, go to [Scratchapixel's website repository]().
- Fork the repository. This will create a copy of the repository in your account (**ps: you can also edit a file directly from within GitHub, but you will still need a GitHub account**. This will clone the file into your repos, and you will need to a make a pull request from there).

**Please be sure you regularly pull & merge changes from the Scratchapixel repository into your cloned repo before making edits. This is because other people are making changes to the project, and you need your repo to be up to date when you start your improvements.**. Here is a link that will tell you how to do that: [Pull new updates from original GitHub repository into forked GitHub repository](https://stackoverflow.com/questions/3903817/pull-new-updates-from-original-github-repository-into-forked-github-repository).

- How you want to edit the content after that point is entirely up to you. You can edit the content directly from GitHub (from your account, using the "forked" repo) or clone the project on your local computer to use the text editor of your choice.
- If you work with a cloned copy (remotely), you will need to install [Git Credential Manager](http://microsoft.github.io/Git-Credential-Manager-for-Windows/Docs/CredentialManager.html). An alternative solution to using Git Credential Manager is to use SSH keys as explained in these two documents: [Generating a new SSH key and adding it to the ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) and [Adding a new SSH key to your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account?tool=webui). The most useful commands you might need if you work with a remote copy is likely to be `git add`, `git commit`, and `git push`. Note that at this point, your changes will only be saved into your fork.
- When you are ready to push your edits to the main website, create a pull request (from which we can merge your changes into the main branch). Your edits will be approved before being merged into the main branch. Ideally, review a lesson thoroughly, and when you are done with your work, generate a pull request (rather than making a pull request for each incremental change).

It would be great if you could mainly focus on typos, layout, grammatical issues, and improving sentences' readability if needed. However, if you find problems with the code snippets or the equations, please feel free to fix them. 

The point of making the project open source at this point is to improve the quality of the current content as much as possible. Therefore, if you wish to write original content or suggest important changes to the structure of a lesson, please get in touch with us (via email/Discord).

Changes will be pushed to www.scratchapixel.org until all the content has been translated to Markdown and reviewed. Once done, we will then publish the content under www.scratchapixel.com. Hopefully, this can be done by the end of 2022, and we can start adding new content again.

### General Structure of a document

- H1 (#) is reserved for the lesson's title, so don't use it on a page.
- H2 (##) used for the paragraph headings of a chapter.

###  Editing rules

- Do not change the directory structure nor the content of metadata files such as `info.txt`, which you will find in the lessons repository. Thank you.
- Don't change the lesson's structure. If you want to make that kind of change, please contact us first. Thank you.
- When you write the heading of a paragraph (##), the first letter of every **key** word is capitalized. For instance, write "This is Lesson on Ray-Tracing" and not "This is a lesson on ray-tracing"
- Use standard markdown tags for quotes, italic, bold, images, etc. (see below).
- No point at the end of headings (h2).
- No word capitalization after a semicolon. For example, write, `Figure 1: this is an example` and not `Figure 1: This is an example`, unless, of course, the word is a noun.
- Avoid double-nested lists (they don't look good on mobile).
- The word Figure (if it refers to a figure) should be written `Figure` (with a capital F).
- All texts in figures should end with a dot (this is checked by the MD to HTML app).

### Markdown Rules

We are using some of the standard rules but added a few specific to Scratchapixel because we want to have something a little more sophisticated (and sometimes more control, too) than what's possible with the basic rules (like adding notes, marking a paragraph as important, etc.).

**Standard Markdown rules**:
- **Italic**: \_italic_
- **Bold**: \*\*bold**
- **Image**: \!\[Coment about the image](/relative/path/to/image.png)
- **Link**: \[Link](/relative/path/to/content). Regarding links, if you link to a lesson on scratcahpixel, the link should start with `/lessons`. This is the path relative to the website's root. If you want to point to a given lesson, you don't need to include the name of the first page of that lesson. For example, you can use `/lessons/3d-basic-rendering/introduction-to-ray-tracing/` instead of `/lessons/3d-basic-rendering/introduction-to-ray-tracing/how-does-it-work`. If you want to point to a chapter different that is not the first chapter, you will, of course, need to include it in the path. Like so: `/lessons/3d-basic-rendering/introduction-to-ray-tracing/raytracing-algorithm-in-a-nutshell` (without the HTML extension; note that there's no `/` at the end of the path).
  Note: you don't need to include the `.html` extension. It's done automatically through the rewrite rules. 
  Also, all paths will be checked during the conversion of the MD to HTML pages. Bad paths will be identified then and fixed at this point.
- **Heading 2**:  \## Some text (space after \##).
- **Heading 1**:  \# Some text (space after \#). Generally, do not use h1's. They are reserved for lessons' titles. The lesson's content should only contain h2s.
- **List**: start a paragraph with a - (**numbered list not supported now**). Remember to put a space after the sign -. Add a closing point at the end of each item.
- **Quote**: \> my quote text
- **Table**: see below
- **Inline-code**: \'example of inline code\' (surround the text with a `  - back single quote).
- **Code**. Create a new line starting with \```,  add your code, then close the block with a new line starting with \```. You can add the word `wrap` just after the first three ! (opening tag). That will force the code within the block to wrap (it will overflow in x otherwise, if the lines are longer than the column width).

**Rules that are specific to Scratchapixel**:
- **Note**: use \<details>\</details>. This is the only HTML tag you should use in a text. It can be used when you want to add a note that is a detail or not as relevant as the core of the content.
  ```
  <details>
  <summary>This is the title of this note</summary>
  Put your content here
  </details>
  ```
  Please respect the formatting (\<details>\</details> tags need to be on their line). You can use the \<summary>\</summary> to give the note a title.
- **Important**: use !!!. This tag can be used when you want to emphasize some content. You need to start a line with three ! And close the block with another three !.
  ```
  !!!!
  This is a critical section.
  You can use multiple lines and lists if you need to.
  !!!
  ```
- **Tables**: tables are not handled the way they are typically in Markdown. While Markdown is "designed" to make the text sort of readable in ASCII, the accepted rule for building a table in MD is just making the life of the editor a real pain when the cells contain multiple lines, which is more often the case than not when it comes to real-world work.
  ```
  |-table{My header title 1, My header title 2}
  |-row
  |-cell
  Some text in this cell
  - the nice thing is that it accepts lists.
  - this is another list item.
  |-cell
  Some text in this other cell
  |-row
  |-cell
  I like cells.
  |-cell
  |-
  ```
  The syntax is self-explanatory. As you can see, it feels similar to how you lay the table in HTML. Be mindful not to create more cells than they are cells in the header row declaration. For the header cell declaration, put the text of the cells better {} and separate them with a comma (leaving no space between the text and the commas and no space between the text and the {}).
  
- **Colored text**: to change the color of a sequence of words, use `@@\rthis is the text I want in red@@` where `\r` stands for red (and `\g` and `\b` for green and blue respectively). There should be no space between the `\r` (or `\g` or `\b`) qualifier and the first character in the colored string.

### Style

For style recommendations, check the Philosophy Behind Writing Content for Scratchapixel section below.

## General File Organization

The lessons' content is stored under `website/lessons`. Directories under this particular directory represent broad categories. Then in each general category, you will find the lessons organized in separate folders. Example: `lessons/3d-basic-rendering/introduction-to-ray-tracing/` is the path to the introduction-to-ray-tracing lesson. Inside that folder, you will find the chapters stored as Markdown files. The order of the lesson is defined in the `info.txt` file stored in the same directory. This is also where the lesson title is defined. Generally, please do not edit the name of the Markdown files, and do not change the content of an `info.txt` file (note that these will be replaced by JSON files soon). Only change the content of an MD file if you want to.

## Pushing Changes

Merges are blocked on the master branch. To make changes, follow the instructions provided above.

## Support for More Languages

XX TODO

## Philosophy Behind Writing Content for Scratchapixel

XX TO DO

We can always do more, but there's a limited number of daily hours. Let's iterate instead. One step at a time. Some first version of a lesson is better than no lesson at all. And then, over time, let's improve things, add more visuals, etc. **together**).

The written style is "casual". So noting "it's okay" vs. "it is okay" is fine. Both versions are acceptable!

## Roadmap

[Trello board](https://trello.com/b/a3aJhki7/scratchapixel)
