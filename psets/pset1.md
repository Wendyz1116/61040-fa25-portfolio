# Pset 1

## Exercise 1: Reading a Concept
### 1. Invariants

* For every request, the remaining count + the count of all the purchases should be equal to the counts added to the request with `addItem`.
* Every purchase is always connected to an existing request in the registry. It is more important because the purpose of the concept is to track purchases of requested gifts. If there’s a purchase not associated with a request, then we shouldn’t track this purchase. The `purchase` action is most affected, since it’s recording a new purchase that needs to be connected to a request. It preserves it because it checks that a request exists for the item before recording the purchase.


### 2. Fixing an Action

* `removeItem` could potentially break this. If a request with existing purchases is removed, the invariant breaks, there will be purchases without a request.
* The fix would be to not allow item removal if purchases exist.

### 3. Inferring Behavior

* Yes, a registry can be opened and closed repeatedly since nothing in the `open` or `close` actions forbids this.
* This should be allowed in case the user wants to reopen a registry after closing it, for example to temporarily hide it while updating items and then make it public again.

### 4. Registry Deletion

* I don't think deletion is needed. Users can just close it and closed registries can simply remain inactive.
* However, deletion might matter for storage management or privacy, like if people want their data removed.


### 5. Queries

* Look at purchases to see which items were purchased, and by what user.
* Look at requests and see which items in the registry have positive count and thus are still available to purchase.



### 6. Hiding Purchases

* Add a boolean flag `hidePurchases` to the `Registry`. When true, the actions will still record and track the purchases internally but prevent the owner `User` from being able to query any of the purchases until the flag is switched off.


### 7. Generic Types

* Using generics for `Item` lets the concept work with different item representations instead of hardcoding attributes.
* This is preferable because it allows for simpler identification and matching in large databases and facilitates changes to item details (like prices or descriptions) without affecting the underlying logic of the concept.
* The item’s name, description, or price also isn’t important for a registry, which only needs to track that items are requested and purchased.


## Exercise 2: Extending a familiar concept

### 1. Complete the definition of the concept state

**State:**
a set of Users with
* A username String
* A password String


### 2. Requires/Effects Specification for Actions

**`register(username: String, password: String): (user: User)`**

* **Requires:** There is no User that already have the username
* **Effects:** Creates and return a new User with username and password.

**`authenticate(username: String, password: String): (user: User)`**

* **Requires:** A User exists with the given username and password.
* **Effects:** return that User

### 3. Essential Invariant

* Each username in Users should be unique. It’s preserved since in initial states there’s no Users. Register is the only way to create a new User and it requires that no existing User has the same username as the new one. This check ensures the invariant is maintained.

### 4. Extension: Registration Confirmed by Email

**1. `register(username: String, password: String, email: String): (user: User, secretToken: String)`**

* **Requires:** There is no User that already have the username
* **Effects:** Create and return a new User with username = username, password = password, isConfirmed = false, and secretToken = a generated secret token

**2. `confirmRegistration(username: String, secretToken: String)`**

* **Requires:** There is a User with username and secretToken
* **Effects:** Set isConfirmed of that User to True

**3. State for Email Confirmation Extension:**

a set of Users with

* A username String
* A password String
* A isConfirmed Boolean
* A secretToken String


## Exercise 3: Comparing concepts

### Concept: PersonalAccessToken

**Purpose:**
Users can use personal access tokens instead of passwords to gain access to their account

**Principle:**
After a user registers with a username and password, they can generate multiple personal access tokens, each with an expiration date (default one year), and authenticate using their username and any valid (not expired) token, until the token is deleted or expires

---

### State

* a set of PersonalAccessTokens with:

  * An accessToken String
  * A expirationDate Date
* a set of Users with:

  * A username String
  * A password String
  * A set of PersonalAccessTokens

---

### Actions

**1. `register(username: String, password: String): (user: User)`**

* **Requires:** There is no User that already have the username
* **Effects:** Creates and return a new User and associate it with username, password, and a empty PersonalAccessTokens set

**2. `authenticate(username: String, password: String): (user: User)`**

* **Requires:** A User exists with the given username and password.
* **Effects:** return that User

**3. `authenticate(username: String, accessToken: String): (user: User)`**

* **Requires:** A User exists with the given username and has a PersonalAccessTokens with accessToken that is not expired
* **Effects:** return that User

**4. `generateNewToken(user: User, expirationDate: Date): (token: PersonalAccessTokens)`**

* **Requires:** User exists
* **Effects:** create a new PersonalAccessTokens token with a random string for accessToken and the given expirationDate, add it to the user’s set and return it

**5. `generateNewToken(user: User): (token: personalAccessToken)`**

* **Requires:** User exists
* **Effects:** create a new PersonalAccessTokens token with a random string for accessToken and an expirationDate one year from the current date, add it to the user’s set and return it

**6. `deleteToken(user: User, token: PersonalAccessTokens)`**

* **Requires:** The user exists and have token in their set of PersonalAccessTokens
* **Effects:** Remove token from the user’s set of PersonalAccessTokens

---

### Differences

A user can have multiple tokens, but only one password. Tokens can expire or be deleted while passwords can only be reset. Revoking a leaked token does not require changing the main password. Passwords also authenticate the account itself, while tokens authenticate a session, script, or integration tied to that account.

---

### Improvement

The GitHub page could be clearer by emphasizing that tokens are not just “password substitutes” but separate, revocable credentials. Highlighting the differences listed earlier could also better distinguish them from passwords, instead of just saying treat tokens like passwords.


## Exercise 4: Defining Familiar Concepts

### 1. Concept: URL Shortener

**Purpose:**
Make long URLs easier to share and remember, helping users efficiently distribute and access web resources.

**Principle:**
A user adds a long URL that they want to shorten. They can shorten it to a custom URL suffix or use an autogenerated URL suffix. Multiple short URLs can point to the same long URL.

---

### State

* A set of LongUrl with:
  * A set of ShortUrls

* A set of ShortUrls with:
  * A shortUrl String
  * A originalUrl LongUrl

---

### Actions

**1. `addUrl(url: String): url: LongUrl`**

* **Effect:** create a new LongUrl with empty set of ShortUrls

**2. `shortenUrl(url: String): shortUrl: ShortUrl`**

* **Requires:** url is an existing LongUrl
* **Effect:** Create a ShortUrl with a random autogenerated suffix as the shortUrl and associate it with the url as the originalUrl.

**3. `shortenUrl(url: String, customSuffix: String): shortUrl: ShortUrl`**

* **Requires:** url is an existing LongUrl and customSuffix is in a proper url format
* **Effect:** Create a ShortUrl with the given customSuffix as the shortUrl and associate it with the url as the originalUrl.

---

**Notes:**
Assume a URL can have multiple ShortUrls and short URLs never expire.

---

### 2. Concept: Conference Room Booking

**Purpose:**
Allow people to reserve shared rooms for meetings or work sessions, ensuring that space is available when needed and conflicts are avoided.

**Principle:**
Users can create rooms and time slots. They can reserve a room for a specific slot if it is available. They can delete their reservations too.

---

### State

* A set of Slots with:

  * A startTime DateTime
  * An endTime DateTime
* A set of Rooms with:

  * A roomId String
  * A set of Slots representing available slots
  * A set of Slots representing booked slots
* A set of Bookings with:

  * A User String
  * A room Room
  * A slot Slot

---

### Actions

**1. `newRoom(roomId: String): room: Room`**

* **Requires:** No existing room has roomId.
* **Effect:** Creates a new Room with empty sets of available and booked slots.

**2. `createSlot(startTime: DateTime, endTime: DateTime): slot: Slot`**

* **Requires:** startTime < endTime and slot does not already exist
* **Effect:** Creates a new Slot with startTime and endTime

**3. `addSlotToRoom(roomId: String, slot: Slot)`**

* **Requires:** Room with roomId exists, slot does not overlap with any existing slots in roomId.
* **Effect:** Adds slot to the room’s set of available slots.

**4. `deleteSlotFromRoom(roomId: String, slot: Slot)`**

* **Requires:** Room with roomId exists, There is no booking in that room for slot.
* **Effect:** Removes slot from the room’s available slots.

**5. `createBooking(user: String, roomId: String, desiredSlot: Slot): booking: Booking`**

* **Requires:** room with roomId exists, desiredSlot is available in roomId and not already booked.
* **Effect:** Adds a new booking linking user, room, and desiredSlot; updates the room’s booked slots.

**6. `deleteBooking(booking: Booking)`**

* **Requires:** booking exists.
* **Effect:** Removes the booking and frees up the corresponding slot in that room.

---

**Notes:**

* Overlapping bookings for the same room are not allowed.
* Each room can have multiple non-overlapping booked slots.

---
### 3. Concept: Time-Based One-Time Password (TOTP)
**Purpose:**
Strengthen authentication by limiting access to users who possess their trusted device

**Principle:**
A user registers with a username, password, and trusted device. To authenticate, they must generate a short-lived TOTP from their trusted device and provide it along with their username and password.

---

### State

* A set of TOTP with:

  * A token String
  * An expirationTime DateTime
* A set of Users with:

  * A username String
  * A password String
  * A trustedDevices Devices
  * A set of TOTP
* A set of Devices

---

### Actions

**1. `register(username: String, password: String, trustedDevice: Device): (user: User)`**

* **Requires:** There is no User that already have the username
* **Effects:** Creates and return a new User and associate it with username, password, trustedDevice, and an empty set of TOTP

**2. `generateTOTP(username: String, password: String): (token: TOTP)`**

* **Requires:** A User exists with the given username and password
* **Effects:** Create a new TOTP token with a random string for the token and an expirationTime 10 minutes from the current time. Add the new TOTP to that User

**3. `authenticate(username: String, token: TOTP): (user: User)`**

* **Requires:** A User exists with the given username, password, and token. The token is not already expired
* **Effects:** return that User

---

### Notes

* The user can only have one trusted device that can generate a TOTP for their account
* A device can be a trusted device for multiple accounts
* Every account is forced to use a TOTP to log in
* TOTP will always expire in 10 minutes

**Security improvements:**
Even if passwords are stolen, attackers cannot log in without a current TOTP generated only by the trusted device.

**Remaining vulnerabilities:**
Phishing or malware on the trusted device can still compromise tokens; social engineering attacks could trick users into revealing tokens in real time.
