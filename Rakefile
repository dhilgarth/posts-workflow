require "rubygems"
require "fileutils"
require "stringex"
require "git"
require "logger"
require "preamble"

## -- Configs -- ##

drafts_dir      = "Drafts"
publish_dir     = "Publish"
new_post_ext    = "md"  # default new post file extension when using the new_post task

# usage rake new_draft[my-new-post] or rake new_draft['my new post']
desc "Begin a new draft in #{drafts_dir}"
task :new_draft, :title, :commit do |t, args|
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
    open(filename, 'w') do |post|
    post.puts "---"
    post.puts "layout: post"
    post.puts "title: \"#{title.gsub(/&/,'&amp;')}\""
    post.puts "date: #{Time.now.strftime('%Y-%m-%d %H:%M')}"
    post.puts "comments: true"
    post.puts "published: false"
    post.puts "type: draft"
    post.puts "categories: "
    post.puts "---"
  end

  if args.commit
    commit = to_boolean(args.commit)
  else
    commit = true
  end

  if commit
    log = Logger.new(STDOUT)
    log.level = Logger::INFO
    git = Git.open(".", :log => log)
    git.branch("blog").checkout
    git.add(filename)
    git.commit("Create empty draft #{filename}")
  end
end

# usage rake publish_draft[draft_specification]
# draft_specification can be any string that completely or partly and uniquly identifies the draft.
desc "Publishes the specified draft to #{publish_dir}"
task :publish_draft, :draft, :commit, :title do |t, args|
  abort("No draft specified") if args.draft.nil?
  draft_filename = args.draft
  draft_filename = GetDraftFilename(drafts_dir, new_post_ext, draft_filename)
  draft_file = "#{drafts_dir}/#{draft_filename}"
  abort("Specified draft not found") unless File.exist?(draft_file)
  draft_content = ""
  front_matter = []
  begin
    data = Preamble.load(draft_file)
    abort("The file you want to publish is empty. Publishing cancelled") if !data[0]
    abort("The file you want to publish doesn't contain any content. Publishing cancelled") if data[1].empty?
    draft_content = data[1]
    front_matter = data[0]
    title = front_matter["title"]
    title.strip! unless title.nil?
    front_matter.delete("title")
    front_matter.delete("date")
    front_matter.delete("published")
    type = front_matter["type"]
    front_matter.delete("type") if type and front_matter["type"] == "draft"
  rescue
    abort("rake aborted!") if ask("The YAML front matter seems to be invalid: #{$!.message}\nDo you want to treat the complete file as content?", ['y', 'n']) == 'n'
    draft_content = File.read(draft_file)
  end

  if args.title
    title = args.title
    title.strip!
  else
    title = %r/\d{4}-\d{2}-\d{2}-(.*?)\.#{new_post_ext}/.match(draft_filename)[1] if title.empty?
  end
  
  mkdir_p "#{drafts_dir}"
  filename = "#{publish_dir}/#{Time.now.strftime('%Y-%m-%d')}-#{title.to_url}.#{new_post_ext}"
  if File.exist?(filename)
    abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end
  
  AddDefaultFrontMatterProperties(front_matter, title)
  
  puts "Publishing draft '#{draft_file}' to '#{filename}'"
  open(filename, 'w') do |post|
    post.puts CreateFrontMatterRepresentation(front_matter)
    post.puts draft_content
  end
  
  if args.commit
    commit = to_boolean(args.commit)
  else
    commit = true
  end
  
  if commit
    log = Logger.new(STDOUT)
    log.level = Logger::INFO
    git = Git.open(".", :log => log)
    git.branch("blog").checkout
    git.add(filename)
    begin
      git.remove(draft_file)
    rescue
      puts "!! Could not delete local draft file '#{draft_file}'. You will have to do this manually !!"
      system "git rm -f --cached #{draft_file}"
    end
    git.commit("Publish #{filename}")
  else
    begin
      File.delete(draft_file)
    rescue
      puts "!! Could not delete local draft file '#{draft_file}'. You will have to do this manually !!"
    end
  end
end

# usage rake list_drafts[] or rake list_drafts[draft_specification] to only view the drafts matching the specification
desc "Shows a list of all drafts. If a draft specification has been supplied only those that match it are shown."
task :list_drafts, :draft_specification do |t, args|
  draft_specification = args.draft_specification
  if draft_specification.nil?
    drafts = GetAllDrafts(drafts_dir, new_post_ext)
    if drafts.count() == 0
      puts "No drafts available!"
    else
      puts "All available drafts:\n" + drafts.join("\n")
    end
  else  
    drafts = FindMatchingDrafts(drafts_dir, new_post_ext, draft_specification)
    if drafts.count() == 0
      puts "No drafts available for specification '#{draft_specification}'!"
    else
      puts "Available drafts with specification '#{draft_specification}':\n" + drafts.join("\n")
    end
  end
end

def GetAllDrafts(drafts_dir, new_post_ext)
    drafts = Array.new
    Dir.foreach(drafts_dir) { |f| drafts.push(f) if File.file?("#{drafts_dir}/#{f}") and File.extname("#{drafts_dir}/#{f}") == ".#{new_post_ext}" }
    return drafts
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
  return draft_filename
end

def FindMatchingDrafts(drafts_dir, new_post_ext, draft_specification)
  draft_specification = draft_specification.downcase
  matchingFiles = Array.new
  GetAllDrafts(drafts_dir, new_post_ext).each do |f| 
    matchingFiles.push(f) if f.include?(draft_specification)
  end
  return matchingFiles
end

def get_stdin(message)
  print message
  STDIN.gets.chomp
end

def ask(message, valid_options)
  if valid_options
    answer = get_stdin("#{message} #{valid_options.to_s.gsub(/"/, '').gsub(/, /,'/')} ") while !valid_options.include?(answer)
    return answer
  else
    return get_stdin(message)
  end
end

def AddDefaultFrontMatterProperties(front_matter, title)
  front_matter["layout"] = "post"
  front_matter["title"] = "\"#{title.gsub(/&/,'&amp;')}\""
  front_matter["date"] = "#{Time.now.strftime('%Y-%m-%d %H:%M')}"
  front_matter["comments"] = true if front_matter["comments"].nil?
  front_matter["published"] = true
end

def CreateFrontMatterRepresentation(front_matter)
    result = "---\n"
    front_matter.each do |key, value|
      result << "#{key}: #{value}\n"
    end
    result << "---\n"
    
    return result
end

def to_boolean(s)
  s and !!s.match(/^(true|t|yes|y|1)$/i)
end
