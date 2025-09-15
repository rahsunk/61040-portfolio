# Exercise 3

**concept** PersonalAccessToken\
**purpose** limit access to known users\
**principle** after a user registers with a username and a password, they can authenticate with that same username and password and be treated each time as the same user\

    **state**\
    a set of Users with:\
    a username String\
    a password String\

2.  **actions**

    register (username: String, password: String): (user: User)\
    **requires**: no profile with username exists\
    **effects**: create a new User with the given username and password

    authenticate (username: String, password: String): (user: User)\
    **requires**: profile with username exists\
    **effects**: allows User access\
