# The Rakefile does all of the real work.

task :default => [:github]

desc "Run your resume locally."
task :local do
   exec('./resume.rb')
end

# TODO: Make this dynamically figure out all of the files needed from Sinatra.
desc "Deploy to remote defined in config.yaml"
task :github do
   require File.expand_path('../resume',__FILE__)
   require 'rubygems'

   # Nice Error checking for gems.
   [ 'git', 'rack/test', 'logger' ].each {|gem|
      begin
         require gem
      rescue LoadError
         puts "The gem #{gem} is not installed.\n"
         exit
      end
   }

   remote = YAML.load_file('config.yaml')['github']['remote']

   browser = Rack::Test::Session.new(Rack::MockSession.new(Sinatra::Application))

   files = [
      'index.html',
      'resume.txt',
   ]

   files = files + Dir.entries("public").keep_if {|file| File.file? "public/#{file}"}

   root = "/tmp/checkout-#{Time.now.to_i}"
   g = Git.clone(remote, root, :log => Logger.new(STDOUT))

   # Make sure this actually switches branches.
   g.checkout(g.branch('gh-pages'))

   files.each {|file|
      browser.get file
      content = browser.last_response.body
      File.open("#{root}/#{file}", 'w') {|f| f.write(content) }
      g.add(File.basename(file))
   }

   g.commit('Regenerating Github Pages.')

   # PUSH!
   g.push(g.remote('origin'), g.branch('gh-pages'))

   puts '--> GitHub Pages Commit and Push successful.'
end
