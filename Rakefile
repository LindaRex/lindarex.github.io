require "rubygems"
require "tmpdir"

require "bundler/setup"
require "jekyll"

require "listen"

GITHUB_REPO_NAME = "lindarex/lindarex.github.io"
PUBLISH_SOURCE_PATH = "../lindarex.github.io/"
DEFAULT_COMMIT_MESSAGE = "minor changes"

desc "==========  SERVE =========="
task :serve do
  puts "==========  SERVE =========="
  options = {
    "source"      => ".",
    "destination" => "_site",
    "incremental" => false,
    "profile"     => false,
    "watch"       => true,
    "serving"     => true,
  }
  Jekyll::Commands::Build.process(options)
  Jekyll::Commands::Serve.process(options)  
end

desc "========== GENERATE =========="
task :generate do
  puts "========== GENERATE =========="
  Jekyll::Site.new(Jekyll.configuration({
    # "mode"        => "production",
    "source"      => ".",
    "destination" => "_site"
  })).process
end

desc "========== GENERATE AND PUBLISH =========="
task :pub, [:commit_message] => [:generate] do |t, args|
  puts "========== GENERATE AND PUBLISH =========="
  cp_r "_site/.", PUBLISH_SOURCE_PATH

  pwd = Dir.pwd
  Dir.chdir PUBLISH_SOURCE_PATH

  # CASE :: IF
  message = "UPDATED AT #{Time.now}"
  if args.commit_message.nil?
    message = message + " :: #{DEFAULT_COMMIT_MESSAGE}"
  else
    message = message + " :: #{args.commit_message}"
  end

  # CASE :: WITH_DEFAULTS
  # args.with_defaults :commit_message => DEFAULT_COMMIT_MESSAGE
  # message = "Updated at #{Time.now} :: #{args.commit_message}"
  system "git add ."
  system "git -c user.name='lindarex' -c user.email='lindarex@naver.com' commit -m #{message.inspect}"
  system "git push origin master --force"
  system "git log -5"

  Dir.chdir pwd
end

####################################################################################################
# original Rakefile
def listen_ignore_paths(base, options)
  [
    /_config\.ya?ml/,
    /_site/,
    /\.jekyll-metadata/
  ]
end

def listen_handler(base, options)
  site = Jekyll::Site.new(options)
  Jekyll::Command.process_site(site)
  proc do |modified, added, removed|
    t = Time.now
    c = modified + added + removed
    n = c.length
    relative_paths = c.map{ |p| Pathname.new(p).relative_path_from(base).to_s }
    print Jekyll.logger.message("Regenerating:", "#{relative_paths.join(", ")} changed... ")
    begin
      Jekyll::Command.process_site(site)
      puts "regenerated in #{Time.now - t} seconds."
    rescue => e
      puts "error:"
      Jekyll.logger.warn "Error:", e.message
      Jekyll.logger.warn "Error:", "Run jekyll build --trace for more information."
    end
  end
end

task :preview do
  base = Pathname.new('.').expand_path
  options = {
    "source"        => base.join('test').to_s,
    "destination"   => base.join('test/_site').to_s,
    "force_polling" => false,
    "serving"       => true,
    "theme"         => "minimal-mistakes-jekyll"
  }

  options = Jekyll.configuration(options)

  ENV["LISTEN_GEM_DEBUGGING"] = "1"
  listener = Listen.to(
    base.join("_data"),
    base.join("_includes"),
    base.join("_layouts"),
    base.join("_sass"),
    base.join("assets"),
    options["source"],
    :ignore => listen_ignore_paths(base, options),
    :force_polling => options['force_polling'],
    &(listen_handler(base, options))
  )

  begin
    listener.start
    Jekyll.logger.info "Auto-regeneration:", "enabled for '#{options["source"]}'"

    unless options['serving']
      trap("INT") do
        listener.stop
        puts "     Halting auto-regeneration."
        exit 0
      end

      loop { sleep 1000 }
    end
  rescue ThreadError
    # You pressed Ctrl-C, oh my!
  end

  Jekyll::Commands::Serve.process(options)
end

