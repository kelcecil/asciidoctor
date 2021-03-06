require File.expand_path '../lib/asciidoctor/version', __FILE__

def prepare_test_env
  # rather than hardcoding gc settings in test task,
  # could use https://gist.github.com/benders/788695
  ENV['RUBY_GC_MALLOC_LIMIT'] = '90000000'
  ENV['RUBY_FREE_MIN'] = '200000'
end

begin
  require 'rake/testtask'
  Rake::TestTask.new(:test) do |test|
    prepare_test_env
    puts "LANG: #{ENV['LANG']}"
    test.libs << 'test'
    test.pattern = 'test/**/*_test.rb'
    test.verbose = true
    test.warning = true
  end
  task :default => :test
rescue LoadError
end

=begin
# Run tests with Encoding::default_external set to US-ASCII
begin
  Rake::TestTask.new(:test_us_ascii) do |test|
    prepare_test_env
    puts "LANG: #{ENV['LANG']}"
    test.libs << 'test'
    test.pattern = 'test/**/*_test.rb'
    test.ruby_opts << '-EUS-ASCII' if RUBY_VERSION >= '1.9'
    test.verbose = true
    test.warning = true
  end
rescue LoadError
end
=end

begin
  require 'cucumber/rake/task'
  Cucumber::Rake::Task.new(:features) do |t|
  end
rescue LoadError
end

=begin
begin
  require 'rdoc/task'
  RDoc::Task.new do |rdoc|
    rdoc.rdoc_dir = 'rdoc'
    rdoc.title = "Asciidoctor #{Asciidoctor::VERSION}"
    rdoc.markup = 'tomdoc' if rdoc.respond_to?(:markup)
    rdoc.rdoc_files.include('LICENSE', 'lib/**/*.rb')
  end
rescue LoadError
end
=end

begin
  require 'yard'
  require 'yard-tomdoc'
  require './lib/asciidoctor'
  require './lib/asciidoctor/extensions'

  # Prevent YARD from breaking command statements in literal paragraphs
  class CommandBlockPostprocessor < Asciidoctor::Extensions::Postprocessor
    def process output
      output.gsub(/<pre>\$ (.+?)<\/pre>/m, '<pre class="command code"><span class="const">$</span> \1</pre>')
    end
  end
  Asciidoctor::Extensions.register do |doc|
    postprocessor CommandBlockPostprocessor
  end

  # register .adoc extension for AsciiDoc markup helper
  YARD::Templates::Helpers::MarkupHelper::MARKUP_EXTENSIONS[:asciidoc] = %w(adoc)
  YARD::Rake::YardocTask.new do |yard|
    yard.files = %w(
        lib/**/*.rb
        -
        CHANGELOG.adoc
        LICENSE
    )
    # --no-highlight enabled to prevent verbatim blocks in AsciiDoc that begin with $ from being dropped
    # need to patch htmlify method to not attempt to syntax highlight blocks (or fix what's wrong)
    yard.options = %w(
        --exclude backends
        --exclude opal_ext
        --hide-api private
        -o rdoc
        --plugin tomdoc
        --title Asciidoctor\ API\ Documentation
    )
  end
rescue LoadError
end

begin
  require 'bundler/gem_tasks'

  # Enhance the release task to create an explicit commit for the release
  Rake::Task[:release].enhance [:commit_release]

  task :commit_release do
    Bundler::GemHelper.new.send(:guard_clean)
    sh "git commit --allow-empty -a -m 'Release #{Asciidoctor::VERSION}'"
  end
rescue LoadError
end

desc 'Open an irb session preloaded with this library'
task :console do
  sh 'bundle console', :verbose => false
end
