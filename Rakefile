require 'net/http'
require 'rake/clean'

task :default => [:wip]

SOURCE_FILES = FileList['livro/livro.asc', 'livro/capitulos/*']
@RELEASE_DIR = 'releases/current'
@BOOK_SOURCE_DIR = 'livro'
@BOOK_SOURCE = 'livro/livro.asc'
@BOOK_TARGET = 'livro/livro.pdf'
@A2X_BIN = '~/ambiente/asciidoc/a2x.py'
WIP_ADOC = "#{@BOOK_SOURCE_DIR}/wip.adoc"
RELEASE_BOOK_SOURCE = "#{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}/livro.asc"
RELEASE_BOOK  = "#{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}/livro.pdf"
RELEASE_WIP_ADOC =  "#{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}/wip.adoc"
RELEASE_WIP_PDF  =  "#{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}/wip.pdf"
OPEN_PDF_CMD=`git config --get producao.pdfviewer`.strip
A2X_COMMAND="-v -k -f pdf --icons -a docinfo1 -a edition=`git describe` -a lang=pt-BR -d book --dblatex-opts '-T computacao -P latex.babel.language=brazilian' -a livro-pdf"
PROJECT_NAME = File.basename(Dir.getwd)
LIVRO_URL = `git config --get livro.url`.strip

directory @RELEASE_DIR

CLEAN.include('releases')

desc "Sync, build and open wip file"
task :wip => [WIP_ADOC, "sync", "wip:build", "wip:open"]
task :edit => ["wip:edit"]

namespace "wip" do

  desc "Create new wip file from book source"
  task "new" do
    cp "#{@BOOK_SOURCE}", "#{@BOOK_SOURCE_DIR}/wip.adoc"
  end

  file WIP_ADOC do
    Rake::Task["wip:new"].invoke
  end

  file RELEASE_WIP_PDF do
    system "#{@A2X_BIN} #{A2X_COMMAND} #{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}/wip.adoc"
  end
  
  desc "Open wip pdf"
  task :open => RELEASE_WIP_PDF do |t|
      puts "#{OPEN_PDF_CMD} #{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}/wip.pdf"
      system "#{OPEN_PDF_CMD} #{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}/wip.pdf"
  end

  desc "open docbook xml file"
  task "xml" do
    system "#{OPEN_PDF_CMD} #{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}/wip.xml"
  end

  desc "Edit source"
  task "edit" do
    system "gvim #{WIP_ADOC}"
  end

  desc "build book from #{@RELEASE_DIR}"
  task :build => [WIP_ADOC, :sync] do
    system "#{@A2X_BIN} #{A2X_COMMAND} #{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}/wip.adoc"
  end

end


desc "Sync, build and open book file"
task :book => [:clean, :archive, "book:build", "book:open"]

namespace "book" do

  desc "Build book"
  task :build => ['sync'] do
    system "#{@A2X_BIN} #{A2X_COMMAND} #{@RELEASE_DIR}/#{@BOOK_SOURCE}"
  end

  desc "open pdf book"
  task "open" do
    system "#{OPEN_PDF_CMD} #{@RELEASE_DIR}/#{@BOOK_TARGET}"
  end

  desc "open docbook xml file"
  task "xml" do
    system "#{OPEN_PDF_CMD} #{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}/livro.xml"
  end

  desc "Edit source"
  task "edit" do
    system "gvim #{@BOOK_SOURCE}"
  end
  
  desc "Release new edition book"
  task :release, [:edition] do |t, args|
    #PROJECT = sh "`git config --get remote.origin.url | cut -f 2 -d / | cut -f 1 -d .`"
    puts "PROJECT_NAME='#{PROJECT_NAME}' LIVRO_URL='#{LIVRO_URL}'"
    mkdir_p "~/releases/#{PROJECT_NAME}"
    cd "~/releases/#{PROJECT_NAME}"
    `wget #{LIVRO_URL}`
    puts "Salvando arquivo em #{Dir.getwd}"
    mv "livro.pdf", "#{PROJECT_NAME}-#{args.edition}.pdf"
    #Dir.mkdir(File.join(Dir.home, ".foo"), 0700)
  end
  
end

desc "archive files from git"
task :archive => :clean do
  system "git archive --format=tar --prefix=#{@RELEASE_DIR}/ HEAD | (tar xf -) "
end

desc "local sync of the files to #{@RELEASE_DIR}"
task :sync => @RELEASE_DIR do |t|
  system "rsync -r --delete #{@BOOK_SOURCE_DIR}/ #{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}"
end

namespace "tag" do

  desc "List project tags"
  task :list do
    sh "git tag --list"
  end
  
  desc "Aplly a tag to the project. It will be used as the edition."
  task :apply, [:tag] do |t, args|
    sh "git tag -a #{args.tag} -m 'Gerando versão #{args.tag}'"
  end
  
  desc "Delete a tag applied."
  task :delete, [:tag] do |t,args|
    sh "git tag -d #{args.tag}"
  end
  
  desc "Push tags"
  task "push" do
    sh "git push origin --tags"
  end
  
  desc "Compare tag with HEAD"
  task :compare, [:v] do |t, args|
    sh "git log --format='- %s. ' #{args.v}..HEAD"
  end
  
end


desc "Open orginal pdf to work"
task :original do
    sh "#{OPEN_PDF_CMD} original/original.pdf"
end


namespace "config" do

  desc "Configure open command. xdg-open for ubuntu and open for osx"
  task :pdfviewer, [:app] do |t,args|
    sh "git config --global producao.pdfviewer #{args.app}"
  end

end

desc "Download new Rakefile"
task :uprake do
  `wget --output-document=Rakefile https://raw.githubusercontent.com/edusantana/novo-livro/master/Rakefile`
end

