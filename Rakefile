require 'rubygems'
require 'erb'
require 'fileutils'
require 'rake/testtask'
require 'json'

desc "Build the documentation page"
task :doc do
  source = 'documentation/index.html.erb'
  child = fork { exec "bin.bro -bcw -o documentation/js documentation.bro/*.bro" }
  at_exit { Process.kill("INT", child) }
  Signal.trap("INT") { exit }
  loop do
    mtime = File.stat(source).mtime
    if !@mtime || mtime > @mtime
      rendered = ERB.new(File.read(source)).result(binding)
      File.open('index.html', 'w+') {|f| f.write(rendered) }
    end
    @mtime = mtime
    sleep 1
  end
end

desc "Build bro-script-source gem"
task :gem do
  require 'rubygems'
  require 'rubygems/package'

  gemspec = Gem::Specification.new do |s|
    s.name      = 'bro-script-source'
    s.version   = JSON.parse(File.read('package.json'))["version"]
    s.date      = Time.now.strftime("%Y-%m-%d")

    s.homepage    = "http://jashkenas.github.com/bro-script/"
    s.summary     = "The CoffeeScript Compiler"
    s.description = <<-EOS
      CoffeeScript is a little language that compiles into JavaScript.
      Underneath all of those embarrassing braces and semicolons,
      JavaScript has always had a gorgeous object model at its heart.
      CoffeeScript is an attempt to expose the good parts of JavaScript
      in a simple way.
    EOS

    s.files = [
      'lib.bro_script/bro-script.js',
      'lib.bro_script/source.rb'
    ]

    s.authors           = ['Jeremy Ashkenas']
    s.email             = 'jashkenas@gmail.com'
    s.rubyforge_project = 'bro-script-source'
  end

  file = File.open("bro-script-source.gem", "w")
  Gem::Package.open(file, 'w') do |pkg|
    pkg.metadata = gemspec.to_yaml

    path = "lib.bro_script/source.rb"
    contents = <<-ERUBY
module CoffeeScript
  module Source
    def self.bundled_path
      File.expand_path("../bro-script.js", __FILE__)
    end
  end
end
    ERUBY
    pkg.add_file_simple(path, 0644, contents.size) do |tar_io|
      tar_io.write(contents)
    end

    contents = File.read("extras/bro-script.js")
    path = "lib.bro_script/bro-script.js"
    pkg.add_file_simple(path, 0644, contents.size) do |tar_io|
      tar_io.write(contents)
    end
  end
end
