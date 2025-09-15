# Exercise 2

1.  **state**\
    a set of Users with:\
    a username String\
    a password String

2.  **actions**

register (username: String, password: String): (user: User)\
 **requires**: no User with username exists\
 **effects**: create a new User with the given username and password

authenticate (username: String, password: String): (user: User)\
 **requires**: User with the same username and password exists\
 **effects**: grants access to the User associated with that username and password

3. Every User must have a unique username. It is preserved by the _register_ action, which is the only action that can create a new User. It does not create a User if there already exists a User with the given name.

4. **state**
   a set of Users with:\
   a username String,\
   a password String\
   a flag confirmed\
   a token String

register (username: String, password: String, email: String): (user: User, token: String)\
 **requires**: no User with username exists\
 **effects**: create a new User with the given username and password, with confirmed set to false, and a secret token to authenticate through their email address.

confirm(username: String, token: String): (user: User)\
 **requires**: User with the same username and token exists\
 **effects**: sets confirmed of the associated User to true, and grants access to this User
