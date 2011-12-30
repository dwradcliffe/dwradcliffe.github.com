---
layout: post
title: Importing csv data to rails using rake
---

Often part of building a website is being able to import static data into the database. Usually the data is in csv format. Here's a quick example of how to make this work.

## Files

* `db/teams.csv` - I like to drop my data here. If its a huge file, you might want to add it to your `.gitignore` file to keep it out of git.

* `lib/tasks/import.rake` - This is where we will define our rake task.

## Task

    require 'csv'

    desc "Import teams from csv file"
    task :import => [:environment] do

      file = "db/teams.csv"
      
      CSV.foreach(file, :headers => true) do |row|
        Team.create {
          :name => row[1],
          :league => row[2],
          :some_other_data => row[4]
        }
      end
  

## Notes

Make sure you carefully match up the rows. Remember the array is base 0.
If your file doesn't have a header row, remove set :headers to false in the foreach.

## Run!

From the terminal, change to the app directory and run `rake import` or `bundle exec rake import`.

## Resources

CSV library docs - [http://ruby-doc.org/stdlib-1.9.2/libdoc/csv/rdoc/CSV.html](http://ruby-doc.org/stdlib-1.9.2/libdoc/csv/rdoc/CSV.html)

Rake docs - [http://rake.rubyforge.org/](http://rake.rubyforge.org/)