# I Love Active Record: Models
## Learning Goals
- Use AR _models_ to perform CRUD tasks on database records
- Joyfully embrace the _Rails Console_ 
- Begin exploring the AR _query interface_

## Active Record as a DSL
Given a table in our database called _students_ that has this structure:

| id | name     | pie         | birthday   |
|:--:|:--------:|:-----------:|:----------:|
|  1 | Sheena   | peach       | 08-13-1977 |
|  2 | Rosa     | apple       | 05-01-2008 |
|  3 | Raquel   | apple       | 03-27-2003 |

We should have a corresponding file called `app/models/student.rb`. It will look like this:

```ruby
class Student < ActiveRecord::Base

end
```

This is our Student _model_, and it is connected to a table in the database called _students_. We don't have to explicitly tell Rails about that connection; by following the naming conventions (thank you, Inflector!), Rails knows which database table to use with which _model_. By doing this you'll also have the ability to map the columns of each row in that table with the attributes of the instances of your model.

For right now, our _models_ will likely be empty Ruby classes. The inherited class (`ActiveRecord::Base`) provides a tremendous amount of functionality. As we move forward in the curriculum, we will begin filling out our models with additional functionality.

## Interacting with models in the _Rails console_
Rails provides a REPL (similar to irb and pry) that pre-loads all the application information when started. It's one of the most useful tools I've encountered. Let's spend some time exploring it. From your application root, run `$ rails console`:

```bash
jeremy@iridium ~/sandbox/ar-practice
 ❤️  :: rails console
Running via Spring preloader in process 48975
Loading development environment (Rails 4.2.6)
2.3.0 :001 >
```

Now we're in the _console_, from here we can do all the stuff that irb and pry provide, plus have access to all of our Rails models. Use the console to create an __instance__ of our Student _model_:

```ruby
2.3.0 :001 > s = Student.new
 => #<Student id: nil, name: nil, pie: nil, motto: nil, created_at: nil, updated_at: nil>
2.3.0 :002 > s.attributes
 => {"id"=>nil, "name"=>nil, "pie"=>nil, "motto"=>nil, "created_at"=>nil, "updated_at"=>nil}
2.3.0 :003 >
```

We can assign data to the instance using plain Ruby:
```ruby
2.3.0 :003 > s.name = "Libby"
 => "Libby"
2.3.0 :004 > s.pie = "Apple"
 => "Apple"
2.3.0 :006 > s.attributes
 => {"id"=>nil, "name"=>"Libby", "pie"=>"Apple", "motto"=>nil, "created_at"=>nil, "updated_at"=>nil}
```

When we are happy with the data in our _model_, we can write it to the database using a single command:

```ruby
2.3.0 :007 > s.save
   (0.3ms)  begin transaction
  SQL (1.8ms)  INSERT INTO "students" ("name", "pie", "created_at", "updated_at") VALUES (?, ?, ?, ?)  [["name", "Libby"], ["pie", "Apple"], ["created_at", "2016-04-17 18:17:52.649965"], ["updated_at", "2016-04-17 18:17:52.649965"]]
   (1.5ms)  commit transaction
 => true
2.3.0 :008 >
```

__Question: ZOMG what just happened?__
The Active Record DSL translates our Ruby code into SQL, and then executes it.

```ruby
Student.where(pie: "apple").order('birthday DESC')
```
becomes

```sql
SELECT * FROM "students" WHERE "pie" = "apple"  ORDER BY birthday DESC;
```

## AR Class Methods: Finding and Organizing Records
ActiveRecord class methods are generally for locating & reading rows from the database:

```ruby
# return a collection of all rows in the students table
students = Student.all

# return the first student (lowest id)
student = Student.first

# return the first student (lowest id) named Libby
libby = Student.find_by(name: 'Libby')

# return students in a specific order
alphabetical_students = Student.order(:name)

# the AR DSL is structured to use chaining to create more specific queries
# here we find all students that prefer apply pie
# *and* return them in reverse alphabetical order
team_apple = Student.where(pie: "Apple").order(name: :desc)

# provide compount conditions to do more specific queries
rosa_and_apple = Student.where(name: "Rosa", pie: "apple")

# use `not` to exclude values from results
not_apple = Student.where.not(pie: "apple")
rosa_but_not_apple = Student.where(name: "Rosa").where.not(pie: "apple")

# we can use `find` to pull a bunch of records at once too
ids = [1,2,3]
Student.find(ids) # returns an array of Student records

# ...but I think using `where` to do this is a more semantic
Student.where(id: ids)
```

### Parameter Binding with Active Record
We learned last week about using _parameter binding_ to (most importantly) make queries more secure (google research: _sql injection_), and to improve the reusability of queries. This idea is equally important in Rails, and AR provides it in a similar manner.

If you write code using the `where` syntax above (like `Student.where(name: "Rosa")`), Active Record _does parameter binding automatically_. Active Record also provides manual parameter binding similar to how we did it with SQLite:

```ruby
# using '?'
Student.where("name = ?", "Rosa")

# using named parameters
Student.where("name = :name AND pie = :pie", name: "Rosa", pie: "apple")
```

### Using AR to modify rows in the database
Instances of Active Record models can be created from a hash, a block or have their attributes manually set after creation. Once an instance has its data attributes defined, it can be saved to the database, making a persistant record of that data.

#### `new` vs. `create`
**.new** returns a new object of that class, but doesn't save it to the database.

```ruby
# `new` with preset data
desi = Student.new(name: "Desi", pie: "blueberry")
desi.id # => nil
```

```ruby
# an "empty" new student
student = Student.new
# assigning data attributes
student.name = "Desi"
student.pie = "blueberry"
```

Meanwhile, `create` will initialize a new instance of the model *and* save it to the database:
```ruby
desi = Student.create(name: "Desi", pie: "blueberry")
desi.id #=> 5
```

#### Editing an existing row
Once we've located a record using `find`, `where`, or other class methods, we can modify the data in the __instances__ returned. Calling `save` on those instances will then persist the altered data back to the database.

```ruby
# find the student we want to modify
rosa = Student.where(name: "Rosa").first
rosa.pie #=> "apple"

# change Rosa's pie preference
rosa.pie = "blueberry"
rosa.pie #=> "blueberry"

# assign rosa's motto
rosa.motto = "All things in moderation, except pie."

# save rosa's data back to the database
rosa.save

(0.1ms)  begin transaction
SQL (0.3ms)  UPDATE "students" SET "pie" = ?, "motto" = ?, "updated_at" = ? WHERE "students"."id" = ?  [["pie", "blueberry"], ["motto", "All things in moderation, except pie."], ["updated_at", "2016-04-17 19:57:22.501674"], ["id", 6]]
(1.3ms)  commit transaction
# => true
```

#### Removing rows from the database
Similar to updating data, we can invoke a method on an _instance_ of an AR model to remove it's associated data from the database

```ruby
# count our students
Student.count #=> 7

# find the student we want to remove
libby = Student.find_by(name: "Libby")

# remove the record from the database
libby.destroy

(0.1ms)  begin transaction
SQL (1.5ms)  DELETE FROM "students" WHERE "students"."id" = ?  [["id", 7]]
(1.5ms)  commit transaction
 #=> <Student id: 7, name: "Libby", pie: nil, motto: nil, birthday: nil, created_at: "2016-04-17 20:04:01", updated_at: "2016-04-17 20:04:01", hobby: nil>

# count our students again
Student.count #=> 6

# now there's not a libby in the database
libby = Student.find_by(name: "Libby") #=> nil
```

## What We Learned
- Active Record is an _ORM_, and provides a _DSL_ for modeling queries
- We explored how to create an Active Record _model_ and _migration_
- Some CRUD queries using Active Record in the _Rails console_

## Ridiculously Helpful Resources
- http://guides.rubyonrails.org/active_record_basics.html
- http://guides.rubyonrails.org/migrations.html  
- http://guides.rubyonrails.org/active_record_querying.html  
