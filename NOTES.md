**How to Define and Use Rake Tasks**

Building a Rake task is easy, since Rake is already available to us as a part of Ruby. 
All we need to do is create a file in the top level of our directory called Rakefile. Here we define our task:

<task :hello do
  # the code we want to be executed by this task
end>

We define tasks with task + name of task as a symbol + a block that contains the code we want to execute.

If you open up the Rakefile in this directory, you'll see our :hello task:

<task :hello do
  puts "hello from Rake!"
end>

Now, in your terminal in the directory of this project, type:

rake hello

You should see the following outputted to your terminal:

# => hello from Rake!
""
-------------------------------------------------------------------------------------------------------------------------------------------

***Describing our Tasks for rake -T***

Rake comes with a handy command, rake -T, that we can run in the terminal to view a list of available Rake tasks and their descriptions. 
In order for <rake -T> to work though, we need to give our Rake tasks descriptions. Let's give our hello task a description now:

<desc 'outputs hello to the terminal'
task :hello do
  puts "hello from Rake!"
end>
Now, if we run rake -T in the terminal, we should see the following:

# rake hello       => outputs hello to the terminal

So handy!
""
--------------------------------------------------------------------------------------------------------------------------------------------
***Namespacing Rake Tasks***

It is possible to namespace your Rake tasks. What does "namespace" mean? A namespace is really just a way to group or contain something,
in this case our Rake tasks. So, we might namespace a series of greeting Rake tasks, like hello, above, under the greeting heading.

Let's take a look at namespacing now. Let's say we create another greeting-type Rake task, hola:

<desc 'outputs hola to the terminal'
task :hola do
  puts "hola de Rake!"
end>

Now, let's namespace both hello and hola under the greeting heading:

<namespace :greeting do
desc 'outputs hello to the terminal'
  task :hello do
    puts "hello from Rake!"
  end
 
  <desc 'outputs hola to the terminal'
  task :hola do
    puts "hola de Rake!"
  end
end>

Now, to use either of our Rake tasks, we use the following syntax:

# rake greeting:hello
# hello from Rake!
 
# rake greeting:hola
# hola de Rake!
""
--------------------------------------------------------------------------------------------------------------------------------------------
**Common Rake Tasks**

***rake db:migrate***

One common pattern you'll soon become familiar with is the pattern of writing code that creates database tables and then "migrating" that code using a rake task.
Our Student class currently has a #create_table method, so let's use that method to build out our own migrate Rake task.

We'll namespace this task under the db heading. This namespace will contain a few common database-related tasks.
We'll call this task migrate, because it is a convention to say we are "migrating" our database by applying SQL statements that alter that database.

<namespace :db do>
  desc 'migrate changes to your database'
  task :migrate => :environment do
    Student.create_table
  end
<end>

But, if we run rake db:migrate now, we're going to hit an error.

***Task Dependency***

You might be wondering what is happening with this snippet:

task :migrate => :environment do
...

This creates a task dependency. Since our Student.create_table code would require access to the config/environment.rb file 
(which is where the student class and database are loaded), we need to give our task access to this file. 

In order to do that, we need to define yet another Rake task that we can tell to run before the migrate task is run.

Let's check out that environment task:

<!-- # in Rakefile -->
 
<task :environment do
  require_relative './config/environment'
end>

After adding our environment task, running rake db:migrate should create our students table.

***rake db:seed***

Another task you will become familiar with is the seed task. This task is responsible for "seeding" our database with some dummy data.

The conventional way to seed your database is to have a file in the db directory, db/seeds.rb, that contains some code to create instances of your class.

Then, we define a rake task that executes the code in this file. This task will also be namespaced under db:

<namespace :db do>
 
  ...
 
  <desc 'seed the database with some dummy data'
  task :seed do
    require_relative './db/seeds.rb'
  end
end>

Now, if we run rake db:seed in our terminal (provided we have already run rake db:migrate to create the database table), we will insert five records into the database.

If only there was some way to interact with our class and database without having to run our entire program...

Well, we can build a Rake task that will load up a Pry console for us.