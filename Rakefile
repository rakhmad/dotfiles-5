=begin
The goal is to be able to run `rake` on a brand new OSX system and magically
get my system back, with no need to resort to ln or installation docs and
with a minimum of input from me.
=end

require ‘rbconfig’

def info_install(pkg)
  puts "* Installing #{pkg}"
end

def info_uninstall(pkg)
  puts "* Uninstalling #{pkg}"
end


def replace_file(file)
  system %Q{rm -rf "$HOME/.#{file}"}
  link_file(file)
end

def link_file(file)
  puts "linking ~/.#{file}"
  system %Q{ln -s "$PWD/#{file}" "$HOME/.#{file}"}
end

def is_windows?
  Config::CONFIG[‘host_os’] =~ /mswin|mingw/
end

def is_ruby_19?
  RUBY_VERSION.to_f == 1.9
end

# platform-specific way to represent a dotfile
# .file for Unix, _file for Windows
def dotfile_path(fname)
  if is_windows?
    "_#{fname}"
  else
    ".#{fname}"
  end
end

def home_directory
  if is_ruby_19?
    # Ruby 1.9 handles Windows home dirs just fine
    # http://redmine.ruby-lang.org/issues/show/1147
    File.expand_path("~")
  else
    if is_windows?
      ENV['USERPROFILE']
    else
      ENV['HOME']
    end
  end
end

# link dotfiles into ~
namespace :link do
  task :all do
    # Via ryan bates, https://github.com/ryanb/dotfiles/blob/master/Rakefile
    replace_all = false
    home_dir = home_directory()
    Dir['*'].each do |file|
      next if %w[Rakefile README.rdoc LICENSE].include?(file)
      next if %w[Rakefile README.textile .gitignore .gitkeep].include?(file)
      dotfile = dotfile_path("#{file}")

      if File.exist?(File.join(home_dir, dotfile))
        if File.identical?(file, File.join(home_dir, dotfile))
          puts "identical ~/#{dotfile}"
        elsif replace_all
          replace_file(file)
        else
          print "overwrite ~/#{dotfile}? [ynaq] "
          case $stdin.gets.chomp
          when 'a'
            replace_all = true
            replace_file(file)
          when 'y'
            replace_file(file)
          when 'q'
            exit
          else
            puts "skipping ~/#{dotfile}"
          end
        end
      else
        link_file(file)
      end
    end
  end
end

namespace :install do
  task :homebrew do
    info_install 'homebrew'
    puts 'You can ignore this message: "/usr/local/.git already exists!"'
    system 'ruby -e "$(curl -fsSL https://gist.github.com/raw/323731/install_homebrew.rb)"'
  end

  # Helpful brews via homebrew
  task :brews => [:homebrew] do
    system <<-EOF
      brew install android-sdk android-ndk \
        colordiff colormake dos2unix \
        fortune git macvim \
        nethack pip python readline \
        watch
    EOF
  end

  task :slime do
    system "cvs -d :pserver:anonymous:anonymous@common-lisp.net:/project/slime/cvsroot co -d emacs-plugins/slime slime"
  end

  task :rvm do
    info_install 'RVM'
    # Requires "bash -c" because by default, the system command uses
    # /bin/sh, which chokes on the "<"s
    system '/bin/bash -c "bash < <( curl http://rvm.beginrescueend.com/releases/rvm-install-head )"'
  end

  task :all => [:brews, :rvm]
end

namespace :uninstall do
  task :homebrew do
    info_uninstall 'homebrew'

    system <<-EOF
      cd `brew --prefix`';
      rm -rf Cellar;
      system brew prune;
      rm -rf Library .git .gitignore bin/brew README.md share/man/man1/brew;
      rm -rf ~/Library/Caches/Homebrew;
    EOF
  end

  task :rvm do
    info_uninstall 'RVM'
    puts "!!! This command requires confirmation!"
    system "rvm implode"
  end

  task :all => [:homebrew, :rvm]
end

namespace :update do
  task :vim_plugins do
    $LOAD_PATH << File.join(File.dirname(__FILE__), 'vim')
    require 'update_bundles'
    update_all()
  end

  # cvs checkout will update it too
  task :slime => ['install:slime']
end

task :default => ['install:all', 'link:all']
