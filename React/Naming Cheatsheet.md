## Naming Components

### HC/LC/b/p/s Pattern
There is a useful pattern to follow when naming functions:

```
high context (HC)? + low context? (LC)? + base + part? + suffix?
```

Take a look at how this pattern may be applied in the table below.

| Name                   | High context (HC) | Low context (LC) | Base name      | Composite Part | Suffix      |
| ---------------------- | ----------------- | ---------------- | -------------- | -------------- | ----------- |
| `Card`                 |                   |                  | `Card`         |                |             |
| `CardImage`            |                   |                  | `Card`         | `Image`        |             |
| `EventCard`            |                   | `Event`          | `Card`         |                |             |
| `UpcomingEventCard`    | `Upcoming`        | `Event`          | `Card`         |                |             |
| `EventCardFallback`    |                   | `Event`          | `Card`         |                | `Fallback`  |

> **Note:** Avoid names that includes two base names, e.g. `ButtonLink` where `Button` is used as a LC and `Link` as a base. It's wrong because it encapsulate two concerns, appearance (How ti looks) and behavioral (How it reacts to user input). You can avoid this by using polymorphic `as` prop, e.g. `<Button as="a" />`.
