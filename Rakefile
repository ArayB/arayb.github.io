# frozen_string_literal: true

task :draft do
  puts 'Provide filename:'
  puts '(Filename becomes url so use hyphenated words e.g. my-awesome-blog)'
  filename = STDIN.gets.chomp

  puts 'Provide blog title:'
  title = STDIN.gets.chomp

  puts 'Provide categories:'
  puts '(Categories is list of words separated by a space e.g ruby rails lego)'
  cats = STDIN.gets.chomp

  content = [
    '---',
    'layout: post',
    "title: #{title}",
    "date: #{Time.now}",
    "categories: #{cats}",
    '---'
  ]
  File.open("_drafts/#{filename}.md", 'w') { |f| f << content.join("\n") }
end
