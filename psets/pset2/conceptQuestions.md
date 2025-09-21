# Concept Questions

1.  The contexts provide a way to keep track of used strings for a specific scope, such that it is possible to use duplicate strings, so long as they are in different contexts.

    In the URL shortening app, a context will be used to represent a shortURL base, such that all registered shortURL suffixes for each base are unique. This allows the app to support the same shortURL suffixes for different shortURL bases without causing conflicts.

2.  The sets of used strings are needed in NonceGeneration because the principle states that it needs to generate a string not used before in a context, so a way to keep track of these used strings is needed to ensure this invariant.

    If a counter _c_ for a context exists such that _c_ increments for every generate action, then:

    Abstraction Function(_c_) => {_c_ unique encoded strings}

3.  One user advantage of using common dictionary words for nonce generation is that it is easy for the user to remember the shortened URL, allowing them to navigate to the target URL easier.

    One disadvantage is that this design choice is less customizable for the user, who may want to generate a shortURL using a suffix of their choice.

    Modifications to NonceGeneration to use this scheme:

    **concept** NonceGeneration [Context]\
    **purpose** generate unique strings within a context\
    **principle** each generate deletes and returns a string from the dictionary for that context

    **state**

         a set of Contexts with
             a dictionary set of Strings

    **actions**

         generate (context: Context) : (nonce: String)
             requires: context exists, dictionary of context is not empty
             effects: returns and deletes a nonce from the dictionary set of context
