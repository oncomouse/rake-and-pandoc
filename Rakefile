require 'fileutils'
require 'yaml'

# Set to the location where compiled output should be placed (add this to your .gitignore file!)
$output_directory = "dist"
# CSL files need to be installed in ~/.csl (or wherever pandoc-citeproc is configured to look on your system):
$csl = "chicago-note-bibliography" #"chicago-author-date"#"chicago-fullnote-bibliography"#"modern-language-association"#"chicago-note-bibliography"
# Uncomment this and set to the location of a bibliography file stored outside the repository:
#$bibliography = "~/global-bibliography.bib"

$output_targets = {
	pdf: [
		'--latex-engine=xelatex'
	],
	html: [],
	docx: [],
	odt: [],
	epub: []
}

def pandoc(output_name, output, options="")
	file = "#{$output_directory}/tmp.md" # Always read from the tmp file (we dump notes there).
    bib_file = File.basename Dir.pwd
	if defined? $bibliography and File.exist? $bibliography
		bibliography_switch =  "--bibliography=#{$bibliography}"
	elsif File.exists? "#{Dir.pwd}/#{bib_file}.bib"
        bibliography_switch =  "--bibliography=#{bib_file}.bib"
    else
        bibliography_switch = ""
    end
	csl_switch = "--csl=#{$csl}.csl"
	if not defined? $csl
		csl_switch = ""
	end
    sh %{ pandoc -s -f markdown+smart --filter=pandoc-citeproc -o #{output_name}.#{output} #{file} #{bibliography_switch} #{csl_switch} #{options}}
	
	Rake::Task["cleanup"].invoke
end

def grab_and_strip_yaml_metadata(contents)
	if(contents[0..2] == "---")
		yaml_string = contents[3..-1].split(/^(---|\.\.\.)$/)[0]
		yaml = YAML.load yaml_string
		contents.gsub!(yaml_string,"")
		contents.gsub!(/^(------|---\.\.\.)/,"")
	else
		yaml = {}
	end
	[yaml, contents]
end

def concat_file(fp, file, strip_metadata=false)
	contents = IO.read(file)
	if(strip_metadata)
		(yaml, contents) = grab_and_strip_yaml_metadata(contents)
		if yaml.has_key? "title"
			contents = "\n##{yaml["title"]}\n\n#{contents}"
		end
	end
	if(File.basename(file) =~ /^([0-9]{1,3})/ and File.exist? "chapters/notes/#{$1}-notes.md")
		contents += "\n\n" + IO.read("chapters/notes/#{$1}-notes.md")
	elsif file =~ /articles\// and File.exist? "articles/notes/#{File.basename(file, '.md')}-notes.md"
		contents += "\n\n" + IO.read("articles/notes/#{File.basename(file, '.md')}-notes.md")
	end
	fp.puts contents
end

def strip_blockquotes(content)
	content.gsub(/\>.*\n/,"")
end


task :setup do
	if not File.directory? $output_directory
		FileUtils.mkdir_p $output_directory
	end
end

# Tasks for working with stand-alone articles based on your manuscript using Pandoc.
#
# They expect files to be stored in a directory called articles and named <article-tag>.md. Put notes in articles/notes
# in file named <article-tag>-notes.md. Do not put spaces in the file names!
#
# Invoke pandoc compilation with: 
#    rake articles:<article-tag>:<operation>
# <operation> can be pdf, docx, or fog:
# - pdf: compile article to dist/<article-tag>.pdf
# - docx: compile article to dist/<article-tag>.docx
# - fog: print information about document style and fog index (requires the UNIX command "style" to be installed)
#        "style" can be installed on OSX using Homebrew. It is part of the "diction" package (run "brew install diction")
namespace :articles do
	Dir.glob("articles/*.md").each do |article|
		base_file_name = File.basename(article,'.md')
		puts base_file_name
		namespace base_file_name.to_sym do
			output_file = "#{$output_directory}/#{base_file_name}"
			task concat: [:setup] do
				File.open("#{$output_directory}/tmp.md", "w") do |fp|
					concat_file(fp,article)
				end
			end
			$output_targets.each do |target, options|
				task target => [:concat] do
					pandoc(output_file, target.to_s, options.join(" "))
				end
			end
			task :fog do
				sh %{ cat #{article} | sed "s/\>.*\\n//" | pandoc -t plain | style }
			end
		end
	end
end

# Tasks for working with individual chapters using Pandoc.
# They expect files to be stored in a directory called articles and named <article-tag>.md. Put notes in articles/notes
# in file named <article-tag>-notes.md. Do not put spaces in the file names!
#
# Invoke pandoc compilation with:
#     rake ch<chapter-number>:<operation>
# <chapter-number> should not have leading zeros! (so a file named 00-frontmatter would have a chapter number of 0)
# <operation> can be pdf, docx, or fog:
# - pdf: compile article to dist/<chapter-filename>.pdf
# - docx: compile article to dist/<chapter-filename>.docx
# - fog: print information about document style and fog index (requires the UNIX command "style" to be installed)
#        "style" can be installed on OSX using Homebrew. It is part of the "diction" package (run "brew install diction")
Dir.glob("chapters/*.md").each do |chapter|
	if(File.basename(chapter) =~ /^([0-9]{1,3})/)
		namespace "ch#{$1.to_i}".to_sym do
			output_file = "#{$output_directory}/#{File.basename(chapter,'.md')}"
			task concat: [:setup] do
				File.open("#{$output_directory}/tmp.md", "w") do |fp|
					concat_file(fp,chapter)
				end
			end
			$output_targets.each do |target, options|
				task target => [:concat] do
					pandoc(output_file, target.to_s, options.join(" "))
				end
			end
			task :fog do
				sh %{ cat #{chapter} | sed "s/\>.*\\n//" | pandoc -t plain | style }
			end
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
# - fog: print information about document style and fog index (requires the UNIX command "style" to be installed)
#        "style" can be installed on OSX using Homebrew. It is part of the "diction" package (run "brew install diction")
task concat: [:setup] do
	File.open("#{$output_directory}/tmp.md", 'w') do |fp|
		# Grab the first file's YAML metadata and treat it as the book's:
		first_file = Dir.glob('chapters/*.md').first
		(yaml, foo) = grab_and_strip_yaml_metadata(IO.read(first_file))
		fp.puts "#{YAML.dump(yaml)}---\n"
		Dir.glob('chapters/*.md') do |file|
			concat_file(fp, file, file != first_file)
		end
	end
end

$output_targets.each do |target, options|
	task target => [:concat] do
		pandoc("#{$output_directory}/book", target.to_s, options.join(" "))
	end
end

task :cleanup do
	FileUtils.rm("#{$output_directory}/tmp.md")
end
task fog: [:concat] do
	sh %{ cat "#{$output_directory}/tmp.md" | sed "s/\>.*\\n//" | pandoc -t plain | style }
	Rake::Task["cleanup"].invoke
end

task :default => [:docx] do
end