# Rakefile to fetch SC16 schecule
# Written by Masahiro Tanaka

require 'net/http'
require 'uri'
require 'rake/clean'

HOST   = 'sc16.supercomputing.org'
PROG   = 'full-program.html'
OUTPUT = 'sc16.ics'
IDLIST = 'id_list'
ICSDIR = 'ics'
CONVERT_LF_TO_SPACE = true
OMIT_NO_SUMMARY = true

# Customize Event Selection
#EVENT_SELECTION = []
EVENT_SELECTION = %w[ bof gb inspkr inv pap pec post wksp ]

# ---- Abbreviation list ----
# #bespkr : HPC Interconnections, Broader Engagement
# bof : BoF
# drs : Doctoral Showcase - Dissertation Research Showcase
# emt : Emerging Technologies
# #eps : HPC Interconnections, HPC Educator Program
# #ers : Doctoral Showcase - Early Research Showcases
# exforum : Exhibitor Forums
# gb : ACM Gordon Bell Finalists
# hpcmat : HPC Matters
# imp : HPC Impact Showcases
# inspkr : Keynotes
# inv : Invited Speakers
# #mswk : Invited Speakers
# #nre : Network Research Exhibitions
# pan : Panels
# pap : paper
# pec : others
# post : poster
# sess : Break, Lunch
# spost : ACM Student Research Competition Posters
# svs : Scientific Visualization Showcases
# tut : tutorial
# wksp : workshop
# ----------------

CLEAN.include(IDLIST,PROG,ICSDIR,OUTPUT)

## Obtain Event IDs
file PROG do
  base = File.basename(PROG,'.html')
  print "fetching #{PROG} from #{HOST}..."
  body = Net::HTTP.get(HOST,"/#{base}/")
  File.open(PROG,"w"){|f| f.write body}
  puts "done"
end

file IDLIST => PROG do
  evlist = []
  File.open(PROG,"r").read.scan(/get_cal\.php\?id=(\w+)/){|a| evlist << a}
  print "writing event id list to '#{IDLIST}'..."
  File.open(IDLIST,"w") do |f|
    evlist.sort.uniq.each{|id| f.puts id}
  end
  puts "done"
end

directory ICSDIR

### Concatinate ICS
file OUTPUT => [IDLIST,ICSDIR] do |t|
  ICSLIST = []
  open(IDLIST) do |f|
    f.each do |x|
      if EVENT_SELECTION.empty? || EVENT_SELECTION.any?{|sel| /#{sel}.*/ =~ x}
        ICSLIST << "#{ICSDIR}/#{x.chomp}.ics"
      end
    end
  end
  #
  task(:get_ics => ICSLIST).invoke
  #
  a1,a3 = read_envelope(ICSLIST.first)
  buf = a1.join("\n")
  ICSLIST.each{|fname| buf << read_vevent(fname)}
  buf << a3.join("\n")
  #
  print "writing ICS to '#{OUTPUT}'..."
  File.open(OUTPUT,"w") do |f|
    buf.each_line{|line| f.print line.chomp+"\n"}
  end
  puts "done"
end

## Fetch ICS
rule ".ics" do |t|
  print "fetching #{t.name}..."
  evid = File.basename(t.name,".ics")
  uri = URI.parse("http://sc16.supercomputing.org/wp-content/plugins/linklings_wp_program/get_cal.php?id=#{evid}")
  body = Net::HTTP.get(uri)
  File.open(t.name,"w"){|f| f.write(body)}
  puts "done"
end

## target
task :default => OUTPUT

## read ics envelope
def read_envelope(fname)
  a1 = a = []
  a2 = []
  a3 = []
  File.open(fname,encoding:"ISO-8859-1") do |f|
    f.each_line do |line|
      line.chomp!
      case line
      when /^BEGIN:VEVENT/
        a = a2
      when /^END:VEVENT/
        a = a3
      else
        a << line
      end
    end
  end
  [a1,a3]
end

## read ics body
def read_vevent(fname)
  File.open(fname,encoding:"ISO-8859-1") do |f|
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
          case line
          when /^SUMMARY:(.*)$/
            if !$1.strip.empty?
              found_summary = true
              a << line
            end
          when /^\w+;ENCODING=QUOTED-PRINTABLE:/
            a << CONVERT_LF_TO_SPACE ? line.gsub(/=0A/," ") : line
          else
            a << line
          end
        end
      end
    end

    if !found_summary
      if OMIT_NO_SUMMARY
        return ""
      else
        a << "SUMMARY:evid="+File.basename(fname,".ics")
      end
    end
    "BEGIN:VEVENT\n"+a.join("\n")+"\nEND:VEVENT\n"
  end
end
