# coding: utf-8
require "yaml"

task :default => [:generate_qlt, :generate_cumulative_qlt, :list_forgotten_questions]

# Maps from sections (e.g.: 'bv07') to enumerables of questions (e.g.: ['VB101', 'VB102']).
# From harvest.json, harvested from web site course material via subdirectory `harvest`:
@section2qs_harvest = {}
# From otherwise_forgotten.yaml, which is manually crafted:
@section2qs_from_otherwise_forgotten = {}

# For each question, the family of questions considered related to
# each other.  This data historically comes from previous courses organized
# independently (rather different split-up in sections), technically from
# file "learn_batches.yaml".
@q2fam = {}
# For each question, the general theme of that question and all in the same family.
@q2theme = {}

# Questions that could belong to a section, as they are related.
# Generated here from the above data.
# May contain the same question in several sections.
@section2qs_closure = {}

# For some of the questions, advice about the course section.
# From conflict_resolution.yaml, which is edited manually.
@q2section_resolved = {}

# The final answer, determined by mixing the above information.
# May not contain all sections, answers for individual sections
# may be incomplete.
@section2qs_answer = {}
# For each section that cannot yet be generated,
# the questions that need to be added to conflict_resolution.yaml.
# Either you can rely on the answer of @section2qs_answer[section],
# or @section2conflicts.key? section
@section2conflicts = {}

# All questions known. (Presently without technics class A questions.)
# This is for double-checking.
@all_qs = Set.new

desc "load harvest result from harvest.json" # loads @section2qs_harvest
task :load_harvest do |t|
  YAML.parse_file('harvest.json').root.children[1].children.each_slice(2) do |section_node, questions_node|
    section = section_node.value.downcase
    @section2qs_harvest[section] = Set.new unless @section2qs_harvest.key? section
    @section2qs_harvest[section].merge questions_node.children.map { |qnode| qnode.value.upcase }
  end
  YAML.parse_file('te_harvest.json').root.children.each do |q2sec|
    q2sec.children.each_slice(2) do |q_node, section_node|
      q = q_node.value.upcase
      section = section_node.value.downcase
      @section2qs_harvest[section] = Set.new unless @section2qs_harvest.key? section
      @section2qs_harvest[section] << q
    end
  end
end

# Take a map that maps some a values to enumerables of b values,
# and reverse it (to a map that maps individual b values to enumerables of a values).
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

desc "load exceptionals from otherwise_forgotten.yaml" # into @section2qs_from_otherwise_forgotten
task :load_otherwise_forgotten do |t|
  q2sections = {}
  YAML.parse_file('otherwise_forgotten.yaml').root.children.each_slice(2) do |q_node, sec_node|
    q, section = q_node.value.upcase, sec_node.value.downcase
    raise "double section #{q2sections[q]} vs #{section} from otherwise_forgotten.yaml" if q2sections.key? q
    q2sections[q] = [section]
  end
  @section2qs_from_otherwise_forgotten = reverse q2sections
end

desc "load learn-together families from learn_batches.yaml" # into q2fam
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
        $stderr.print "WARN: In no family: #{q}\n"
        @section2qs_closure[section] << q
      end
    end
  end
end

desc "Load lists of all questions"
task :load_all_questions do |t|
  # ['b', 'te', 'v', 'ta'].each do |subject|
  ['b', 'te', 'v'].each do |subject|
    File.open("alle_#{subject}.qlt") do |f|
      @all_qs.merge f.each_line.drop(2).map{|qline| qline.chomp}.to_a
    end
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

  all_qs.each do |q|
    # First try.
    section = @q2section_resolved[q]
    
    sections_of = q2sections_otherwise_forgotten[q]
    if section.nil?
      # Second try.
      section = sections_of.first unless sections_of.nil?
    else
      # First and second try should not conflict in results.
      unless sections_of.nil? or section == sections_of.first
        raise "Conflict #{q}: #{section} in conflict_resolution.yaml vs. #{sections_of.first} in otherwise_forgotten.yaml"
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

def question_kind(section, qs)
  (qs.first)[0].downcase.to_sym
end

def write_qlt(dir, section, qs, upto)
  subject = question_kind(section, qs)
  fach = {t: 'Technik', b: 'Betriebskunde', v: 'Vorschriften'}[subject]
  raise "Subject not implemented: #{subject}" if fach.nil?
  filename = dir + "/#{{t: 'te', b: 'be', v: 'vo'}[subject]}-#{upto ? 'upto-' : ''}#{section}.qlt".upcase
  $stderr.write("INFO: Writing #{filename}\n")
  if (qs.size < 3)
    $stderr.write "WARN: Cannot generate qlt file #{filename} for #{section}: Have only #{qs.size} of >= 3 questions, namely  #{qs.inspect}.\n"
  else
    show = (34 < qs.size ? 34 : qs.size)
    time = (show / 34.0 * 60 + 0.8).to_i
    File.open("#{filename}", 'w') do |f|
      f.write("# #{fach}fragen #{upto ? 'bis einschl. ' : ''}Lektion #{section}\r\n")
      f.write("#{{t: 'TE', b: 'BE', v: 'VE'}[subject]}=#{show}\r\n")
      f.write("Zeit=#{time}\r\n")
      f.write(qs.sort.join("\r\n"))
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
        $stderr.write "WARN: Cannot write #{question_kind(section,qs_one_kind)} for #{section}; please decide [#{kind_letters2conflicts[kind_letter].sort.join(', ')}]\n"
      else
        write_qlt "data", section, qs_one_kind, false
      end
    end
  end
end

task :generate_cumulative_qlt => [:decide] do |t|

  kind2cumulated = {}
  kind_letters_with_conflicts = Set.new

  @section2qs_answer.keys.find_all{|section| not @section2conflicts.key? section}.sort.each do |section|
    conflicts = @section2conflicts[section]
    unless conflicts.nil?
      conflicts.sort.chunk{|q| q[0]}.each do |kind_letter, conflicts_this_kind|
        kind_letters_with_conflicts << kind_letter
      end
    end
    qs = @section2qs_answer[section]
    qs.sort.chunk{|q| q[0]}.each do |kind_letter, qs_one_kind|
      kind = question_kind(section, qs)
      kind2cumulated[kind] = Set.new unless kind2cumulated.key? kind
      unless kind_letters_with_conflicts.include? kind_letter
        write_qlt "data", section, kind2cumulated[kind].merge(qs), true
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

desc "List question conflicts in prototype potential_conflicts.yaml for conflict_resolution.yaml"
task :list_question_conflicts => [:decide] do |t|

  cfxq2sections = reverse(@section2conflicts)
  q2map_node = {}
  
  YAML.parse_file('conflict_resolution.yaml').root.children.each_slice(2) do |q_node, map_node|
    q2map_node[q_node.value.upcase] = map_node
  end

  $stderr.write("INFO: Generating file potential_conflicts.yaml.\n")
  File.open("potential_conflicts.yaml", 'w') do |f|
    f.write("---\n\n# File started with:\n# rake list_question_conflicts\n\n")
    SortedSet.new(cfxq2sections.keys).merge(q2map_node.keys).each do |q|
      f.write("#{q}:\n")
      if cfxq2sections.key? q
        if 1 < cfxq2sections[q].size
          f.write("  am_Besten_nach_Lektion: \"?\"\n")
          candidates = cfxq2sections[q].sort.map{|s| "\"#{s}\""}
          f.write("  Lektionskandidaten: [ #{candidates.join(', ')} ]\n")
          f.write("  Fragenthema: \"#{@q2theme[q].chomp}\"\n")
          similar_qs = @q2fam[q].find_all{|vq| vq != q}.map{|vq| "\"#{vq}\""}
          f.write("  verwandte_Fragen: [ #{similar_qs.join(', ')} ]\n")
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

  forgotten_qs = @all_qs - qs_remembered

  if 0 < forgotten_qs.size
    $stderr.write("WARN: Following questions unhandled:\n")
    forgotten_qs.each_slice(10) {|slice| $stderr.write("#{slice.join(', ')}\n") }
    File.open("grep.sh", "w", 0755) do |f|
      forgotten_qs.each do |q|
        f.write("grep -li '#{q}' harvest/*.html && echo '#{q}\n'\n")
      end
      $stderr.write("WARN: See also grep.sh.\n")
    end
  else
    $stderr.write("INFO: Deal with #{qs_remembered.size} of #{all_qs.size} questions.\n")
  end
end

