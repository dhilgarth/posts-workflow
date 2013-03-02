Contains the default layout and accompanying rake file for a two stage blogging workflow using markdown files.  

#Usage

##Step 1: Create a draft
Execute `rake new_draft[<title>]`.  
This will generate a new, empty file in the folder `Drafts`. The format is `yyyy-mm-dd-<title>.md` and uses the current date.  
The empty file will automatically be committed.

This empty file is your draft. Write your blog post in it.

##Step 2: Publish the draft
Execute `rake publish_draft[<draft_specification>]` or `rake publish_draft[<draft_specification>, <title>]` if you don't want to use the title from the draft.  
This will create a file in the folder `Publish`. The format again is  `yyyy-mm-dd-<title>.md`.  
It again uses the current date, i.e. it doesn't use the date from the draft.  
If no `<title>` has been specified, the title of the draft will be used.  
The file will contain the complete content from the draft and before that a default [YAML front matter](https://github.com/mojombo/jekyll/wiki/yaml-front-matter) similar to the one [Octopress](http://octopress.org) generates.  

Please note that this script is agnostic to the blog engine you are using, as long as this engine knows how to interpret markdown files and the YAML front matter.  
That means that files in `Publish` will not auto-magically appear on your blog :) You will have to configure your blogging engine to pick these files up.

##Optional: View available drafts
If you want to know which drafts are available, you can use `rake list_drafts` or `rake list_drafts[<draft_specification>]`.

#Configuration

The rakefile contains three constants you can change:

- `drafts_dir      = "Drafts"`: If you want to give your drafts folder a different name. Changing the value here doesn't rename the folder in git or on the disk. You will have to do that manually.
- `publish_dir     = "Publish"`: If you want to give a different name to the folder that contains the posts to be published. Again, no automatic renaming.
- `new_post_ext    = "md"`: The file extension for drafts and finished posts. Is used when creating a new draft, when evaluating a draft specification and when publishing a draft.

## What is a draft specification?

This can be any string uniquely identifying a draft. This comparison is not case sensitive.  
Assume the Drafts folder contains these files:

    2013-03-02-Title1.md
    2013-03-02-Title2.md
    2013-03-02-Foo.md
    2013-03-02-Bar.md
    2013-03-02-Foo.markdown
    2013-03-02-Fubar.md
    2013-03-01-Fubar.md

Now, among others, the following specifications would be valid: 

- `title1`
- `foo`: There is only one file with the correct extension (`.md`) whose filename contains `foo`.
- `1-fubar`: Would match `2013-03-01-Fubar.md`.
- `-b`: Would match `2013-03-02-Bar.md`. Not really descriptive, but if you are lazy... :)
