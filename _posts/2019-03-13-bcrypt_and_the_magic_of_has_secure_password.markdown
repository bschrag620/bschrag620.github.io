---
layout: post
title:      "BCrypt and the magic of has_secure_password"
date:       2019-03-13 20:33:56 -0400
permalink:  bcrypt_and_the_magic_of_has_secure_password
---


When it comes to handling user authentication and protecting users passwords in Rails, the BCrypt gem is a must. There are a couple key components to successfully utilize BCrypt and I wanted to walk through those.

# gem 'bcrypt', '~> 3.1.7'
Step 1 is going to be to include bcrypt inside the gem file.
# password_digest
![](https://imgur.com/lrNSaEe.png)
The next step in creating a secure space for your user's password is to create the users table with the proper field: **password_digest** . When BCrypt does it's magic to store the password, this is where it going to store the encrypted password. If the users table is created with simply a **password** field, then BCrypt will fail. I'll touch on the why of that failure shortly.

# has_secure_password
![](https://imgur.com/MZRja8z)
After properly creating the **password_digest** field, it's time to give the User model the proper class method. **has_secure_password** is made available by the bcrypt gem. By declaring the User model **has_secure_password**, this adds some instance methods to help set and authenticate user passwords. It also, very handily, creates some validations for our model. The importance of the user table containing a **password_digest** field is tied into these instance methods that become inherited , the bcrypt methods are made to write specifically to that field.  If the users table were to be made with only a **password** field, the results would be:
![](https://imgur.com/9OXKEws)

By implementing **has_secure_password**, bcrypt will take care of encrypting the user's password and writing it to the appropriate field within the database. This is very important simply because if there someone were ever able to retrieve user info from your database, plain text passwords is not what we want them to see.

I think it's intersting to note here that **has_secure_password** actually adds a **password** and **password=** methods to the User model. This is important to understand because it means there is a proper, and conversly an improper, way to create the user signup and login forms.
![](https://imgur.com/9hoVP3Q)

Focusing a bit more closely on the above snapshot, it can be seen that whenever **password=** gets called, **has_secure_password** is going to store an encrypted version of the plain text password in the **password_digest**  field. Maybe it's not so magical after all ;-)
![](https://imgur.com/G9w1M7c)

# creating your user signup and login form
(and this is where I've stumbled...)
Because **has_secure_password** auto-magically creates a **password** attribute accessor for our model, that's exactly what the field name should be that our users are filling out.
![](https://imgur.com/ckE4RnC)
![](https://imgur.com/k8Kp483)
As stated earlier, this also leaves the door open to doing it a wrong way, naming the fields **password_digest**. Oddly enough, this works. Let's see what happens though in `rails c` if that's done. First, the proper way: 
![](https://imgur.com/B0LDdZb)
Notice a new user was created and by simply saying **password=**, the users password became a jumbled, hashed mess in the **password_digest** field. Now, let's do it the wrong way:
![](https://imgur.com/p6cB41M)
By explicitly calling **password_digest=** the users password got stored as plain text. This is a big no-no!!! So when creating your signup and login forms, simply use **password** for the field names.

# more magic: authenticate_password
Ok, so we've seen that it isn't **really** magic, but it kind of is. One of the other amazing instance methods that you will gain access to through use of **has_secure_password** is  **#authenticate_password**
![](https://imgur.com/zUrTzZQ)
**#authenticate_password** acts as a means to pass in a plain text password and validate it as equal to the hashed password in the database. This is simply accomplished by calling **is_password?**, which is a class method contained in bcrypt.
![](https://imgur.com/dpAREkh)
In our controller, this would look something like:
![](https://imgur.com/CeYBqwt)
This allows us to safely test a user inputed password to see if it matches the record in the database.
# still producing magic: validations

The final topic I'd like to touch on are a couple of key validations. First, **has_secure_password** will automatically ensure that **password_digest** is an existing field within the table. Seond, it will ensure that it is not empty. This of course means that if a user tries to make an account and leaves the password field empty, the user cannot be saved into the database. This is great of course, but the really magical validation that gets implemented, in my opinion, is **password_confirmation**. 

The **confirmation** validator can be added to any fields that you want to be sure the user properly entered (email, username, password are quite common) when they are signing up to create an account. The **confirmation** validator will handle comparing the regular field to it's confirmation field for equality (password == password_confirmation, email == email_confirmation, etc.). This signup snapshot has these fields built in already:
![](https://imgur.com/ckE4RnC)

Since **has_secure_password** takes care of implementing these validations on password, there is no requirement to explicitly call the validations within the model. However, if you wanted to add in confirmations on other fields that aren't the password, this is what that might look like:
![](https://imgur.com/rl1McAY)

The other key thing about **password_confirmation** that I love that they intentionally did is you don't HAVE to use it. Looking at the very last line (117), **password_confirmation** is allowed to be blank.
![](https://imgur.com/g4EaSqW)

To recap, let's shorten this all up into the key points of how to implement and leverage the **magic** of the **has_secure_password** class method and bcrypt.
1. include bcrypt in your gem file
2. use has_secure_password on the model you are trying to protect
3. properly name the signup and login fields as only "password"
4. utilize #authenticate on the user to test if their password is correct
5. win!

For me info visit the Ruby API below or the source pages for secure_password and bcrypt and thanks for reading!

Brad
Ruby API for has_secure_password: https://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html#method-i-has_secure_password

secure_password source: https://github.com/rails/rails/blob/master/activemodel/lib/active_model/secure_password.rb

bcrypt/password source: https://github.com/codahale/bcrypt-ruby/blob/master/lib/bcrypt/password.rb
