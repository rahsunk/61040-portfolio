# Exercise 1

1. The sum of the count numbers of the Purchases in a Request must be at most the count number of the Request. Additionally, every Purchase should belong to a Request with the same Item.

The relation between Purchase and Request is more important because it is what is most responsible for maintaining the relation between the recipient and the gift giver.

The **purchase** action is most affected by this design, since it is responsible for creating a new Purchase and modifying the corresponding Request. It preserves this invariant by first checking if the Registry has a Request for an Item with at least _count_, and decrementing the count of the corresponding Request upon creating the Purchase. This ensures that no Purchase object is created for a Request if it has a _count_ less than the Purchase count, and maintains the relationship between Request and Purchase.

2. The removeItem action can break this invariant. Performing this action would remove the Request with the corresponding item from the given Registry. If this Request had any Purchases, then those Purchases would no longer be linked to a Request. One way to fix this would be to add another constraint where you can not remove a Request with an item if purchases have already been made for it.

3. Yes, the actions imply a Registry can be repeatedly opened and closed, since **open** a registry satisifes the requirements to use **close**, and vice versa. A reason to allow this would be to give the recipient more control, such as if they wanted to temporarily stop Purchases to review their Requests.

4. No, because each Registry has a flag that the recipient can toggle to make it public or private. They also already have the power to remove the Requests in their Registry.

5. A Registry owner can check to see what Users have made Purchases for them, so they could possibly thank them. A gift giver could check to see how what Requests have been made in a particular Registry and decide to make a requested purchase for them.

6. To hide Purchases from the recipient, you could add a flag to a Request, _hidden_, controlling whether or not the owner of the Registry it is in can see the Request's purchases. We can modify addItem so that if it creates a Request, it defaults _hidden_ to true.

We can also define new methods:

showPreferences(registry: Registry)
**requires**: registry exists and has at least one Request with _hidden_ set to true
**effects**: makes all Requests in registry have _hidden_ be false

hidePreferences(registry: Registry)
**requires**: registry exists and has at least one Request with _hidden_ set to false
**effects**: makes all Requests in registry have _hidden_ be true.

7. Representing User and Item as generic types gives more freedom to implementations of GiftRegistration and abstracts them to make them easier to understand compared to giving them defined attributes.
