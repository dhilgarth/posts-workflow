Contains the default layout and accompanying rake file for a two stage blogging workflow using markdown files.  

#Usage

##Step 1: Create a draft
Execute `rake new_draft[<title>,<commit>]`.  
Examples:

    rake new_draft["My new post",false]
    rake new_draft["My new post"]

This will generate a new file in the folder `Drafts`. The format is `yyyy-mm-dd-<title>.md` and uses the current date.  
It will contain a default [YAML front matter](https://github.com/mojombo/jekyll/wiki/yaml-front-matter) similar to the one [Octopress](http://octopress.org) generates, but it will have `published` set to `false` and `type` set to `draft`. The first one is honored by Octopress, the second one by Ruhoh and will result in the post not showing up in the Blog.  
The file will automatically be committed, if `commit` has been omitted or specified as `true`. The script will ensure that you are on the branch `blog` when it commits. This is the branch that should receive all commits related to your posts.

This new file is your draft. It is automatically opened in your favorite editor, so you can immediately start writing your article. :)

##Step 2: Publish the draft
When you have finished writing your blog post, execute `rake publish_draft[<draft_specification>,<commit>,<title>]`  
Examples:

    rake publish_draft["new post"]
    rake publish_draft["new post",false]
    rake publish_draft["new post",true,"New title"]

This will create a file in the folder `Publish`. The format again is  `yyyy-mm-dd-<title>.md`.  
It again uses the current date, i.e. it doesn't use the date from the draft.  
If no `<title>` has been specified, the title of the draft will be used:

- If the draft contains a `title` property in its YAML front matter, it is used.
- If that property doesn't exist, the title is extracted from the file name of the draft.

The file will contain the content from the draft along with an adjusted YAML front matter:

- The property `title` will be set to the new title determined above.
- The property `date` will be set to the current date
- The property `layout` will be set to `"post"`
- The property `published` will be set to `true`
- The property `type` will be removed, if it is `"draft"`
- If the draft is missing the `comments` property, it will be added with a value of `true`; otherwise, the value from the draft will be used.
- All other properties from the draft will be used without changes.

The published post will automatically be committed, if `commit` has been omitted or specified as `true`. Again, it ensures you are on the `blog` branch.

**Please note:**  
This script is mostly agnostic to the blog engine you are using, as long as this engine knows how to interpret markdown files and the YAML front matter.  
That means that files in `Publish` will not auto-magically appear on your blog :) You will have to configure your blog engine to pick these files up.

##Optional: View available drafts
If you want to know which drafts are available, you can use `rake list_drafts` or `rake list_drafts[<draft_specification>]`.

##Optional: Open a draft in the editor
This is a little shortcut, so you don't have to switch to your file commander: `rake edit_draft[<draft_specification>]`.  
This will open the draft in the editor defined in the rakefile.

#Configuration

The rakefile contains four constants you can change:

- `drafts_dir      = "Drafts"`: If you want to give your drafts folder a different name. Changing the value here doesn't rename the folder in git or on the disk. You will have to do that manually.
- `publish_dir     = "Publish"`: If you want to give a different name to the folder that contains the posts to be published. Again, no automatic renaming.
- `new_post_ext    = "md"`: The file extension for drafts and finished posts. Is used when creating a new draft, when evaluating a draft specification and when publishing a draft.
- `editor          = "/path/to/editor"`: The editor used to open a new draft.

##What is a draft specification?

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

##Shorthand version of tasks
Because I am lazy, I added four shorthand tasks that just forward the parameters to the actual tasks:
- `nd` -> `new_draft`
- `pd` -> `publish_draft`
- `ld` -> `list_drafts`
- `ed` -> `edit_draft`