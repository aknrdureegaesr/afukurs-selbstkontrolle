require "yaml"

task :default => [:generate_qlt, :generate_cumulative_qlt]

@section2qs_harvest = {}
@q2section_harvest = {}
@q2fam = {}
@q2theme = {}
@q2sections = {}
@q2section_resolved = {}
@section2qs_closure = {}
@sym2section2qs = {t: {}, b: {}, v:{} }

desc "load learn-together families from learn_batches.yaml"
task :load_families do |t|
  YAML.parse_file('learn_batches.yaml').root.children.each do |family|
    thema = family.children[1 + family.children.find_index {|s| "Thema" == s.value}].value
    fragen = family.children[1 + family.children.find_index {|s| "Fragen" == s.value}].value
    v = []
    fragen.split.each do |frage|
      if /^\w\w\d\d\d$/ =~ frage
        v << frage.upcase
      elsif /^(\w\w)(\d\d\d)\-(\w\w)(\d\d\d)$/ =~ frage
        raise "I am confused about \"#{frage}\"" unless $1 == $3
        ($2.to_i .. $4.to_i).each { |i| v << "#{$1.upcase}#{i}" }
      else
        raise "I am confused about \"#{fragen}\" -> \"#{frage}\""
      end
    end
    v.each do |frage|
      raise "#{v.inspect} vs. #{@q2fam[frage].inspect} for #{frage}" if @q2fam.key? frage
      @q2fam[frage] = v
      @q2theme[frage] = thema
    end
  end
end


desc "load harvest result from harvest.json"
task :load_harvest do |t|
  # JSON is a subset of YAML.
  key = nil
  YAML.parse_file('harvest.json').root.children[1].children.each do |node|
    if node.instance_of? Psych::Nodes::Scalar
      key = node.value
    elsif node.instance_of? Psych::Nodes::Sequence
      value = node.children.map do |qnode|
        raise "Did not expect #{qnode.inspect}" unless qnode.instance_of? Psych::Nodes::Scalar
        qnode.value.upcase
      end
      raise "Expecting key to be set." if key.nil?
      @section2qs_harvest[key] = Set.new(value.to_a)
      key = nil
    else
      raise "Did not expect #{node.inspect}"
    end
  end
  @section2qs_harvest.keys.sort.each do |section|
    @section2qs_harvest[section].each do |q|
      if @q2section_harvest.key? q
        $stderr.write "WARN: #{q} found in both #{@q2section_harvest[q]} and #{section}\n"
      else
        @q2section_harvest[q] = section
      end
    end
  end
end

desc "calculate question closure for each session"
task :section_closure => [:load_harvest, :load_families] do |t|
  @section2qs_harvest.each do |section, harvest_set|
    @section2qs_closure[section] = Set.new()
    harvest_set.each do |q|
      if @q2fam.key? q
        @q2fam[q].each do |other_q|
          @section2qs_closure[section] << other_q
        end
      else
        $stderr.print "WARN: In no family: #{q}\n"
        @section2qs_closure[section] << q
      end
    end
  end
end

def write_qlt(subject, section, qs)
  fach = {t: 'Technik', b: 'Betriebskunde', v: 'Vorschriften'}[subject]
  raise "No such subject: #{subject}" if fach.nil?
  filename = "#{{t: 'te', b: 'be', v: 'vo'}[subject]}-#{section}.qlt".upcase
  $stderr.write("INFO: Writing #{filename}\n")
  raise "Need at least three questions for #{filename}, have #{qs.size}, namely  #{qs.inspect}" if(qs.size < 3)
  show = (34 < qs.size ? 34 : qs.size)
  time = (show / 34.0 * 60 + 0.8).to_i
  File.open(filename, 'w') do |f|
    f.write("# #{fach}fragen Lektion #{section}\r\n")
    f.write("#{{t: 'TE', b: 'BE', v: 'VE'}[subject]}=#{show}\r\n")
    f.write("Zeit=#{time}\r\n")
    f.write(qs.sort.join("\r\n"))
    f.write("\r\n")
  end
end

desc "Generate the .qlt - files (as far as possible)"
task :generate_qlt => [:decide] do |t|
  @sym2section2qs.keys.sort.each do |sym|
    @sym2section2qs[sym].keys.sort.each do |section|
      qs = @sym2section2qs[sym][section]
      write_qlt(sym, section, qs) unless qs == :no_data || qs.empty?
    end
  end
end

task :generate_cumulative_qlt => [:decide] do |t|
  @sym2section2qs.keys.sort.each do |sym|
    cumulative = Set.new()
    @sym2section2qs[sym].keys.sort.each do |section|
      if cumulative != :no_data
        qs = @sym2section2qs[sym][section]
        if qs != :no_data
          if not qs.empty?
            cumulative.merge qs
            if !qs.empty? and qs.size < cumulative.size
              write_qlt(sym, "upto-#{section}", cumulative.sort.to_a)
            end
          end
        else
          cumulative = :no_data
        end
      end
    end
  end
end

task :decide => [ :find_question_conflicts, :read_conflict_resolution ] do |t|
  @section2qs_closure.keys.each do |section|
    qs = Set.new(@q2section_resolved.keys.find_all{|q| @q2section_resolved[q] == section})
    @section2qs_harvest[section].each do |hq|
      qs << hq unless @q2section_resolved.key? hq
    end
    need_to_resolve = Set.new
    @section2qs_closure[section].each do |q|
      if not @q2sections.key? q
        raise "WTF? #{q} in #{section} (transitive) but...\n" unless @q2section_harvest.key? q
      elsif 1 == @q2sections[q].size
        qs << q
      elsif @q2section_resolved.key? q
        # ok.
      elsif 
        need_to_resolve << q
      end
    end
    subject2qs = {}
    qs.sort.chunk{|q| q[0].downcase.to_sym}.each do |sym, qs|
      subject2qs[sym] = qs
    end
    subject2qs.keys.sort.each do |sym|
      qs = subject2qs[sym]
      undecided_ones = need_to_resolve.find_all {|q_undecided| q_undecided[0].downcase.to_sym == sym}
      if undecided_ones.empty?
        @sym2section2qs[sym][section] = qs
      else
        @sym2section2qs[sym][section] = :no_data
        $stderr.write("Cannot generate #{section}.qlt for #{sym}, need to fix in conflict_resolution.yaml #{undecided_ones.sort.join(', ')}\n")
      end
    end
  end  
end

desc "Read conflict_resolution.yaml"
task :read_conflict_resolution do |t|
  YAML.parse_file('conflict_resolution.yaml').root.children.each_slice(2) do |q, m|
    m.children.each_slice(2) do |key, value|
      if key.value == 'am_Besten_nach_Lektion' and value.value =~ /^\w\w?\d\d$/
        @q2section_resolved[q.value] = value.value
      end
    end
  end
end

desc "Find question conflicts"
task :find_question_conflicts => [:section_closure] do |t|
  @section2qs_closure.keys.each do |section|
    @section2qs_closure[section].each do |q|
      if @q2section_harvest.key? q
        # All is well.
      elsif @q2sections.key? q
        @q2sections[q] << section
      else
        @q2sections[q] = [section]
      end
    end
  end
end

desc "List question conflicts in prototype potential_conflicts.yaml for conflict_resolution.yaml"
task :list_question_conflicts => [:find_question_conflicts] do |t|
  File.open("potential_conflicts.yaml", 'w') do |f|
    f.write("---\n")
    @q2sections.keys.sort.each do |q|
      if 1 < @q2sections[q].size
        f.write("#{q}:\n")
        f.write("  am_Besten_nach_Lektion: \"?\"\n")
        candidates = @q2sections[q].map{|s| "\"#{s}\""}
        f.write("  Lektionskandidaten: [ #{candidates.join(', ')} ]\n")
        f.write("  Fragenthema: \"#{@q2theme[q].chomp}\"\n")
        similar_qs = @q2fam[q].find_all{|vq| vq != q}.map{|vq| "\"#{vq}\""}
        f.write("  verwandte_Fragen: [ #{similar_qs.join(', ')} ]\n")
      end
    end
    f.write("\n")
  end
end
