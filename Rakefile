require "rubygems"
require "fileutils"
require "stringex"
require "git"
require "logger"

## -- Configs -- ##

drafts_dir      = "Drafts"
publish_dir     = "Publish"
new_post_ext    = "md"  # default new post file extension when using the new_post task

# usage rake new_draft[my-new-post] or rake new_draft['my new post']
desc "Begin a new draft in #{drafts_dir}"
task :new_draft, :title do |t, args|
  if args.title
    title = args.title
  else
    title = get_stdin("Enter a title for your post: ")
  end
  mkdir_p "#{drafts_dir}"
  filename = "#{drafts_dir}/#{Time.now.strftime('%Y-%m-%d')}-#{title.to_url}.#{new_post_ext}"
  if File.exist?(filename)
    abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end
  puts "Creating new draft: #{filename}"
  FileUtils.touch filename
  g = Git.open(".", :log => Logger.new(STDOUT))
  g.add(filename)
  g.commit("Create empty draft #{filename}")

end

# usage rake publish_draft[draft_specification]
# draft_specification can be any string that completely or partly and uniquly identifies the draft:
desc "Publishes the specified draft to #{publish_dir}"
task :publish_draft, :draft, :title do |t, args|
  abort("No draft specified") if args.draft.nil?
  draft_filename = args.draft
  draft_filename = GetDraftFilename(drafts_dir, new_post_ext, draft_filename)
  draft_file = "#{drafts_dir}/#{draft_filename}"
  abort("Specified draft not found") unless File.exist?(draft_file)
  if args.title
    title = args.title
  else
    title = %r/\d{4}-\d{2}-\d{2}-(.*?)\.#{new_post_ext}/.match(draft_filename)[1]
  end
  mkdir_p "#{drafts_dir}"
  filename = "#{publish_dir}/#{Time.now.strftime('%Y-%m-%d')}-#{title.to_url}.#{new_post_ext}"
  if File.exist?(filename)
    abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end
  puts "Publishing draft '#{draft_file}' to '#{filename}'"
  open(filename, 'w') do |post|
    post.puts "---"
    post.puts "layout: post"
    post.puts "title: \"#{title.gsub(/&/,'&amp;')}\""
    post.puts "date: #{Time.now.strftime('%Y-%m-%d %H:%M')}"
    post.puts "comments: true"
    post.puts "categories: "
    post.puts "---"
    post.puts File.read(draft_file)
  end
  File.delete(draft_file)
  g = Git.open(".", :log => Logger.new(STDOUT))
  g.add(filename)
  g.commit("Publish #{filename}")
end

# usage rake list_drafts[] or rake list_drafts[draft_specification] to only view the drafts matching the specification
desc "Shows a list of all drafts. If a draft specification has been supplied only those that match it are shown."
task :list_drafts, :draft_specification do |t, args|
  draft_specification = args.draft_specification
  if draft_specification.nil?
    drafts = GetAllDrafts(drafts_dir, new_post_ext).join("\n") 
    puts "All available drafts:\n#{drafts}"
  else  
    drafts = FindMatchingDrafts(drafts_dir, new_post_ext, draft_specification).join("\n")
    puts "Available drafts with specification '#{draft_specification}':\n#{drafts}"
  end
end

def GetAllDrafts(drafts_dir, new_post_ext)
    drafts = Array.new
    Dir.foreach(drafts_dir) { |f| drafts.push(f) if File.file?("#{drafts_dir}/#{f}") and File.extname("#{drafts_dir}/#{f}") == ".#{new_post_ext}" }
    answer = drafts;
end

def GetDraftFilename(drafts_dir, new_post_ext, draft_specification)
  draft_filename = draft_specification
  unless File.exist?("#{drafts_dir}/#{draft_filename}")
    possible_drafts = FindMatchingDrafts(drafts_dir, new_post_ext, draft_filename)
    if possible_drafts.count() > 1
      multiple_drafts = possible_drafts.join("\n")
      abort("Multiple drafts found:\n#{multiple_drafts}\n\nPlease specify the draft including the date.")
    end
    abort("No drafts found that match the specification '#{draft_specification}'\nUse 'rake list_drafts' to get a list of available drafts.") unless possible_drafts.count() == 1
    draft_filename = possible_drafts[0]
    abort("rake aborted") if ask("Found possible draft '#{draft_filename}'. Did you mean this one?", ['y', 'n']) == 'n'
  end
  answer = draft_filename
end

def FindMatchingDrafts(drafts_dir, new_post_ext, draft_specification)
  draft_specification = draft_specification.downcase
  matchingFiles = Array.new
  GetAllDrafts(drafts_dir, new_post_ext).each do |f| 
    matchingFiles.push(f) if f.include?(draft_specification)
  end
  answer = matchingFiles
end

def get_stdin(message)
  print message
  STDIN.gets.chomp
end

def ask(message, valid_options)
  if valid_options
    answer = get_stdin("#{message} #{valid_options.to_s.gsub(/"/, '').gsub(/, /,'/')} ") while !valid_options.include?(answer)
  else
    answer = get_stdin(message)
  end
  answer
end
