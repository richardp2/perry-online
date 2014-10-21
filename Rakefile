require "rubygems"
require "tmpdir"

require "bundler/setup"
require "jekyll"

# Change your GitHub reponame
GITHUB_REPONAME = "richardp2/perry-online"
BITBUCKET_REPO = "richardp2/perry-online"


desc "Build and preview the site"
task :preview => [:grunt, :clean] do
  puts "## Building a preview of the site"
  pids = [
    spawn("jekyll serve -w --drafts")
  ]
  
  trap "INT" do
    Process.kill "INT", *pids
    puts "\n## Preview site shutdown"
    exit 1
  end
  
  loop do
    sleep 1
  end
end

desc 'Runs grunt'
task :grunt do
  puts "## Concatenating & minifying/uglifying css & js files"
  system "grunt"
end
  
desc 'Delete generated _site files'
task :clean do
  puts "## Cleaning up build folder (if it exists)"
  system "rm -rf _site"
end
  
  
desc "Commit the source branch of the site"
task :commit do
  puts "## Adding unstaged files"
  system "git add -A > /dev/null"
  puts "\n## Committing changes with commit message from file 'changes'"
  system "git commit -aF changes"
end
  
desc "Push source file commits up to origin"
task :push do
  puts "## Check there is nothing to pull from origin"
  system "git pull"
  puts "## Pushing commits to origin"
  system "git push origin source"
end
  

desc "Build the site ready for deployment"
task :build => [:grunt, :clean] do
  puts "## Generate the Jekyll site files"
  Jekyll::Site.new(Jekyll.configuration({
    "source"      => ".",
    "destination" => "_site"
  })).process
  puts "## Build complete"
end


desc "Generate and deploy blog to master"
task :deploy, [:message] => [:commit, :push, :build] do |t, args|
  args.with_defaults(:message => "Site updated at #{Time.now.utc}")
  
  puts "## Push built site to master branch"  
  Dir.mktmpdir do |tmp|
    # Clone the master branch into a temporary directory"
    system "git clone git@github.com:#{GITHUB_REPONAME}.git -b gh-pages #{tmp}"
    
    # Delete all files in the temporary directory to ensure deleted file are removed"
    rm_rf "#{tmp}/*"
    
    # Copy the build site to the temporary directory
    cp_r "_site/.", tmp
    
    # Store the current working directory for latere
    pwd = Dir.pwd
    
    # Change to the temporary directory
    Dir.chdir tmp
    
    # Add unstaged files, commit them, add the additional repository at Bitbucket and push to origin
    system "git add -A"
    system "git commit -m #{args[:message].inspect}"
    system "git remote set-url --add origin git@bitbucket.org:#{BITBUCKET_REPO}.git"
    system "git push origin gh-pages"
    
    # Change back to the previous working directory
    Dir.chdir pwd
  end
  
  puts "\nSite Published and Deployed to GitHub"
  puts "\nHave a nice day :-)"
end