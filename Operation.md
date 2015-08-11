The [Operation](Operation.md) class represents asynchronous operations that can communicate
intermediate results via _events_. Event listeners are (un)registered for an
operation via the ([#off](#off.md)) [#on](#on.md) method, and the operation proper is started
with the [#start](#start.md) method. `google-musicmanager.js` does not actually expose a
constructor for this class, but it is documented here as the return values of
the [Client](Client.md) methods are of this type.

Each [Operation](Operation.md) has a private namespace that is bound to the `this` context
of its constituent functions and registered event listeners. By default, the
namespace contains the array `events` of event listeners and the function
`emit` for generating events. In `google-musicmanager.js`, this namespace
additionally includes copies of the `id` and `token` fields of its creating
[Client](Client.md). As these are hard copies, they need to be manually updated whenever
the corresponding values change for the [Client](Client.md).

All [Operation](Operation.md)s in `google-musicmanager.js` can emit at least the following
two events:

**end():**
Emitted when the operation finishes successfully.

**error(err, resume):**
Emitted when the operation encounters an error. `err` is an
(operation-specific) object representing the error. `resume` is a
zero-argument function that can be used to restart the operation from the
point where it failed; this is obviously only useful if the error is transient
in nature and/or can be fixed by manipulating the operation's private
namespace, e.g. by replacing an invalid `token`.

# off [(event, listener)](method.md) #

Removes the first instance, if any, of the function `listener` from the
listeners for the named `event`. Returns the [Operation](Operation.md) instance.

# on [(event, listener)](method.md) #

Adds the function `listener` to the list of listeners for the named `event`.
Returns the [Operation](Operation.md) instance.

# start [()](method.md) #

Starts the operation. Calling `start` twice for the same [Operation](Operation.md) instance
is undefined behavior.