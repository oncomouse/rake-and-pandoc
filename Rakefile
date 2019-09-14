# frozen_string_literal: true

require 'fileutils'
require 'yaml'
require 'tempfile'

# Set to the location where compiled output should be placed. You will also
# want to add this location to your .gitignore file:
OUTPUT_DIRECTORY = 'dist'
# Set this to the name of the CSL file you wish to use to format your citations.
# You can install CSL files by cloning the CSL styles repository to your system.
# For instance,
#
#    git clone https://github.com/citation-style-language/styles ~/.csl
#
# Will install the CSL files into the location pandoc usually looks for them.
#
# This is currently set to use Chicago fullnote, but you could set to any system
# supported in CSL, such as modern-language-association
CSL = 'chicago-fullnote-bibliography'
# Set to the location of a bibliography file stored outside the repository:
BIBLIOGRAPHY = nil # For instance, '~/global.bib'

OUTPUT_TARGETS = {
  pdf: [
    '--pdf-engine=xelatex'
  ],
  html: [],
  docx: [],
  odt: [],
  epub: []
}.freeze

TEMPFILE = Tempfile.new(%w[tmp md])

def bibliography_switch
  bib_file = File.basename Dir.pwd
  if !BIBLIOGRAPHY.nil? && File.exist?(BIBLIOGRAPHY)
    "--bibliography=#{BIBLIOGRAPHY} --csl=#{CSL}.csl"
  elsif File.exist? "#{Dir.pwd}/#{bib_file}.bib"
    "--bibliography=#{bib_file}.bib --csl=#{CSL}.csl"
  else
    ''
  end
end

def pandoc(output_name, output, options = '')
  file = TEMPFILE.path
  sh %(pandoc -s -f markdown+smart -o #{output_name}.#{output} #{file} #{bibliography_switch} #{options})
  Rake::Task['cleanup'].invoke
end

def grab_and_strip_yaml_metadata(contents)
  if contents[0..2] == '---'
    yaml_string = contents[3..-1].split(/^(---|\.\.\.)$/)[0]
    yaml = YAML.safe_load yaml_string
    contents.gsub!(yaml_string, '')
    contents.gsub!(/^(------|---\.\.\.)/, '')
  else
    yaml = {}
  end
  [yaml, contents]
end

def concat_file(fp, file, strip_metadata = false)
  contents = IO.read(file)
  if strip_metadata
    (yaml, contents) = grab_and_strip_yaml_metadata(contents)
    contents = "\n##{yaml['title']}\n\n#{contents}" if yaml.key? 'title'
  end
  if File.basename(file) =~ /^([0-9]{1,3})/ && File.exist?("chapters/notes/#{Regexp.last_match(1)}-notes.md")
    contents += "\n\n" + IO.read("chapters/notes/#{Regexp.last_match(1)}-notes.md")
  elsif file =~ %r{articles/} && File.exist?("articles/notes/#{File.basename(file, '.md')}-notes.md")
    contents += "\n\n" + IO.read("articles/notes/#{File.basename(file, '.md')}-notes.md")
  end
  fp.puts contents
end

def strip_blockquotes(content)
  content.gsub(/\>.*\n/, '')
end

task :setup do
  FileUtils.mkdir_p OUTPUT_DIRECTORY unless File.directory? OUTPUT_DIRECTORY
end

# Tasks for working with individual chapters using Pandoc.
# They expect files to be stored in a directory called articles and named
# <article-tag>.md. Put notes in articles/notes in file named
# <article-tag>-notes.md. Do not put spaces in the file names!
#
# Invoke pandoc compilation with:
#     rake ch<chapter-number>:<operation>
# <chapter-number> should not have leading zeros! (so a file named
# 00-frontmatter would have a chapter number of 0)
# <operation> can be pdf, docx, or fog:
# - pdf: compile article to dist/<chapter-filename>.pdf
# - docx: compile article to dist/<chapter-filename>.docx
# - fog: print information about document style and fog index (requires the UNIX
#        command "style" to be installed) "style" can be installed on OSX using
#        Homebrew. It is part of the "diction" package ("brew install diction").
#        On Ubuntu, "apt-get install diction" will install "style".
Dir.glob('chapters/*.md').each do |chapter|
  next unless File.basename(chapter) =~ /^([0-9]{1,3})/

  namespace "ch#{Regexp.last_match(1).to_i}".to_sym do
    output_file = "#{OUTPUT_DIRECTORY}/#{File.basename(chapter, '.md')}"
    task concat: [:setup] do
      concat_file(TEMPFILE, chapter)
      TEMPFILE.close
    end
    OUTPUT_TARGETS.each do |target, options|
      task target => [:concat] do
        pandoc(output_file, target.to_s, options.join(' '))
      end
    end
    task :fog do
      sh %( cat #{chapter} | sed "s/\>.*\\n//" | pandoc -t plain | style )
    end
  end
end

# Tasks for working with the entire manuscript in pandoc.
#
# Invoke pandoc compilation with:
#     rake <operation>
# <operation> can be pdf, docx, or fog:
# - pdf: compile article to dist/book.pdf
# - docx: compile article to dist/book.docx
# - fog: print information about document style and fog index (requires the UNIX
#        command "style" to be installed) "style" can be installed on OSX using
#        Homebrew. It is part of the "diction" package ("brew install diction").
#        On Ubuntu, "apt-get install diction" will install "style".
task concat: [:setup] do
  # Grab the first file's YAML metadata and treat it as the book's:
  first_file = Dir.glob('chapters/*.md').first
  (yaml,) = grab_and_strip_yaml_metadata(IO.read(first_file))
  TEMPFILE.puts "#{YAML.dump(yaml)}---\n"
  Dir.glob('chapters/*.md') do |file|
    concat_file(TEMPFILE, file, file != first_file)
  end
  TEMPFILE.close
end

OUTPUT_TARGETS.each do |target, options|
  task target => [:concat] do
    pandoc("#{OUTPUT_DIRECTORY}/book", target.to_s, options.join(' '))
  end
end

task :cleanup do
  # FileUtils.rm("#{OUTPUT_DIRECTORY}/tmp.md")
  TEMPFILE.unlink
end
task fog: [:concat] do
  sh %( cat "#{TEMPFILE.path}" | sed "s/\>.*\\n//" | pandoc -t plain | style )
  Rake::Task['cleanup'].invoke
end

task default: [:docx] do
end

