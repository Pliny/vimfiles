# Requirements:
#  - ctags
#  - vim with ruby support for Command-T
#  - vim version 7.3.584+ needed for YouCompleteMe
#  - Get latest vundle here: git@github.com:gmarik/vundle.git
#    and place it  vimfiles/bundle/

task :default => [:link, :tmp_dirs, :update, :command_t, :youcompleteme]

desc %(Update or create bundles in the bundle/ directory)
task :update do
  sh "vim +BundleInstall +qall"
end

desc %(Make ~/.vimrc and ~/.gvimrc symlinks)
task :link do
  %w[vimrc gvimrc].each do |script|
    dotfile = File.join(ENV['HOME'], ".#{script}")
    if File.exist? dotfile
      warn "~/.#{script} already exists"
    else
      ln_s File.join('.vim', script), dotfile
    end
  end
end

task :tmp_dirs do
  mkdir_p "_backup"
  mkdir_p "_temp"
end

desc %(Compile YouCompleteMe plugin)
task :youcompleteme => :macvim_check do
  Dir.chdir "bundle/YouCompleteMe" do
    puts "Compiling YouCompleteMe plugin..."
    sh "./install.sh --clang-completer"
  end

end

desc %(Compile Command-T plugin)
task :command_t => :macvim_check do
  vim = which('mvim') || which('vim') or abort "vim not found on your system"
  ruby = read_ruby_version(vim)

  Dir.chdir "bundle/Command-T/ruby/command-t" do
    if ruby
      puts "Compiling Command-T plugin..."
      sh(*Array(ruby).concat(%w[extconf.rb]))
      sh "make clean && make"
    else
      warn color('Warning:', 31) + " Can't compile Command-T, no ruby support in #{vim}"
      sh "make clean"
    end
  end
end

task :macvim_check do
  if mvim = which('mvim') and '/usr/bin/vim' == which('vim')
    warn color('Warning:', 31) + " You have MacVim installed, but `vim` still opens system Vim."
    warn "To use MacVim version when you invoke `vim`:  " + color("$ ln -s mvim #{File.dirname(mvim)}/vim", 37)
  end
end

def color msg, code
  if $stderr.tty? then "\e[1;#{code}m#{msg}\e[m"
  else msg
  end
end

# Read which ruby version is vim compiled against
def read_ruby_version vim
  script = %{require "rbconfig"; print File.join(RbConfig::CONFIG["bindir"], RbConfig::CONFIG["ruby_install_name"])}
  version = `#{vim} --nofork --cmd 'ruby #{script}' --cmd 'q' 2>&1 >/dev/null | grep -v 'Vim: Warning'`.strip
  version unless version.empty? or version.include?("command is not available")
end
#
# Read which python version is vim compiled against
def read_python_version vim
  script = %{print "hello"}
  version = `#{vim} --nofork --cmd 'python #{script}' --cmd 'q' 2>&1 >/dev/null | grep -v 'Vim: Warning'`.strip
  version unless version.empty? or version.include?("command is not available")
end

# Cross-platform way of finding an executable in the $PATH.
#
#   which('ruby') #=> /usr/bin/ruby
def which cmd
  exts = ENV['PATHEXT'] ? ENV['PATHEXT'].split(';') : ['']
  ENV['PATH'].split(File::PATH_SEPARATOR).each do |path|
    exts.each { |ext|
      exe = "#{path}/#{cmd}#{ext}"
      return exe if File.executable? exe
    }
  end
  return nil
end
