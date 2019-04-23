# A quick look at Rails Custom Validation

I  recently started working with Ruby(Almost 2 months now) and Ruby on  Rails(A little over 3 weeks). Working with Rails’ Active Record  framework is one of my favorite things about the framework. In this  post, we shall be looking at validations in Active Records, custom ones  particularly. Here is a quick intro to Active Records before we move to  the good stuff.

Active  Records is one of the core gems that makes up Ruby on Rails, it is the  part of the framework that deals with databases. It is an ORM framework  that let’s you build a schema for database using pure ruby and it is  based on the Active Record design pattern described by [Martin Fowler](https://medium.com/r/?url=https%3A%2F%2Fmartinfowler.com%2F).  So, with Active Records, you are creating your db, creating tables,  storing, retrieving and deleting data using ruby which it translates to  SQL under the hood. 

### **Quick intro**

Suppose we have a student model with properties first name and last name, to use Active Records we just need to extend the **ApplicationRecord** and when we run `rails db:migrate `it gives us the SQL statement for it.

```ruby
class Student < ApplicationRecord
end



# CREATE TABLE students (
#    id int(11) NOT NULL auto_increment,
#    first_name varchar(255),
#    last_name varchar(255),
#    PRIMARY KEY  (id)
# );
```

To interact with the database, we use methods inherited from the ApplicationRecord super class

```ruby
Student.all
#SELECT * FROM students

Sudent.create(first_name: "Ade", last_name: "Obi")
#INSERT INTO "students" ("first_name", "last_name", "created_at", "updated_at") VALUES ("Ade", "Obi", "2019-04-18 06:05:43.426106", "2019-04-18 06:05:43.426106") 

ade = Student.find(1)
#SELECT  "students".* FROM "students" WHERE "students"."id" = 1 LIMIT 1
```

### **Validation**

Because,  we write web applications for users other than ourselves, we cannot be  sure that the users will always input valid data into the database, to  enforce this, Active Records provides us a mini-validation framework  which ensures the presence of data, uniqueness of certain fields and so  on.

Let’s  look at our students table above, we wouldn’t want to create a user  without a first name or last name which presently is possible, to  mitigate this, we just need to modify our Student class like so

```ruby
class Student < ApplicationRecord
  validates :first_name, presence: true
  validates :last_name, presence: true
end
```

With  this modification, when you create a Student instance without the first  name or last name attributes, it is an invalid student and active  records will not persist it to the database. 

Active records also provides us with methods to check if our data is valid or invalid

```ruby
s1 = Student.create(first_name: "David", last_name: "Mustapha")
s1.valid? 
# => true

s2 = Student.new
# <Student id: nil, first_name: nil, last_name: nil, created_at: nil, updated_at: nil>

s2.valid?
# => false
```

with this, we do not even have to attempt to save the data.

Apart  from just preventing the data from being persisted, Active records also  provides an errors list which which holds the attributes that failed  validations and user friendly messages to present to the users. These  errors can be accessed as shown in the snippet below.

```ruby

s4 = Student.new
s4.valid? # => false
s4.errors
#<ActiveModel::Errors:0x00007fffea94a240 @base=#<Student id: nil, first_name: nil, last_name: nil, created_at: nil, updated_at: nil>, @messages={:first_name=>["can't be blank"], :last_name=>["can't be blank"]}, @details={:first_name=>[{:error=>:blank}], :last_name=>[{:error=>:blank}] :count=>2}]}>

s4.errors.messages
# => {:first_name=>["can't be blank"], :last_name=>["can't be blank"]}
 
s5 = Student.new(first_name: "David", last_name: "Mustapha")
s5.valid? # => true
s5.errors.messages # => {}
```

There  is a lot more on validation but it’s not the topic of this article for a  deep dive, you can get an in-depth explanation from the ruby on rails  guide chapter on Validation. 

### **Custom Validation**

Sometimes,  we might want to use certain validations that are more than just  ensuring presence of an attribute, length, uniqueness or any of the  helpers provided by Active records. Luckily, Active records allows for  us to define our own custom validations which is the point of this  article.

So,  let’s say for our Student model, we have a compulsory student  registration number column which has to be filled from the registration  form(I know this can be auto-generated) which should always start with  the registration year. Now, Active records does not provide this kind of  validation out of the box but have made it possible for us to define it  and use it.

There are mainly two ways to define your own validation logic,

- Custom Validator
- Custom Methods

**Custom Validator**

To validate using a custom validator, you just need to define your validation logic in a class that extends **ActiveModel::Validator** and implements the validate method, which takes the record to be validated as its argument.

If  validation fails it adds the attribute to the errors array along with  it’s error message. So, in our case, we’ll have RegNumValidator as seen  below,

```ruby
class RegNumValidator < ActiveModel::Validator
  def validate(record)
    unless record.reg_num.starts_with? record.reg_year
      record.errors[:reg_num] << 'Registration number must begin with Registration year'
    end
  end
end
```

To use this validator in the Student model, we use the `validates_with` method

```ruby
class Student
  include ActiveModel::Validations
  validates_with RegNumValidator
end
```

With this, when a user tries to create a student with the wrong registration 
number, the record creation fails and an error message can be shown.

```ruby

s6 = Student.new(first_name: "David", last_name: "Mustapha", reg_year: 2019, reg_num: "2016DAVM")
s6.valid? # => false
s6.errors.messages
# => {:reg_num=>["Registration number must start with registration date"]}
 
s7 = Student.new(first_name: "David", last_name: "Mustapha", reg_year: 2019, reg_num: "2019DAVM")
s7.valid? # => true
s7.errors.messages # => {}
```

**Custom Methods**

To  use custom methods for validation you just need to define a method to  use for validation in your model class and call it like you would call  any of the in-built validations i.e using `validate`. Using the same logic as what we had above, our model would look like this. 

```ruby
class Student < ApplicationRecord
  validate :reg_num_validator
 
  def reg_num_validator
    if reg_num.starts_with? reg_year
      errors.add("Registration number must start with #{reg_year}")
    end
  end
end
```

### Conclusion

I  hope this article has given you the necessary knowledge to begin to  explore Active records validation and custom validation especially.  Whenever you have a validation requirement that is not part of the  existing active record validation api, you can just write one yourself.