require 'rubygems'
require 'Hpricot'

# if true a partial generated by _partial.rb is deleted after the add
CLEANUP_AFTER_GENERATION = false
OUTPUT_DIRECTORY = './site'

# This task generate all needed directories
task :generate_directory_structure do
  ['layouts', 'modules', 'pages', 'pages/commons', 'pages/index', OUTPUT_DIRECTORY].each do |dir|
    system("mkdir #{dir}")
  end
  puts "Done"
end

task :build_site do
  print "Pages to generate:"
  pages = Dir.entries('./pages').reject { |item| item == '.' || item == '..' || item == 'commons' }
  print ' \'' + pages.join('\', \'') + '\''
  puts ""

  pages.each do |page|
    puts "\n*** Page '#{page}' ***"
    page_path = "./pages/#{page}"
    # Get main source
    puts "\tGetting main source for page..."
    if File.exist?("./pages/#{page}/pp-main.html")
      main_code = Hpricot(open(page_path + "/pp-main.html"))
      # Get layout needed for this page
      if main_code.search('pp-layout').any?
        layout = main_code.search('pp-layout').first.inner_html
        puts "\tUsing layout: '#{layout}'"
      else
        puts "\tUsing layout: none!"
      end
      if (layout && File.exist?("./layouts/#{layout}.html")) || !layout
        # Get partials needed from the main source
        partials = main_code.search('pp-include').map { |i| i.inner_html }
        # Checking existence of all needed partials
        source = main_code.html
        # Looking for partial follows this gerarchy:
        # - _partial.rb in page's directory
        # - _partial.rb in commons directory
        # - modules
        # - _partial.html in page's directory
        # - _partial.html in commons directory
        if partials.any?
          puts "\tPartials needed for page:"
          partials.each do |p|
            print "\t\t#{p} ... "
            if File.exist?("./pages/#{page}" + '/_' + p + '.rb')
              # This is a ruby partial in the page's directory
              external_source = eval(File.read("./pages/#{page}" + '/_' + p + '.rb'))
              if external_source.nil?
                # nil returned ... it contains just a method? => try to load a new generated html partial?
                source.gsub!("<pp-include>#{p}</pp-include>", File.read("./pages/#{page}" + '/_' + p + '.html'))
                print "Generated + Added (in page dir)\n"
                if CLEANUP_AFTER_GENERATION
                  File.delete("./pages/#{page}" + '/_' + p + '.html')
                  puts "./pages/#{page}/_#{p}.html NOT DELETED" if File.exist?("./pages/#{page}" + '/_' + p + '.html')
                end
              else
                # evaluation returned a string ... just insert it in the final source
                source.gsub!("<pp-include>#{p}</pp-include>", external_source)
                print "Executed + Added (in page dir)\n"
              end
            elsif File.exist?("./pages/commons" + '/_' + p + '.rb')
              # This is a ruby partial in commons directory
              external_source = eval(File.read("./pages/commons" + '/_' + p + '.rb'))
              if external_source.nil?
                # nil returned ... it contains just a method? => try to load a new generated html partial?
                source.gsub!("<pp-include>#{p}</pp-include>", File.read("./pages/commons" + '/_' + p + '.html'))
                print "Generated + Added (in commons dir)\n"
                if CLEANUP_AFTER_GENERATION
                  File.delete("./pages/commons" + '/_' + p + '.html')
                  puts "./pages/commons/_#{p}.html NOT DELETED" if File.exist?("./pages/commons" + '/_' + p + '.html')
                end
              else
                # evaluation returned a string ... just insert it in the final source
                source.gsub!("<pp-include>#{p}</pp-include>", external_source)
                print "Executed + Added (in commons dir)\n"
              end
            elsif File.exist?("./modules/#{p}/pp-main.rb")
              # This is a ruby partial in modules directory
              source_from_module = eval(File.read("./modules/#{p}/pp-main.rb"))
              if source_from_module.nil?
                # nil returned ... it contains just a method? => try to load a new generated html partial?
                source.gsub!("<pp-include>#{p}</pp-include>", File.read("./modules/#{p}/_#{p}.html"))
                print "Generated + Added (from module)\n"
                if CLEANUP_AFTER_GENERATION
                  File.delete("./modules/#{p}/_#{p}.html")
                  puts "./modules/#{p}/_#{p}.html NOT DELETED" if File.exist?("./modules/#{p}/_#{p}.html")
                end
              else
                # evaluation returned a string ... just insert it in the final source
                source.gsub!("<pp-include>#{p}</pp-include>", source_from_module)
                print "Executed + Added (from module)\n"
              end
            elsif File.exist?("./pages/#{page}" + '/_' + p + '.html')
              # This is an html partial in page's directory
              source.gsub!("<pp-include>#{p}</pp-include>", File.read("./pages/#{page}" + '/_' + p + '.html'))
              print "Added (from page dir)\n"
            elsif File.exist?("./pages/commons" + '/_' + p + '.html')
              # This is an html partial in commons directory
              source.gsub!("<pp-include>#{p}</pp-include>", File.read("./pages/commons" + '/_' + p + '.html'))
              print "Added (from commons)\n"
            else
              puts "ERROR - Missing partial (#{p})"
            end
          end
        else
          puts "\tPartials needed for '#{page}' page: none!"
        end
      else
        puts "\tUnable to find ./layouts/#{layout}.html, this page won't be generated!"
      end
      # Create a new file html (future final)
      f = File.new("#{OUTPUT_DIRECTORY}/#{page}.html", 'w+')
      if layout
        source = source.gsub("<pp-layout>#{layout}</pp-layout>", '') if layout
        f << File.read("./layouts/#{layout}.html").gsub("<pp-content />", source)
      else
        f << source
      end
      f.close
    else
      puts "\tUnable to find ./pages/#{page}/pp-main.html, this page won't be generated!"
    end
  end
end

task :clean do
  # This script delete every page, partial, etc
  Dir.glob('./pages/*/*').each do |f|
    if File.delete(f)
      puts "#{f}... DELETED"
    else
      puts "#{f}... NOT DELETED"
    end
  end
  Dir.glob('./site/*').each do |f|
    if File.delete(f)
      puts "#{f}... DELETED"
    else
      puts "#{f}... NOT DELETED"
    end
  end
end