require "rubygems"

## -- Rsync Deploy config -- ##
# Be sure your public key is listed in your server's ~/.ssh/authorized_keys file
ssh_user       = "nfsn"
ssh_port       = "22"
document_root  = "/home/public"
rsync_delete   = false
rsync_args     = ""  # Any extra arguments to pass to rsync
deploy_default = "rsync"


## -- Misc Configs -- ##

public_dir      = "public"    # compiled site directory
source_dir      = "content"    # source file directory
static_dir      = "static"
posts_dir       = "post"    # directory for blog files
new_post_ext    = "markdown"  # default new post file extension when using the new_post task

#######################
# Working with Hugo #
#######################

desc "Generate Hugo site"
task :build do
  system "compass compile --css-dir #{static_dir}/stylesheets/"
  system "hugo"
  system "./scripts/crush.pl"
end

desc "Watch the site and regenerate when it changes"
task :serve do
  puts "Starting to watch source with Hugo and Compass."
  hugoPid = Process.spawn("hugo server")
  compassPid = Process.spawn("compass watch --css-dir #{static_dir}/stylesheets/")

  trap("INT") {
    [hugoPid, compassPid].each { |pid| Process.kill(9, pid) rescue Errno::ESRCH }
    exit 0
  }

  [hugoPid, compassPid].each { |pid| Process.wait(pid) }
end



# usage rake new_post[my-new-post] or rake new_post['my new post'] or rake new_post (defaults to "new-post")
desc "Begin a new post in #{source_dir}/#{posts_dir}"
task :new_post, :title do |t, args|
  if args.title
    title = args.title
  else
    title = get_stdin("Enter a title for your post: ")
  end
  raise "### You haven't set anything up yet. First run `rake install` to set up an Octopress theme." unless File.directory?(source_dir)
  mkdir_p "#{source_dir}/#{posts_dir}"
  filename = "#{source_dir}/#{posts_dir}/#{Time.now.strftime('%Y-%m-%d')}-#{to_url(title)}.#{new_post_ext}"
  if File.exist?(filename)
    abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end
  puts "Creating new post: #{filename}"
  open(filename, 'w') do |post|
    post.puts "---"
    post.puts "title: \"#{title.gsub(/&/,'&amp;')}\""
    post.puts "date: #{Time.now.strftime('%Y-%m-%dT%H:%M:%SZ')}"
    post.puts "draft: true"
    post.puts "url: /#{to_url(title)}/"
    post.puts "---"
  end
end

##############
# Deploying  #
##############
#


def run(cmd)
  puts cmd
  system(cmd)
end
desc "Default deploy task"
task :deploy do
  Rake::Task["build"].execute
  system "chmod 664 static/images/*"
  system "chmod 777 static/images/drawings"
  system "chmod 777 static/images/rust-talk"
  system "chmod 777 static/images/stl-talk"
  Rake::Task["#{deploy_default}"].execute
  puts "Clearing Cloudflare cache"
  system "bash scripts/cloudflare_clear_cache.sh"
end

desc "Deploy website via rsync"
task :rsync do
  exclude = ""
  if File.exists?('./rsync-exclude')
    exclude = "--exclude-from '#{File.expand_path('./rsync-exclude')}'"
  end
  puts "## Deploying website via Rsync"
  ok_failed run("rsync --size-only -avze 'ssh -p #{ssh_port}' #{exclude} #{rsync_args} #{"--delete" unless rsync_delete == false} #{public_dir}/ #{ssh_user}:#{document_root}")
end

def ok_failed(condition)
  if (condition)
    puts "OK"
  else
    puts "FAILED"
  end
end

def to_url(title)
  title.downcase.gsub(/[^\w]/, '-')
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

desc "list tasks"
task :list do
  puts "Tasks: #{(Rake::Task.tasks - [Rake::Task[:list]]).join(', ')}"
  puts "(type rake -T for more detail)\n\n"
end
