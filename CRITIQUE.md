# RoomReserve Critique

Fill in each section. One section per milestone. Keep it short and specific. Point at
files and methods, not adjectives.

---

## Milestone 1: The design as it is

Describe the system as the code actually builds it.

**Data model.** There is no booking class or object in this implementation of the codebase. Instead, the data is fragmented and stored across multiple generic collections. The bookings are stored in a Map in InMemoryStore, keyed by room ID and the value is an array of startMinutes to endMinutes. 

**Operations.** Callers can create, cancel, reschedule and list. The I/O structure is strings that get manually parsed and return plain strings containing either confirmations or error messages.

**Structure.** 
RequestHandler is the main controller. It is responsible for parsing string inputs, performing validation and returning string responses. 
InMemoryStore is the database that holds the underlying structure and handles the R/W data of the collections.

**The no-double-booking invariant.** 
There is an overlap check in RequestHandler.createBooking. It iterates over all existing slots and checks if the new start time is before an existing end time and the existing start times are before the end times. There is also a second check that checks if the exact slot already exists in InMemoryStore.addSlot.

A call comes into rescheduleBooking that gets parsed into minutes. It queries the store to find the user with the old booking to make sure that eactually exists. The flaw seen is that it calls removeSlot and then immediately calls add, with no checks for existing bookings. InMemoryStore.addSlot then checks if the exact slot already exists, which means there is potential for a double booking.

---

## Milestone 2: Two design problems

Two problems. For each one, fill in all three parts.

### Problem 1

**The problem.** Code Duplication. The logic used to parse time strings into `long` minutes and validate the `"HH:MM"` format is copy-pasted almost identically across multiple methods.

**Where in the code.** In `RequestHandler.java`. The parsing block appears in `createBooking`, `cancelBooking`, and `rescheduleBooking`.

**What it makes expensive.** Any future change to the time format (e.g. supporting 12-hour AM/PM times or adding seconds) would require updating this logic in three different places. Forgetting to update one would introduce bugs.

### Problem 2

**The problem.** Primitive Obsession. The system represents complex domain concepts (like a Booking) using primitive data types (like `long[]` arrays or concatenated Strings like `"roomId|date"`) rather than creating dedicated classes.

**Where in the code.** Primarily in `InMemoryStore.java` (using `Map<String, List<long[]>>` and `Map<String, String>`), but also in `RequestHandler.java` when it works with these primitives instead of domain objects.

**What it makes expensive.** Extending the data model. If a future change requires adding "number of attendees" or a "meeting title" to a booking, it becomes very expensive and error-prone because there is no `Booking` object to easily add fields to. You would either have to add and maintain a third `Map`, or change the `long[]` structure everywhere.

---

## Milestone 3: Two alternative decompositions

Two different ways to carve up this system. A different split of responsibility, not a
list of local code fixes. Read the handout's appendix before writing this section.

### Alternative A

**The decomposition.** Introduce a `Booking` domain object and a `TimeParser` utility. The `Booking` object encapsulates the start time, end time, user, and room, and owns the logic to check if it overlaps with another `Booking`. `InMemoryStore` manages collections of `Booking` objects instead of raw `long[]` primitives. `RequestHandler` delegates time string parsing to the `TimeParser` and interacts with the store exclusively using `Booking` objects.

**One tradeoff.** Creating a formal domain model means more classes and objects are instantiated on every request, which slightly increases memory overhead compared to primitive arrays. It also requires an upfront, sweeping refactoring of both `InMemoryStore` and `RequestHandler` to swap out the primitives, causing high churn on code that currently works.

### Alternative B

**The decomposition.** Split `RequestHandler` into a `StringController` and a `BookingService`. The `StringController` owns only the parsing of string inputs and formatting of string outputs/errors. The `BookingService` owns the core business rules (like enforcing the no-double-booking invariant) and coordinates between the controller and the `InMemoryStore`. The rules live in the service layer, keeping the controller dumb and the store focused purely on data storage.

**One tradeoff.** Introducing a middle service layer adds indirection. A simple request now has to travel through three layers (Controller -> Service -> Store) instead of two. This means a developer has to trace through more files to understand the end-to-end flow of a basic request, even if the individual files are more focused.

### Preference

I prefer Alternative A if we anticipate adding more properties to a booking (like attendees, meeting titles, or recurrence rules) or more complex time logic (like timezones). Primitive arrays simply cannot scale to handle those additions. However, I would pick Alternative B instead if the data shape (`startMinutes`, `endMinutes`) is guaranteed to be fixed forever, but the business rules for *when* you can book (e.g., enforcing maximum booking lengths or user quotas) are expected to grow rapidly. In that case, a dedicated service layer for business rules is more valuable than a rich domain model.
