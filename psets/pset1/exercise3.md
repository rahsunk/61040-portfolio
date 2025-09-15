# Exercise 3

**concept** PersonalAccessToken\
**purpose** provides an alternative to passwords to authenticate with GitHub\
**principle** after a user creates a token, they or someone else can authenticate GitHub using the token and be granted access to that user's library.

**state**\
a set of Users with:\
a username String\
a set of Tokens

a set of Tokens with:\
a name String\
an owner User\

**actions**

createToken (User: owner): (token: Token)\
**requires**: owner exists
**effects**: create a new token of a random name belonging to owner, and include it in owner's set of Tokens

removeToken (token: Token):
**requires**: token exists
**effects**: removes token from its associated User

authenticate (username: String, token: Token): (user: User)\
**requires**: token exists in the User with the given username
**effects**: grants access to User associated with that token

PersonalAccessToken differs from PasswordAuthentication in that a User can have multiple tokens instead of just one password, and that these tokens can be removed so that they are no longer able to be used to authenticate an account. While PasswordAuthentication is generally used so that only the creator of User has access to their account, PersonalAccessToken is more intended to give other users and machines access to your account.

I would update the GitHub documentation to be more clear about who or what personal access tokens are intended for. The documentation currently stresses that they are similar to passwords, which is true, but it only implies key differences: you can have multiple tokens, you can revoke/remove tokens, etc.
