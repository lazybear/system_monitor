role :run_getstats, "localhost"    # -- default: define whatever other hosts here: "host1", "host2"
set :cap_user, "sashka"            # -- user which runs this task on other hosts
set :app_location, "/opt/getstats" # -- location where we want to install this app on other hosts
set :app, "Rakefile"
set :app_dep, "Gemfile"

desc "-- get system stats from hosts"
task :getstats, :roles => :run_getstats do
   run "/usr/bin/rake -f #{app_location}/#{app} HOST=localhost PORT=8888 FILE=/Users/sashka/work/system_monitor/monitor.rrd"
end

desc "-- install bundler and dependencies"
task :install, :roles => :run_getstats do
   run "sudo mkdir #{app_location}"
   run "sudo chown #{cap_user} #{app_location}"
   upload(app, app_location, :via => :scp)
   upload(app_dep, app_location, :via => :scp)
   run "sudo /usr/bin/gem install bundler"
   run "cd #{app_location} && /usr/bin/bundle install"
end
