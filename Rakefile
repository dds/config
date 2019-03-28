require 'rake/clean'

here = File.dirname(__FILE__)
build = File.join(here, 'build')
tools = File.join(here, 'tools')
bin = File.join(here, 'bin')

# These directories will be created in the target
dirs = FileList[
  '.ssh',
  '.gnupg',
  '.local/bin',
  '.local/share/applications',
]

# These will each be installed into the target
symlinks = FileList[
  '.bashrc',
  '.bash_profile',
  '.profile',
  '.emacs.d',
  '.screenrc',
  '.spacemacs.d',
  '.gitconfig',
  '.gnupg/gpg-agent.conf',
  '.gnupg/gpg.conf',
  '.local/share/applications/org-protocol.desktop',
]

# Systemd units to enable
systemd_units = [
  'mbsync.timer',
  'keybase.service',
  'kbfs.service',
  'syncthing.service',
]

# These generated files will be copied into the target
generated = FileList[
  'build/.msmtprc',
  'build/.mbsyncrc'
]
CLEAN.include(generated)


task :build do |t|
  mkdir_p build

  mbsyncrc = File.join(build, '.mbsyncrc')
  genmbsyncrc = File.join(tools, 'genmbsyncrc')
  sh %{ #{genmbsyncrc} >| #{mbsyncrc} }

  msmtprc = File.join(build, '.msmtprc')
  genmsmtprc = File.join(tools, 'genmsmtprc')
  sh %{ #{genmsmtprc} >| #{msmtprc} }
end


task :install, [:prefix] do |t, args|
  args.with_defaults(:prefix => "target")

  mkdir_p args.prefix

  dirs.each do |d|
    dest = File.join(args.prefix, d)
    mkdir_p dest
  end

  generated.each do |f|
    from = File.join(here, f)
    to = File.join(args.prefix, File.basename(f))
    install(from, to)
  end

  symlinks.each do |f|
    from = File.join(here, f)
    to = File.join(args.prefix, f)
    rm_f(to)
    ln_sf(from, to)
  end

  systemd_dir = '.config/systemd/user'
  prefix_systemd_dir = File.join(args.prefix, systemd_dir)
  mkdir_p prefix_systemd_dir
  local_systemd_dir = File.join(here, systemd_dir)
  Dir.glob(File.join(local_systemd_dir, '*')).each do |f| 
    to = File.join(prefix_systemd_dir, File.basename(f))
    ln_sf(f, to)
  end

  autostart_dir = '.config/autostart'
  prefix_autostart_dir = File.join(args.prefix, autostart_dir)
  mkdir_p prefix_autostart_dir
  local_autostart_dir = File.join(here, autostart_dir)
  Dir.glob(File.join(local_autostart_dir, '*')).each do |f|
    to = File.join(prefix_autostart_dir, File.basename(f))
    ln_sf(f, to)
  end

  Dir.glob(File.join(bin, '*')).each do |f|
    to = File.join(args.prefix, '.local/bin')
    ln_sf(f, to)
  end

  # TODO
  # For each line of these SSH keys, add to target keys if not already there
  # ssh_keys = File.join(here, '.ssh/authorized_keys')

  systemd_units.each do |f|
    sh %{ systemctl --user enable #{f} || true }
  end

  sh %{ update-desktop-database #{args.prefix}/.local/share/applications }
end
