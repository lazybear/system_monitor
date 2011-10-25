#    Copyright (C) 2011 Alexandre Berman (sashka@lazybear.net)
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:

# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.

# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
# THE SOFTWARE.
#
# -- usage: rake help

require "rubygems"
require "bundler/setup"
require "sys/cpu"
require "sys/proctable"
require "sys/filesystem"
require "net/telnet"
include Sys

# -- global vars
task :default => [:run]

# -- send stats to RRDTool over network (assuming RRDcached is running)
def send_to_rrdtool(cpu, procs, disk_used)
   puts "\n=> sending stats to RRDTool...\n"
   session = Net::Telnet::new("Host" => @rrdcached_host, "Port" => @rrdcached_port, "Timeout" => 5, 
                              "Prompt" => /^0 errors/)  { |resp| 
      print "=> "+resp
   }
   session.cmd("UPDATE #{@rrdtool_file} #{Time.now.to_i.to_s}:#{cpu}:#{procs}:#{disk_used}") { |c| print c }
   session.close
end

# -- help
desc "-- help..."
task :help do
   puts "\n\nUsage:\n\n"
   puts "a) rake HOST=<rrdcached_host> PORT=<rrdcached_port> FILE=<rrdtool_database_file> (-- using RRDTool)\n\n"
   puts "   Eg: rake HOST=localhost PORT=8888 FILE=/Users/sashka/work/system_monitor/monitor.rrd\n\n"
   puts "b) rake local (-- just print stats on STDOUT and exit)\n\n"
end

# -- check to make sure we are correctly setup
def do_setup
   if (ENV['HOST'] != nil and ENV['PORT'] != nil and ENV['FILE'])
      @rrdcached_host = ENV['HOST'] 
      @rrdcached_port = ENV['PORT']
      @rrdtool_file   = ENV['FILE']
   else
      raise "-- ERROR: HOST, PORT, FILE env variables are not defined, can't send stats to RRDTool... Exiting."
   end
end

# -- get stats
desc "-- getting stats..."
task :run do
   do_setup
   cpu, procs, b_used = do_run
   send_to_rrdtool(cpu, procs, b_used)
   clean_exit
end

# -- get stats: local only, no RRDTool
desc "-- getting stats for local consumption, without use of RRDTool..."
task :local do
   do_run
   clean_exit
end

# -- main method
def do_run
   # -- sys/cpu only gets average CPU usage, if we want specific usage at a given time, use:
   #    'top -l 1' (and then massage it with regex to get just CPU info)
   # -- CPU.load_avg[]: 1, 5 and 15 min averages
   #puts "Load averages: " + CPU.load_avg.join(", ")
   # -- CPU
   cpu = CPU.load_avg[0].to_s
   puts "\n\n-- CPU average (1 min): " + cpu
   # -- Procs
   procs = ProcTable.ps.length.to_s
   puts "-- Processes          : " + procs
   # -- Disk usage
   stat = Sys::Filesystem.stat("/")
   b_available = stat.block_size * stat.blocks_free / 1024
   b_total     = stat.block_size * stat.blocks / 1024
   b_used      = b_total - b_available
   puts "-- Disk used          : " + b_used.to_s
   return cpu, procs, b_used
end

# -- what do we do on exit ?
def clean_exit
   puts("\n=> DONE\n\n")
   exit(0)
end
