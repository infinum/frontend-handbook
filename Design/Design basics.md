> “Good design is like a refrigerator—when it works, no one notices, but when it doesn’t, it sure stinks.” –Irene Au

A good design implementation is important for the end user experience, so we should be careful when implementing the design.

## Common mistakes

* sizes - are paddings, font sizes and line heights correct?
* responsive - does the content break in a right way?
* colors - are right colors used?
* states - are all states (active, focused, hover) implemented according to the design?
* interactions - are all animations described in the design implemented?

## Communication with designers

Making a great product is a common goal for developers and designers. When working towards this goal, some compromises might be made. This might be because of some technical limitations or because something would just take too long and wouldn't be worth it. If you notice a situation like this, talk to your designer and find the best compromise together.

## Baseline design assumptions

The design that you get should have as much details as possible, but even if something is missing, it should be assumed unless defined otherwise

### Interactions

All interactions (focus, hover, active) should be animated when switching between states. The default animation should last 0.3 seconds and have a `ease-in-out` easing.
