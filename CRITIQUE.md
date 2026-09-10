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

**The problem.** Name it, using the vocabulary from lecture (milestone 2 in the
handout names the three).

**Where in the code.** File and method.

**What it makes expensive.** A concrete future change, or something that already goes
wrong today. What breaks first?

### Problem 2

**The problem.**

**Where in the code.**

**What it makes expensive.**

---

## Milestone 3: Two alternative decompositions

Two different ways to carve up this system. A different split of responsibility, not a
list of local code fixes. Read the handout's appendix before writing this section.

### Alternative A

**The decomposition.** What are the pieces, what does each own, and where do the rules
live?

**One tradeoff.** Something this option actually costs. "No real downside" is not a
tradeoff.

### Alternative B

**The decomposition.**

**One tradeoff.**

### Preference

Which one, and under what conditions? Say what the choice depends on, and what would
make you pick the other one instead.
