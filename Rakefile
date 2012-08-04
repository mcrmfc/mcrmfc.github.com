require 'rubygems'
require 'bundler/setup'
require 'ftools'
require 'jekyll'
$: << File.dirname( __FILE__)
require '_plugins/generate_sitemap'

task :generate_sitemap do
  puts 'Generating sitemap..'
  include Jekyll::Filters
  options = Jekyll.configuration({})
  site = Jekyll::Site.new(options)
  site.read
  Jekyll::SitemapGenerator.new.generate(site)
  puts 'Done.'
end

task :generate_tags do
  puts 'Generating tags...'
  include Jekyll::Filters

  options = Jekyll.configuration({})
  site = Jekyll::Site.new(options)
  site.read_posts('')

  html =<<-HTML
---
layout: default
title: Tags
---

<ul class="breadcrumb">
  <li class="home"><a href="/">Home</a></li>
  <li>Tags</li>
</ul>

<h2>Tags</h2>

  HTML

  site.categories.sort.each do |category, posts|
    html << <<-HTML
      <h3 id="#{category}">#{category}</h3>
    HTML

    html << '<ul class="posts">'
    posts.each do |post|
      post_data = post.to_liquid
      html << <<-HTML
          <li>
            <div>#{date_to_string post.date}</div>
            <a href="#{post.url}">#{post_data['title']}</a>
          </li>
      HTML
    end
    html << '</ul>'
  end

  File.open('tags.html', 'w+') do |file|
    file.puts html
  end

  puts 'Done.'
end

task :tag_cloud do
  puts 'Generating tag cloud...'
  require 'rubygems'
  require 'jekyll'
  include Jekyll::Filters

  options = Jekyll.configuration({})
  site = Jekyll::Site.new(options)
  site.read_posts('')

  html = ''
  max_count = site.categories.map{|t,p| p.count}.max
  site.categories.sort.each do |tag, posts|
    s = posts.count
    font_size = ((20 - 10.0*(max_count-s)/max_count)*2).to_i/2.0
    html << "<a href=\"/tags/#{tag}\" title=\"Postings tagged #{tag}\" style=\"font-size: #{font_size}px; line-height:#{font_size}px\">#{tag}</a> "
  end
  File.open('_includes/tag_cloud.html', 'w+') do |file|
    file.puts html
  end
  puts 'Done.'
end

task :prepare_release do
 puts 'Preparing to release site...'
  Rake::Task[:generate_tags].invoke
  Rake::Task[:tag_cloud].invoke
  Rake::Task[:generate_sitemap].invoke
end


task :default => :prepare_release
