Exercise 1: Reading a Concept

- **Invariants.** What are two invariants of the state? (Hint: one is about aggregation/counts of items, and one relates requests and purchases). Say which one is more important and why; identify the action whose design is most affected by it, and say how it preserves it.
    - The first invariant is that the current total purchased count (sum of all purchase counts) of an item and its current request count must add up to the initial request count.
    - The second invariant is that all the purchases in the set of purchases of an item cannot have another item as a target.
    - The most important is the first invariant that pertains to counts. The action whose design is most affected by it is `purchase`, and the invariant is preserved by it because the request count is decreased by the amount that the total purchase count is increased.

- **Fixing an action.** Can you identify an action that potentially breaks this important invariant, and say how this might happen? How might this problem be fixed?
    - `addItem` can potentially break the important invariant by increasing the count of an item request while purchases are being made.
    - This would cause the sum of the total purchase counts and current item request counts to be higher than the initial item request counts.
    - This problem can be fixed by not allowing the action to be called when the registry is active and purchases are being made.

- **Inferring behavior.** The operational principle describes the typical scenario in which the registry is opened and eventually closed. But a concept specification often allows other scenarios. By reading the specs of the concept actions, say whether a registry can be opened and closed repeatedly. What is a reason to allow this?
    - A reason to allow this could be if the owner changes their mind about closing or opening the registry at any given time.

- **Registry deletion.** There is no action to delete a registry. Would this matter in practice?
    - In practice, and from a user perspective, no, it would not matter. This is because once a registry is closed it can no longer be viewed by any of the owner’s target users.

- **Queries.** What are two common queries likely to be executed against the concept state? (Hint: one is executed by a registry owner, and one by a giver of a gift.)
    - By the owner: a query about which items and amounts have been purchased.
    - By a giver: a query about which items have not been purchased and are available to give.

- **Hiding purchases.** A common feature of gift registries is to allow the recipient to choose not to see purchases so that an element of surprise is retained. How would you augment the concept specification to support this?
    - This could be implemented by adding a boolean option to the registry that does not display the count of items to the owner.

- **Generic types.** The User and Item types are specified as generic parameters. The Item type might be populated by SKU codes, for example. Explain why this is preferable to representing items with their names, descriptions, prices, etc.
    - It allows flexibility across applications and uses of the registry. It also abstracts away unnecessary complexity.

---

Exercise 2

- **Complete the definition of the concept state.**
  - **state**
    - a set of Users with
      - a username String
      - a password String

- **Write a requires/effects specification for each of the two actions.** (Hints: The register action creates and returns a new user. The authenticate action is primarily a guard, and doesn’t mutate the state.)
  - **actions**
    - register (username: String, password: String): (user: User)
      - **requires** username does not exist
      - **effect** create a new User with this username and this password
    - authenticate (username: String, password: String): (user: User)
      - **requires** there exists a User with this username
      - **effect** authenticates the user if the password is correct; otherwise it does not

- **What essential invariant must hold on the state? How is it preserved?**
    - Every User has a unique username with one password associated with it.
    - `register` preserves that by requiring that usernames cannot be duplicated and that only one password can be registered with the username.
    - `authenticate` does not change usernames or passwords; it just authenticates, so the invariant is preserved.

- **One widely used extension of this concept requires that registration be confirmed by email. Extend the concept to include this functionality.** (Hints: you should add (1) an extra result variable to the register action that returns a secret token that (via a sync) will be emailed to the user; (2) a new confirm action that takes a username and a secret token and completes the registration; (3) whatever additional state is needed to support this behavior.)
  - **(revised) state**
    - a Confirmed set of Users with
      - a username String
      - a password String
    - an Unconfirmed set of Users with
      - a username String
      - a password String
      - a token String
  - **(revised) actions**
    - register (username: String, password: String): (user: User, token: String)
      - **requires** username does not exist
      - **effect** create a new User with this username and this password
    - confirm (username: String, token: String)
      - **requires** there exists an Unconfirmed User with this username with this token associated with it
      - **effect** confirms the User with this username and moves it to the Confirmed set of Users

---

Exercise 3

- **concept:** PersonalAccessToken [User, Scope]
- **purpose:** limit access to GitHub through APIs or the command line
- **principle:** users create a personal authentication token with desired scopes that act as permissions for certain actions; then, users can use their token to authenticate their actions when interacting with GitHub through HTTPS
- **state**
  - a set of Tokens with
    - an owner User
    - an authToken String
    - a scopes set of Scope
    - an optional expirationTime Time
- **actions**
  - generateToken (owner: User, scopes: set of Scope, expirationTime: Time): (token: Token)
    - **requires** owner exists and scopes in the set are existing and valid scopes
    - **effects** create a new Token with all the specified parameters
  - deleteToken (owner: User, authToken: String)
    - **requires** a token with this authToken exists and is associated with this user
    - **effects** remove the Token from the set of Tokens
  - authenticate (username: String, authToken: String): (user: User)
    - **requires** a user with this username exists and a token with this authToken also exists
    - **effects** authenticates the user if the authToken is correct and unexpired; otherwise it does not

- **Notes**
  - The primary difference between PersonalAccessToken and PasswordAuthentication is that PersonalAccessToken allows for more flexible and granular control over the authentication of a service. A user can have different tokens with different permissions that can be created, managed, and deleted independently of each other. PasswordAuthentication is better for general application use that does not need complex management of permissions or service interactions.
  - To more effectively demonstrate this, I would add more examples of personal token usage to the documentation. As of right now the only example has to do with `git clone`, so I would add a different situation where the usage of personal tokens comes in handy.

---

Exercise 4

- **URL Shortener**
  - **concept:** URLShortener [User]
  - **purpose:** shorten the original length of a site URL while keeping the destination
  - **principle:** users select a site URL they wish to shorten; then users can decide if they want their shortened URL with a generated or custom suffix; finally, users can access their original site using this short URL
  - **state**
    - a set of URLs with
      - an originalURL String
      - a suffix String
      - an owner User
  - **actions**
    - createAutogeneratedURL (owner: User, originalURL: String): (shortenedURL: URL)
      - **requires** this user exists
      - **effects** creates a new URL with this originalURL and a unique randomly generated suffix, and this owner
    - createCustomURL (owner: User, originalURL: String, customSuffix: String): (shortenedURL: URL)
      - **requires** this user exists and customSuffix does not already exist
      - **effects** creates a new URL with this originalURL, this custom suffix, and this owner
    - changeURL (caller: User, shortenedURL: URL, newURL: String)
      - **requires** shortenedURL exists and caller is the owner of the shortenedURL
      - **effects** changes the originalURL of the shortenedURL to newURL
    - lookup (suffix: String): (originalURL: String)
      - **requires** a URL with the suffix exists
      - **effects** returns the originalURL that is associated with the URL with this suffix
    - deprecate (shortenedURL: URL)
      - **requires** shortenedURL exists
      - **effects** removes this URL from the set
  - **Notes**
    - The URL does not have an attribute like `shortenedURL String` because I am operating under the assumption that this is a service like TinyURL, where the domain is in place, and all that matters is the suffix to construct the actual URL.

- **Billable Hours Tracking**
  - **concept:** BillableHoursTracking
  - **purpose:** track billable hours of employees for record keeping
  - **principle:** employees start their working-hours session by selecting a project with a description of their work; after their working hours are over, employees end their session
  - **state**
    - a set of Employees with
      - a name String
      - an ID Number
    - a set of Projects with
      - a projectName String
      - a projectDetails String
    - a set of Sessions with
      - a user Employee
      - a project Project
      - a description String
      - a startTime DateTime
      - an endTime DateTime
      - an expirationTime DateTime
      - an expired Flag
      - a userEnded Flag
  - **actions**
    - startSession (employee: Employee, project: Project, description: String): (session: Session)
      - **requires** this employee exists, this project exists, and the employee has no ongoing sessions
      - **effects** creates a new session for this employee under this project with this description and the current time as the startTime; it also has an expirationTime of startTime + X hours, with X being an arbitrary number set by the organization; the session is initialized with both flags set to FALSE
    - endSession (employee: Employee, session: Session)
      - **requires** this employee exists, this session exists, and the employee has this session associated with them
      - **effects** sets the endTime of the session to the current time; sets the userEnded flag to TRUE
    - endExpiredSession (session: Session)
      - **requires** this session exists and the current time is past the session’s expirationTime
      - **effects** if the userEnded flag is FALSE, sets the endTime of the session to the expirationTime; sets the expired flag to TRUE
    - changeSessionEndTime (session: Session, time: DateTime)
      - **requires** this session exists and the time is later than the session’s startTime
      - **effects** sets the endTime of the session to the desired time
    - changeSessionStartTime (session: Session, time: DateTime)
      - **requires** this session exists and the time is earlier than the session’s endTime
      - **effects** sets the startTime of the session to the desired time
  - **Notes**
    - There are two flags in order to avoid expiring a session that has already ended. The expiration time is the startTime plus an offset that is determined by the organization and is outside the scope of the employee.

- **Conference Room Booking [User]**
  - **concept:** ConferenceRoomBooking
  - **purpose:** allow for organized and trackable management of conference room reservations
  - **principle:** users select a desired time period for their booking; then they are able to see available rooms along with capacities and details; finally, they can select a room that then becomes unavailable for that time period
  - **state**
    - a set of Rooms with
      - a name String
      - a capacity Number
      - an accommodations set of Strings
      - a features set of Strings
    - a set of Reservations with
      - a room Room
      - a reserver User
      - a reservationStartTime DateTime
      - a reservationEndTime DateTime
  - **actions**
    - createReservation (reserver: User, room: Room, startTime: DateTime, endTime: DateTime): (reservation: Reservation)
      - **requires** user and room exist, startTime >= current time, endTime > startTime
      - **effects** creates a reservation for this room with this reserver, startTime, and endTime
    - cancelReservation (reserver: User, reservation: Reservation)
      - **requires** reserver and reservation exist, and reserver is associated with the target reservation
      - **effects** deletes the reservation from the set of Reservations
    - queryAvailableRooms (reserver: User, room: Room, startTime: DateTime, endTime: DateTime, capacityMinimum: Number, desiredAccommodations: set of Strings, desiredFeatures: set of Strings): (rooms: set of Rooms)
      - **requires** user and room exist, and endTime > startTime
      - **effects** returns a set of Rooms that are not part of a reservation between startTime and endTime and that also have a capacity of at least capacityMinimum, whose accommodations include the desiredAccommodations, and whose features contain the desiredFeatures
    - addRoom (name: String, capacity: Number, accommodations: set of Strings, features: set of Strings): (room: Room)
      - **requires** —
      - **effects** creates a Room with this name, capacity, features, and accommodations
    - deleteRoom (room: Room)
      - **requires** room exists
      - **effects** deletes the room from the set of Rooms
