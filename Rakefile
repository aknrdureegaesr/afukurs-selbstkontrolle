# coding: utf-8
require "yaml"

# The BNetzA re-uses question numbers for the T-exam for class E and class A.
# So, e.g., TA101 is one question for T-exam E and a DIFFERENT question for T-exam A.
# Internally, we pre-fix the class A questions with an aditional "A".

desc "Do most (not included: save_question_conflicts and harvest_l)."
task :default => [:generate_qlt, :generate_l_qlt, :generate_cumulative_qlt, :generate_cumulative_qlt_berlin, :list_forgotten_questions]

# Maps from sections (e.g.: 'bv07') to enumerables of questions (e.g.: ['VB101', 'VB102']).
# From harvest.json, harvested from web site course material via subdirectory `harvest`:
@section2qs_harvest = {}
# From otherwise_forgotten.yaml, which is manually crafted,
# and also from its sybling unmentioned_class_a.yaml:
@section2qs_from_otherwise_forgotten = {}

# For each question, the family of questions considered related to
# each other.  This data historically comes from previous class E courses organized
# independently (that is, rather different split-up in sections), technically from
# file "learn_batches.yaml".
@q2fam = {}
# For each question, the general theme of that question and all others in the same family.
@q2theme = {}

# Questions that could belong to a section, as they are related.
# Generated here from the above data.
# May contain the same question in several sections.
@section2qs_closure = {}

# For some of the questions, advice about the course section.
# From conflict_resolution.yaml, which is edited manually.
@q2section_resolved = {}

# The final answer to "which question in which section",
# determined by mixing the above information.
# May not contain all sections. Also, answers for individual sections
# may be incomplete.
@section2qs_answer = {}
# For each section that cannot yet be generated, that is,
# which may contain certain questions we are not sure about yet,
# the questions that need to be added to conflict_resolution.yaml
# to allow generation.
# For a given section, you can either rely on
# @section2qs_answer[section] being complete, or else you'll have
# @section2conflicts.key? section
@section2conflicts = {}

# According to Eckart http://www.amateurfunkpruefung.de/lehrg/aufgabensuche_E.html ,
# and, according to (yea! to succint URIs)
# https://www.bundesnetzagentur.de/SharedDocs/Downloads/DE/Sachgebiete/Telekommunikation/Unternehmen_Institutionen/Frequenzen/Amateurfunk/Fragenkatalog/Liste_der_nicht_mehr_relavanten_Fragen.pdf?__blob=publicationFile&v=1
# the following questions are never ever asked and hence highly non-pertinent:
@never_asked_questions = \
Set.new [
  # questions never asked according to Eckart
  'TB702', 'TB804', 'TE303', 'TE306', 'TG203', 'TK303', 'TK305', 'TK309', 'TK319',
  # questions never asked according to BNetzA:
  'ATE112', 'ATH117', 'ATH123', 'ATH146', 'ATH147', 'ATI101', 'ATL217', 'ATL218',
  'TC503', 'TE104', 'TI101', 'TL214',
  'BC105', 'BC216', 'BE308', 'BF109',
  'VE102', 'VE131', 'VE136', 'VE148', 'VE153', 'VG103', 'VG104', 'VG106', 'VG107', 'VG113',
  'VI111', 'VI113', 'VI114', 'VI115', 'VI122', 'VI123', 'VK104'
]

# All known questions, for double-checking purposes.
@all_qs = Set.new

desc "load harvest result from *harvest.json files"
task :load_harvest do |t|
  YAML.parse_file('harvest.json').root.children[1].children.each_slice(2) do |section_node, questions_node|
    section = section_node.value.downcase
    @section2qs_harvest[section] = Set.new unless @section2qs_harvest.key? section
    @section2qs_harvest[section].merge questions_node.children.map { |qnode| qnode.value.upcase }
  end
  ['te_harvest.json', 'ta_harvest.json', 'bv_harvest.json'].each do |harvest_file|
    YAML.parse_file(harvest_file).root.children.each do |q2sec|
      q2sec.children.each_slice(2) do |q_node, section_node|
        q = q_node.value.upcase
        section = section_node.value.downcase
        @section2qs_harvest[section] = Set.new unless @section2qs_harvest.key? section
        @section2qs_harvest[section] << q
      end
    end
  end
end

# Take a map that maps some "a" values to enumerables of "b" values,
# and reverse it (to a map that maps individual "b" values to enumerables of "a" values).
def reverse(a2bs)
  b2as = {}
  a2bs.each do |a, bs|
    bs.each do |b|
      b2as[b] = Set.new unless b2as.key? b
      b2as[b] << a
    end
  end
  b2as
end

desc "load exceptionals from otherwise_forgotten.yaml and unmentioned_class_a.yaml" # into @section2qs_from_otherwise_forgotten
task :load_otherwise_forgotten do |t|
  q2sections = {}
  YAML.parse_file('otherwise_forgotten.yaml').root.children.each_slice(2) do |q_node, sec_node|
    q, section = q_node.value.upcase, sec_node.value.downcase
    raise "double section #{q2sections[q]} vs #{section} from otherwise_forgotten.yaml" if q2sections.key? q
    q2sections[q] = [section]
  end
  class_a_yaml = YAML.parse_file('unmentioned_class_a.yaml').root
  if class_a_yaml and class_a_yaml.children
    class_a_yaml.children.each_slice(2) do |q_node, sec_node|
      q, section = "A#{q_node.value.upcase}", sec_node.value.downcase
      raise "double section #{q2sections[q]} vs #{section} from otherwise_forgotten.yaml / unmentioned_class_a,yaml" if q2sections.key? q
      q2sections[q] = [section]
    end
  end    
  @section2qs_from_otherwise_forgotten = reverse q2sections
end

desc "load learn-together families for class E from learn_batches.yaml"
task :load_families do |t|
  YAML.parse_file('learn_batches.yaml').root.children.each do |family|
    theme = family.children[1 + family.children.find_index {|s| "Thema" == s.value}].value
    q_strings = family.children[1 + family.children.find_index {|s| "Fragen" == s.value}].value
    qs = Set.new
    q_strings.split.each do |q_string|
      if /^\w\w\d\d\d$/ =~ q_string
        qs << q_string.upcase
      elsif /^(\w\w)(\d\d\d)\-(\w\w)(\d\d\d)$/ =~ q_string
        raise "I am confused about \"#{q_string}\"" unless $1 == $3
        qs.merge(($2.to_i .. $4.to_i).to_a.map{|i| "#{$1.upcase}#{i}"})
      else
        raise "I am confused about \"#{q_strings}\" -> \"#{q_string}\""
      end
    end
    qs.each do |q|
      raise "Confusing information in learn_batches.yalm: #{v.inspect} vs. #{@q2fam[q].inspect} for #{q}" if @q2fam.key? q
      @q2fam[q] = qs
      @q2theme[q] = theme
    end
  end
end

desc "calculate question closure for each session" # into @section2qs_closure
task :section_closure => [:load_harvest, :load_otherwise_forgotten, :load_families] do |t|
  (@section2qs_harvest.to_a + @section2qs_from_otherwise_forgotten.to_a).each do |section, harvest_set|
    @section2qs_closure[section] = Set.new() unless @section2qs_closure.key? section
    harvest_set.each do |q|
      if @q2fam.key? q
        @section2qs_closure[section].merge(@q2fam[q])
      else
        $stderr.print "WARN: In no family: #{q}\n" unless /AT\w\d*/ =~ q
        @section2qs_closure[section] << q
      end
    end
  end
end

desc "Read conflict_resolution.yaml"
task :read_conflict_resolution do |t|
  YAML.parse_file('conflict_resolution.yaml').root.children.each_slice(2) do |q_node, map_node|
    map_node.children.each_slice(2) do |key_node, value_node|
      if key_node.value == 'am_Besten_nach_Lektion' and value_node.value =~ /^\w\w?\d\d$/
        q, section = q_node.value.upcase, value_node.value
        raise "#{@q2section_resolved[q]} vs #{section} for #{q} in conflict_resolution.yaml" if @q2section_resolved.key? q
        @q2section_resolved[q] = section
      end
    end
  end
end

desc "Load lists of all questions"
task :load_all_questions do |t|
  ['b', 'te', 'v'].each do |subject|
    File.open("alle_#{subject}.qlt") do |f|
      @all_qs.merge f.each_line.drop(2).map{|qline| qline.chomp}.to_a
    end
  end
  File.open("alle_ta.qlt") do |f|
    @all_qs.merge f.each_line.drop(2).map{|qline| "A#{qline.chomp}"}.to_a
  end
end

desc "Run the decision algorithm."
task :decide => [ :load_otherwise_forgotten, :load_harvest, :section_closure, :read_conflict_resolution ] do |t|
  # @q2section_resolved
  q2sections_otherwise_forgotten = reverse(@section2qs_from_otherwise_forgotten)
  q2sections_harvest = reverse(@section2qs_harvest)
  q2sections_closure = reverse(@section2qs_closure)

  all_qs = Set.new(@q2section_resolved.keys)
  all_qs.merge(q2sections_otherwise_forgotten.keys)
  all_qs.merge(q2sections_harvest.keys)
  all_qs.merge(q2sections_closure.keys)

  (all_qs - @never_asked_questions).each do |q|
    # First try.
    section = @q2section_resolved[q]
    
    sections_otherwise_forgotten = q2sections_otherwise_forgotten[q]
    unless sections_otherwise_forgotten.nil? or sections_otherwise_forgotten.size == 1
      raise "ERROR: Conflicting answer #{section_otherwise_forgotten.inspect} for #{q}."
    end
    if section.nil?
      # Second try.
      section = sections_otherwise_forgotten.first unless sections_otherwise_forgotten.nil?
    else
      # First and second try should not conflict in results.
      unless sections_otherwise_forgotten.nil? or section == sections_otherwise_forgotten.first
        raise "ERROR: Conflict #{q}: #{section} in conflict_resolution.yaml vs. #{sections_otherwise_forgotten.first} in otherwise_forgotten.yaml"
      end
    end

    need_decision = section.nil?
    [q2sections_harvest, q2sections_closure].each do |q2sections_next_attempt|
      if need_decision and q2sections_next_attempt.key? q
        sections = q2sections_next_attempt[q]
        if 1 == sections.size
          section = sections.first
        else
          sections.each do |conflict_section|
            @section2conflicts[conflict_section] = Set.new unless @section2conflicts.key? conflict_section
            @section2conflicts[conflict_section] << q
          end
        end
        need_decision = false
      end
    end

    unless section.nil?
      @section2qs_answer[section] = Set.new unless @section2qs_answer.key? section
      @section2qs_answer[section] << q
    end
  end
end

def question_kind(qs)
  (qs.first)[0].downcase.to_sym
end

def write_qlt(dir, section, qs, filenamestem, upto)
  subject = question_kind(qs)
  fach = {t: 'Technik', b: 'Betriebskunde', v: 'Vorschriften', a: 'A-Technik'}[subject]
  raise "Subject not implemented: #{subject}" if fach.nil?
  filename = dir + "/#{{t: 'te', b: 'be', v: 'vo', a: 'ta'}[subject]}-#{filenamestem}.qlt".upcase
  $stderr.write("INFO: Writing #{filename}\n")
  if (qs.size < 3)
    $stderr.write "WARN: Cannot generate qlt file #{filename} for #{section}: Have only #{qs.size} of >= 3 questions, namely  #{qs.inspect}.\n"
  else
    max_show = :a == subject ? 51 : 34
    show = (max_show < qs.size ? max_show : qs.size)
    max_time = :a == subject ? 90.0 : 60.0
    time = (show * max_time / max_show + 0.8).to_i
    File.open("#{filename}", 'w') do |f|
      f.write("# #{fach}fragen #{upto ? 'bis einschl. ' : ''}Lektion #{section}\r\n")
      f.write("#{{t: 'TE', b: 'BE', v: 'VE', a: 'TA'}[subject]}=#{show}\r\n")
      f.write("Zeit=#{time}\r\n")
      unless subject == :a
        f.write(qs.sort.join("\r\n"))
      else
        f.write(qs.map{|q| q[1..q.length]}.sort.join("\r\n"))
      end
      f.write("\r\n")
    end
  end
end

desc "Generate the .qlt - files (as far as possible)"
task :generate_qlt => [:decide] do |t|
  @section2qs_answer.keys.sort.each do |section|
    kind_letters2conflicts = {}
    conflicts = @section2conflicts[section]
    unless conflicts.nil?
      conflicts.sort.chunk{|q| q[0]}.each do |kind_letter, conflicts_this_kind|
        kind_letters2conflicts[kind_letter] = conflicts_this_kind
      end
    end
    qs = @section2qs_answer[section]
    qs.sort.chunk{|q| q[0]}.each do |kind_letter, qs_one_kind|
      if kind_letters2conflicts.key? kind_letter
        $stderr.write "WARN: Cannot write #{question_kind(qs_one_kind)} for #{section}; please decide [#{kind_letters2conflicts[kind_letter].sort.join(', ')}]\n"
      else
        write_qlt "data", section, qs_one_kind, section, false
      end
    end
  end
end

def generate_cumulative(sections_and_filenamestems_in_order, dir = "data")

  kind2cumulated = {}
  kind_letters_with_conflicts = Set.new

  sections_and_filenamestems_in_order.each do |section, filenamestem|
    conflicts = @section2conflicts[section]
    unless conflicts.nil?
      conflicts.sort.chunk{|q| q[0]}.each do |kind_letter, conflicts_this_kind|
        kind_letters_with_conflicts << kind_letter
      end
    end
    qs = @section2qs_answer[section]
    qs.sort.chunk{|q| q[0]}.each do |kind_letter, qs_one_kind|
      kind = question_kind(qs_one_kind)
      kind2cumulated[kind] = Set.new unless kind2cumulated.key? kind
      unless kind_letters_with_conflicts.include? kind_letter
        write_qlt dir, section, kind2cumulated[kind].merge(qs_one_kind), filenamestem, true
      end
    end
  end
end

desc "Generate cumulative QLT files, section order used by Berlin CCC course."
task :generate_cumulative_qlt_berlin => [:decide] do |t|
  # Accomodating https://www.chaoswelle.de/Kurs , which progresses in a different order:
  generate_cumulative [ ['bv01', 'bis-bv01'],
                        ['e01', 'bis-e01'],
                        ['bv02', 'bis-bv02'],

                        ['e02', 'bis-e02'],
                        ['e03', 'bis-e03'],
                        ['bv03', 'bis-bv03'],

                        ['e04', 'bis-e04'],
                        ['bv05', 'bis-bv03-n-bv05'],

                        ['e05', 'bis-e05'],
                        ['bv06', 'bis-bv03-n-bv05-bis-bv06'],

                        ['e06', 'bis-e06'],
                        ['bv13', 'bis-bv03-n-bv05-n-bv06-n-bv13'],
                        ['e17', 'bis-e06-n-e17'],

                        ['e07', 'bis-e07-n-e17'],
                        ['bv07', 'bis-bv03-n-bv05-bis-bv07-n-bv13'],

                        ['e08', 'bis-e08-n-e17'],
                        ['e09', 'bis-e09-n-e17'],
                        
                        ['e10', 'bis-e10-n-e17'],
                        ['bv08', 'bis-bv03-n-bv05-bis-bv08-n-bv13'],

                        ['e11', 'bis-e11-n-e17'],

                        ['e12', 'bis-e12-n-e17'],
                        ['e13', 'bis-e13-n-e17'],
                        ['e14', 'bis-e14-n-e17'],

                        ['bv10', 'bis-bv03-n-bv05-bis-bv08-n-bv10-n-bv13'],
                        ['bv11', 'bis-bv03-n-bv05-bis-bv08-n-bv10-n-bv11-n-bv13'],

                        ['e15', 'bis-e15-n-e17'],
                        ['e18', 'bis-e15-n-e17-n-e18'],
                        ['bv14', 'bis-bv03-n-bv05-bis-bv08-n-bv10-n-bv11-n-bv13-n-bv14'],

                        ['e16', 'bis-e18'],
                        ['bv12', 'bis-bv03-n-bv05-bis-bv08-n-bv10-n-bv11-bis-bv14'],
                        ['bv09', 'bis-bv03-n-bv05-bis-bv14'],

                        ['bv04', 'bis-bv14'],

                        ['a01', 'bis-a01'],
                        ['a02', 'bis-a02'],

                        ['a03', 'bis-a03'],
                        ['a04', 'bis-a04'],

                        ['a05', 'bis-a05'],
                        ['a06', 'bis-a06'],

                        ['a07', 'bis-a07'],

                        ['a08', 'bis-a08'],

                        ['a09', 'bis-a09'],
                        ['a10', 'bis-a10'],

                        ['a11', 'bis-a11'],

                        ['a12', 'bis-a12'],

                        ['a13', 'bis-a13'],

                        ['a17', 'bis-a13-n-a17'],

                        ['a14', 'bis-a14-n-a17'],
                        ['a15', 'bis-a15-n-a17'],

                        ['a18', 'bis-a15-n-a17-bis-a18'],
                        ['a19', 'bis-a15-n-a17-bis-a19'],

                        ['a16', 'bis-a19-ie-all']
                      ], 'berlin'
                                                
end

desc "Generate cumulative QLT files, normal section order."
task :generate_cumulative_qlt => [:decide] do |t|
  generate_cumulative @section2qs_answer.keys.sort.map{|section| [section, "bis-#{section}"]}
end

desc "Save question conflicts to prototype potential_conflicts.yaml for conflict_resolution.yaml"
task :save_question_conflicts => [:decide] do |t|

  cfxq2sections = reverse(@section2conflicts)
  q2map_node = {}
  
  YAML.parse_file('conflict_resolution.yaml').root.children.each_slice(2) do |q_node, map_node|
    q2map_node[q_node.value.upcase] = map_node
  end

  $stderr.write("INFO: Generating file potential_conflicts.yaml.\n")
  File.open("potential_conflicts.yaml", 'w') do |f|
    f.write("---\n\n# File template generated with:\n# rake save_question_conflicts\n\n")
    SortedSet.new(cfxq2sections.keys).merge(q2map_node.keys).each do |q|
      f.write("#{q}:\n")
      if cfxq2sections.key? q
        if 1 < cfxq2sections[q].size
          f.write("  am_Besten_nach_Lektion: \"?\"\n")
          candidates = cfxq2sections[q].sort.map{|s| "\"#{s}\""}
          f.write("  Lektionskandidaten: [ #{candidates.join(', ')} ]\n")
          unless @q2theme[q].nil?
            f.write("  Fragenthema: \"#{@q2theme[q].chomp}\"\n")
          end
          unless @q2fam[q].nil?
            similar_qs = @q2fam[q].find_all{|vq| vq != q}.map{|vq| "\"#{vq}\""}
            f.write("  verwandte_Fragen: [ #{similar_qs.join(', ')} ]\n")
          end
        else
          raise "What's that about one conflict section for #{q}?"
        end
      else
        q2map_node[q].children.each_slice(2) do |key_node, value_node|
          f.write("  #{key_node.value}: ")
          if value_node.instance_of? Psych::Nodes::Scalar
            f.write("\"#{value_node.value}\"\n")
          elsif value_node.instance_of? Psych::Nodes::Sequence
            quoted_strings = value_node.children.map{|c| "\"#{c.value}\""}.join(", ")
            f.write("[ #{quoted_strings} ]\n")
          else
            f.write("\"(original value surpressed)\"\n");
            $stderr.write("WARN: Surpressing YAML #{value_node.inspect}\n")
          end
        end
      end
    end
    f.write("\n")
  end
end

desc "List questions never mentioned in the course."
task :list_forgotten_questions => [:decide, :load_all_questions, :load_otherwise_forgotten] do |t|
  
  qs_remembered = Set.new
  @section2qs_answer.values.each{|qs| qs_remembered.merge qs}
  @section2conflicts.values.each{|qs| qs_remembered.merge qs}

  forgotten_qs = @all_qs - @never_asked_questions - qs_remembered

  if 0 < forgotten_qs.size
    $stderr.write("WARN: Following questions unhandled:\n")
    forgotten_qs.each_slice(10) {|slice| $stderr.write("#{slice.join(', ')}\n") }
    File.open("grep.sh", "w", 0755) do |f|
      forgotten_qs.each do |q|
        if q[0] == 'A'
          f.write("grep -li '#{q[1..q.length]}' harvest/a*.html && echo '#{q}\n'\n")
        else
          f.write("grep -li '#{q}' harvest/[bt]*.html && echo '#{q}\n'\n")
        end
      end
    end
    if forgotten_qs.any? { |q| q[0] == 'A' }
      File.open("unmentioned_class_a_template.yaml", 'w') do |f|
        f.write("---\n")
        forgotten_qs.each do |q|
          if q[0] == 'A'
            f.write("# #{q[1..q.length]}: \n")
          end
        end
      end
      $stderr.write("WARN: See also grep.sh and unmentioned_class_a_template.yaml.\n")
    else
      $stderr.write("WARN: See also grep.sh .\n")
    end
  else
    $stderr.write("INFO: Deal with #{qs_remembered.size} of #{@all_qs.size} questions.\n")
  end
end

desc "Reverse unmentioned_class_a.yaml and publish on stdout."
task :reverse_unmentioned_class_a do |t|
  class_a_yaml = YAML.parse_file('unmentioned_class_a.yaml').root
  if class_a_yaml and class_a_yaml.children
    q2sections = {}
    count = 0
    class_a_yaml.children.each_slice(2) do |q_node, sec_node|
      q, section = q_node.value.upcase, sec_node.value.downcase
      raise "double section #{q2sections[q]} vs #{section} from otherwise_forgotten.yaml / unmentioned_class_a,yaml" if q2sections.key? q
      q2sections[q] = [section]
      count += 1
    end
    $stdout.write "# #{count} questions.\n"
    sections2qs = reverse(q2sections)
    sections2qs.keys.sort.each do |section|
      qs = sections2qs[section]
      $stdout.write "#{section}: [#{qs.to_a.sort.join(", ")}]\n"
    end
  else
    raise "ERROR: No material found in unmentioned_class_a.yaml."
  end    
end

#
# Here is stuff connected with the district L slides.
#

desc "Umbrella tasks for L slides (not included: harvest_l)"
task :generate_l_qlt => [:load_all_questions] do |t|
  questionrangeslesson = [] # fromq, toq, lesson
  YAML.safe_load(File.read("l-material.yaml")).each do |mp|
    if mp["kind"] == "Technik A"
      # deal with it later
    elsif mp["questions"] == "-"
      # No questions here?
    else
      ft = /^([TVB][A-Z]\d{3})\s*\-\s*([TVB][A-Z]\d{3})$/.match(mp["questions"])
      if ft
        questionrangeslesson << [ft[1], ft[2], mp['lesson']]
      else
        raise "Could not parse \"#{mp["questions"]}\" in #{mp.inspect}"
      end
    end
  end
  YAML.safe_load(File.read("l-detour.yaml")).each do |dt|
    ft = /^([TVB][A-Z]\d{3})\s*\-\s*([TVB][A-Z]\d{3})$/.match(dt["questions"])
    if ft
      questionrangeslesson << [ft[1], ft[2], dt['lesson']]
    else
      raise "Could not parse \"#{dt["questions"]}\" in #{dt.inspect}"
    end
  end
  questionrangeslesson = questionrangeslesson.sort {|a, b| a[0] <=> b[0]}
  lesson2questionlists = Hash.new do |h, k|
    h[k] = {t: [], b: [], v: []}
    h[k]
  end
  i = 0
  no_lesson = []
  @all_qs.sort.each do |q|
    if q[0..1] == "AT"
    # We don't deal with class A yet.
    elsif @never_asked_questions.include? q
      # Ignore these. They are not asked in the exams.
    else
      repeatcount = 0
      repeat = true
      while(repeat)
        repeatcount += 1
        if repeatcount > 500
          raise "oops endless loop #{q}"
        end
        case questionrangeslesson[i][0] <=> q
        when -1
          # q is above lower bound of range
          case questionrangeslesson[i][1] <=> q
          when -1
            # q is also above upper bound of range.
            # goto next range and try again.
            i += 1
            if questionrangeslesson.size <= i
              # There is no next range.
              i = questionrangeslesson.size - 1
              no_lesson << q
              repeat = false
            end
          when +1
            # q is in the range.
            lesson2questionlists[questionrangeslesson[i][2]][question_kind([q])] << q
            repeat = false
          when 0
            # q is exactly the last question of the range.
            lesson2questionlists[questionrangeslesson[i][2]][question_kind([q])] << q
            repeat = false
          else
            raise "I did not think this could happen."
          end
        when +1
          # q is below lower bound of range,
          # go back one range if that helps any.
          if 0 == i
            # There is no range to go back to.
            no_lesson << q
            repeat = false
          elsif q <= questionrangeslesson[i-1][1]
            # q is at or below upper bound of previous range, so stepping back may help.
            i -= 1
          else
            # q is somehow stuck between ranges.
            no_lesson << q
            repeat = false
          end
        when 0
          # q is exactly equal to the lower bound of range,
          # so clearly contained in the range.
          lesson2questionlists[questionrangeslesson[i][2]][question_kind([q])] << q
          repeat = false
        else
          raise "I did not think this could happen."
        end
      end
    end
  end
  raise "Did not find #{no_lesson.size} questions #{no_lesson.inspect}" unless 0 == no_lesson.size

  previous = {t: Set.new, b: Set.new, v: Set.new, a: Set.new}
  dot_lessons = lesson2questionlists.keys.sort{|a, b| a.to_s <=> b.to_s}.reduce({t: [], b: [], v: []}) do |tbv, dot_lesson|
    key = {"1" => :t, "2" => :b, "3" => :v}[dot_lesson.to_s[0]]
    raise "No key \"#{key}\" in #{tbv.inspect} for \"#{dot_lesson}\"" unless tbv[key]
    tbv[key] << dot_lesson
    tbv
  end
  Dir.mkdir("berlin_2020") unless Dir.exist?("berlin_2020")
  YAML.safe_load(File.read("berlin-l-2020.yaml")).each do |b_day|
    now = {t: Set.new, b: Set.new, v: Set.new}
    [:t, :b, :v].each do |sym|
      max_dots = b_day[sym.to_s].to_s
      while 0 < dot_lessons[sym].size and dot_lessons[sym][0].to_s <= max_dots
        dot_lesson = dot_lessons[sym].slice! 0
        [:t, :b, :v].each do |sym2|
          now[sym2].merge(lesson2questionlists[dot_lesson][sym2])
        end
      end
    end
    l_number = b_day["day"]
    if l_number.size == 2
      l_number = "#{l_number[0]}0#{l_number[1]}"
    end
    [:t, :b, :v].each do |sym3|
      unless previous[sym3].superset? now[sym3] 
        write_qlt("berlin_2020", l_number, now[sym3], l_number, false)
        previous[sym3].merge now[sym3]
      end
      unless now[sym3].superset? previous[sym3]
        write_qlt("berlin_2020", l_number, previous[sym3], "bis-#{l_number}", true)
      end
    end
  end
end

desc "Download the l-material-table and generate l-material.json"
task :harvest_l do |t|
  require 'net/http'
  require 'nokogiri'
  client = Net::HTTP.new('www.darc.de', 443)
  client.use_ssl = true
  resp = client.request_get("/der-club/distrikte/l/referat-fuer-aus-und-weiterbildung/",
                                {
                                  "user-agent" => "DJ3EI's QLT spider https://github.com/aknrdureegaesr/afukurs-selbstkontrolle"
                                })
  raise "Could not grab table: #{response.inspect}" unless resp.is_a?(Net::HTTPSuccess)
  raise "Cannot parse #{respone['content-type']}" unless /^text\/html(;.*)?$/i =~ resp['content-type']
  page = Nokogiri::HTML(resp.body)
  File.open("l-material.yaml", "w") do |out|
    [
      ["c261949", "Technik A"],
      ["c259595", "Technik E"],
      ["c259883", "Betrieb"],
      ["c260155", "Vorschriften"]
    ].each do |root_div, kind|
      out.write("\n")
      page.xpath("//div[@id=\"#{root_div}\"]//table/tr").each do |tr|
        tds = tr.xpath("./td")
        chapter = tds[0].inner_text
        if chapter == "Kapitel"
          # Life would have been easier
          # had they choosen to use th for what it's for.          
        elsif chapter.match?(/^\s*$/)
          # CSS? Yes? What? Not here yet? Ah. I see.
        else
          out.write("-\n")
          out.write("  kind:      \"#{kind}\"\n")
          out.write("  lesson:    \"#{chapter}\"\n")
          out.write("  questions: \"#{tds[1].inner_text}\"\n")
          out.write("  what:      \"#{tds[3].inner_text}\"\n")
          out.write("  whodoneit: \"#{tds[2].inner_text}\"\n")
          out.write("  duration:  \"#{tds[4].inner_text}\"\n\n")
        end
      end
    end
  end
end

