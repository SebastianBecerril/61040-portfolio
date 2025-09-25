# Concept Questions

## 1) Contexts
**The NonceGeneration concept ensures that the short strings it generates will be unique and not result in conflicts. What are the contexts for, and what will a context end up being in the URL shortening app?**

- Contexts are in place so that separate domains have their own unique shortenings. Contexts will end up being used in the URL shortening app with the **shortUrlBase** values, since those are the domains that need to be separated. It does not matter if TinyURL and bit.ly both have a link that has the same suffix first, as there can be no confusion since they are separate spaces.

## 2) Storing used strings
**Why must the NonceGeneration store sets of used strings? One simple way to implement the NonceGeneration is to maintain a counter for each context and increment it every time the generate action is called. In this case, how is the set of used strings in the specification related to the counter in the implementation? (In abstract data type lingo, this is asking you to describe an abstraction function.)**

- A set of used strings is used in NonceGeneration since that is an intuitive way of keeping track of strings that have been used, so that unique strings can be generated and then added to this used set. In the case of using a counter, the idea of only generating unique strings would be the same. To keep track of “a set of used strings” with a counter, we would define a non-colliding transform or hash function that translates an integer into a unique string. Therefore, every time **generate** is called, the counter increases by 1, and this new counter is transformed into a unique string logically by the fact that this new counter is not the same as all the previous counter numbers.

## 3) Words as nonces
**One option for nonce generation is to use common dictionary words (in the style of yellkey.com, for example) resulting in more easily remembered shortenings. What is one advantage and one disadvantage of this scheme, both from the perspective of the user? How would you modify the NonceGeneration concept to realize this idea?**

- **Advantage:** The shortenings are easier to remember and type, as people know the spellings of words compared to a random string of letters that can be easily mistyped.
- **Disadvantage:** There is a finite amount of English words, so eventually you would run out.
- **Realization:** Instead of generating a random string, randomly select an English word that is not in the used set.

---

# Synchronization Questions

## 1) Partial matching
**In the first sync (called **generate**), the `Request.shortenUrl` action in the **when** clause includes the `shortUrlBase` argument but not the `targetUrl` argument. In the second sync (called **register**) both appear. Why is this?**

- The purpose of the **generate** sync is to generate the nonce for this particular instance; **shortUrlBase** is needed since it is the context for the nonce, but **targetUrl** is not relevant for this operation. In the **register** sync, both appear because the output of **register** is the entire URL (base + nonce), so both arguments are relevant and needed.

## 2) Omitting names
**The convention that allows names to be omitted when argument or result names are the same as their variable names is convenient and allows for a more succinct specification. Why isn’t this convention used in every case?**

- This convention is not used in every case because there are many occasions when variable naming is needed. This might be for the simple reason of being more specific and allowing for better readability of concepts, or it might be a more specific case of two arguments/results having the same type and needing to be differentiated.

## 3) Inclusion of request
**Why is the request action included in the first two syncs but not the third one?**

- The request action is not included in the third sync because the third sync is triggered strictly by the **register** action, and it does not need the context or arguments of the request (base and `targetUrl`).

## 4) Fixed domain
**Suppose the application did not support alternative domain names, and always used a fixed one such as “bit.ly.” How would you change the synchronizations to implement this?**

- Assuming that we are strictly changing the syncs and cannot modify the concept definitions, this can be done by simply hardcoding our one domain (e.g., **bit.ly**) into the `shortUrlBase` argument of **generate** and **register**.

## 5) Adding a sync
**These synchronizations are not complete; in particular, they don’t do anything when a resource expires. Write a sync for this case, using appropriate actions from the **ExpiringResource** and **UrlShortening** concepts.**

- **sync** `expiration`  
- **when** `ExpiringResource.expireResource(): (resource: shortUrl)`  
- **then** `UrlShortening.delete(shortUrl)`

---

# Extending the Design

**Design a couple of additional concepts to realize this extension, and write them out in full (but including only the essential actions and state). It should not be necessary to make any changes to the existing concepts.**

## **Concept:** `ResourceRegistration [Resource, User]`
- **Purpose:** keep records of who registered a resource  
- **Principle:** after registering a resource, it will be attached to a user  
- **State:**  
  - a set of **Resources** with  
    - an **owner**: `User`  
- **Actions:**  
  - **attachOwner(owner: User, resource: Resource): ()**  
    - **Requires:** resource exists and has no owner; user exists  
    - **Effect:** associates the existing resource with an owner that can then have special privileges to it

## **Concept:** `AnalyticsRecords [Resource, User]`
- **Purpose:** view analytics of resource usage  
- **Principle:** after a resource is used, its usage analytics reflect that by increasing the usage counter  
- **State:**  
  - a set of **Resources** with  
    - a **usageCounter**: `number`  
- **Actions:**  
  - **createAnalytics(owner: User, resource: Resource): ()**  
    - **Requires:** owner owns the resource and both exist  
    - **Effect:** creates the `usageCounter` for the resource and starts it at 0  
  - **increaseUsageCounter(resource: Resource, increase: number)**  
    - **Requires:** resource exists  
    - **Effect:** increases the `usageCounter` of a resource by the `increase` number  
  - **viewAnalytics(user: User, resource: Resource): (usageCounter: number)**  
    - **Requires:** resource has an analytics record attached to it  
    - **Effects:** returns the `usageCounter` analytics of the resource if the user owns it; if not, it returns a prohibited-action exception

**Specify three essential synchronizations with your new concepts: one that happens when shortenings are created; one when shortenings are translated to targets; and one when a user examines analytics. (Hint: for the last one, you can simplify by assuming a request for analytics that just asks for the number of lookups for a particular short URL, but that ensures that the result is not visible to everyone.)**

- **sync** `createAnalytics`  
  - **when**  
    - `Request.shortenUrl(owner: User)`  
    - `UrlShortening.register(): (shortUrl)`  
  - **then**  
    - `ResourceRegistration.attachOwner(owner, resource: shortUrl)`  
    - `AnalyticsRecords.createAnalytics(owner, resource: shortUrl)`

- **sync** `updateAnalytics`  
  - **when** `UrlShortening.lookup(shortUrl): ()`  
  - **then** `AnalyticsRecords.increaseUsageCounter(resource, 1)`

- **sync** `viewAnalytics`  
  - **when** `Request.viewAnalytics(user, shortUrl)`  
  - **where** `shortUrl.owner == user`  
  - **then** `AnalyticsRecords.viewAnalytics(user, shortUrl)`

---

# Extending the design

## Allowing users to choose their own short URLs
- This can be done by first adding two new arguments to the request: a flag called **customSuffixFlag** and a string for it called **customSuffix**. A new sync called **registerCustom** is created that follows the same logic from **register**, but with an extra **when** condition of the **customSuffixFlag** being true. We also modify the original **generate** and **register** syncs to have an additional **when** condition of the **customSuffixFlag** being false.

## Using the “word as nonce” strategy to generate more memorable short URLs
- This is simply done by modifying the **NonceGeneration** concept. The **generate** action now returns a random English word instead of a random string, and we keep the logic of the used set.

## Including the target URL in analytics, so that lookups of different short URLs can be grouped together when they refer to the same target URL
- This feature can be added by first adding a new concept called **TargetURLAnalytics**. For the state, a set of **GroupAnalytics** that each have a set of **targetUrl** resources, each with a set of associated **shortUrls** that have the same owner. This way we group **targetUrls** with their respective **shortUrls** that all have the same owner. For actions, they would follow similar logic to the actions for the **Analytics** concept described previously.  For syncs, we create three syncs very similar to the already existing analytics logic: a sync for adding a shortUrl analytics to its parent targetUrl group, a sync for updating the targetUrl group analytics when the shortUrl member analytics are updated, and a sync to view the targetUrl analytics.

## Generate short URLs that are not easily guessed
- In my opinion, this feature would be undesirable and should not be included. This is because the focus of shortened URLs should not be privacy, as that should fall under the scope of the target URL. If administrators do not want their site to be easily accessed, then they should implement features in their platform that work toward that.

## Supporting reporting of analytics to creators of short URLs who have not registered as users
- I believe this feature to be undesirable and it should not be included. Even though it might just be URL analytics, sensitive information like this should be protected under some sort of system of authentication. This could be done in a variety of ways, but the typical user-and-password system is a proper way to implement this. If we allow non-users to view analytics, this could lead to many potential security problems, such as unauthorized access to others’ analytics or the leakage of a user’s own. Therefore, in order to view analytics, users should be registered.
