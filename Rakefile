# Rakefile to fetch SC14 schecule
# Written by Masahiro Tanaka

require 'net/http'
require 'uri'

HOST   = 'sc14.supercomputing.org'
OUTPUT = 'sc14.vcs'
LIST   = 'list'
VCSDIR = 'vcs'

# Customize Event Selection
EVENT_SELECTION = %w[
 bof
 gb
 mswk
 inspkr
 pap
 post
 wksp
]

# ---- Abbreviation list ----
# bespkr : HPC Interconnections, Broader Engagement
# bof : BoF
# drs : Doctoral Showcase - Dissertation Research Showcase
# emt : Emerging Technologies
# eps : HPC Interconnections, HPC Educator Program
# ers : Doctoral Showcase - Early Research Showcases
# exforum : Exhibitor Forums
# gb : ACM Gordon Bell Finalists
# imp : HPC Impact Showcases
# inspkr : Keynotes & Plenary Talks
# mswk : Invited Speakers
# nre : Network Research Exhibitions
# pan : Panels
# pap : paper
# pec : others
# post : poster
# spost : ACM Student Research Competition Posters
# svs : Scientific Visualization Showcases
# tut : tutorial
# wksp : workshop
# ----------------

## Obtain Event IDs
file LIST do
  print "fetching SC14 schedule..."
  body = Net::HTTP.get(HOST,"/schedule")
  puts "done"

  evlist = []
  body.each_line do |line|
    if /vcs\.php\?evid=(\w+)'/ =~ line
      evlist << $1
    end
  end

  print "writing event id list to '#{LIST}'..."
  File.open(LIST,"w") do |f|
    evlist.sort.uniq.each do |id|
      f.puts id
    end
  end
  puts "done"
end.invoke

FileUtils.mkdir_p(VCSDIR)

VCSLIST = []
open(LIST) do |f|
  f.each do |x|
    if EVENT_SELECTION.empty? ||
        EVENT_SELECTION.any?{|sel| /#{sel}.*/ =~ x}
      VCSLIST << "#{VCSDIR}/#{x.chomp}.vcs"
    end
  end
end

## Fetch VCS
rule ".vcs" do |t|
  print "fetching #{t.name}..."
  evid = File.basename(t.name,".vcs")
  uri = URI.parse("http://#{HOST}/schedule-data/vcs.php?evid=#{evid}")
  body = Net::HTTP.get(uri)
  File.open(t.name,"w"){|f| f.write(body)}
  puts "done"
end

task :default => OUTPUT

## Concatinate VCS
file OUTPUT => VCSLIST do
  buf = <<EOL
BEGIN:VCALENDAR
PRODID:-//Microsoft Corporation//Outlook MIMEDIR//EN
VERSION:1.0
EOL
  #
  EVENT_SELECTION.each do |sel|
    Dir.glob("vcs/#{sel}*.vcs").each do |fname|
      buf << read_vevent(fname)
    end
  end
  #
  buf << <<EOL
END:VCALENDAR
EOL
  #
  File.open(OUTPUT,"w") do |f|
    buf.each_line do |line|
      f.print line.chomp+"\n"
    end
  end
end

def read_vevent(fname)
  open(fname) do |f|
    prt = false
    a = []
    found_summary = false
    f.each_line do |line|
      line.chomp!
      case line
      when /^BEGIN:VEVENT/
        prt = true
      when /^END:VEVENT/
        prt = false
      else
        if prt
          if /^SUMMARY:(.*)$/ =~ line
            if !$1.strip.empty?
              found_summary = true
              a << line
            end
          else
            a << line
          end
        end
      end
    end
    if !found_summary
      a << "SUMMARY:evid="+File.basename(fname,".vcs")
    end
    "BEGIN:VEVENT\n"+a.join("\n")+"\nEND:VEVENT\n"
  end
end

# Copyright (c) 2014 Masahiro TANAKA
#
# MIT License
#
# Permission is hereby granted, free of charge, to any person obtaining
# a copy of this software and associated documentation files (the
# "Software"), to deal in the Software without restriction, including
# without limitation the rights to use, copy, modify, merge, publish,
# distribute, sublicense, and/or sell copies of the Software, and to
# permit persons to whom the Software is furnished to do so, subject to
# the following conditions:
#
# The above copyright notice and this permission notice shall be
# included in all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
# EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
# MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
# NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
# LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
# OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
# WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
