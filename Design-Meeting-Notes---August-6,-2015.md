# Default Query Tracking Behavior

We attempted to answer the question: "Is it still appropriate to track query results by default, given that in many applications SaveChanges will never be called on the context. E.g. A GET action in a web app."

- We noted that tracking query results significantly reduces query efficiency. And,
- We agreed that it is likely that customers are often tracking query results unnecessarily. But,
- A majority in the room felt that we should still track by default because it is more predictable and inline with existing behavior. However,
- We want to make it easier for specific apps to opt-in to different tracking behavior. So,
- We will add a new top level DbContext option that controls whether or not query results are tracked by default:

enum QueryTrackingBehavior (Tracked, NoTracking) // default is Track

- Accordingly we will add an AsTracked query operator to complement AsNoTracking.

# Reverse Engineering SQLite Column Types

Content coming soon...

# Discussion

Please use the [discussion issue](https://github.com/aspnet/EntityFramework/issues/2791) to provide feedback, ask questions, etc.