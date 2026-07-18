# Project 5: Event-Driven Smart Home Simulation

## Description

Simulate a smart home where sensors publish events and devices react: a thermostat raises temperature events, a motion sensor raises movement events, and subscribers (heating, lights, a security alarm, an activity logger) respond — without the sensors knowing any of them exist. A simulation clock advances time step by step, feeding scripted and random readings through the system. This project makes delegates, events, lambdas, and the publish/subscribe pattern concrete.

## Difficulty

**Intermediate** — estimated effort: 6–9 hours.

## Chapters Used

- 14 Delegates, Events & Lambdas
- 12 Collections & Generics
- 08–10 OOP chapters (classes, interfaces)
- 13 Exceptions (defensive event raising)

## Requirements Checklist

- [ ] A `TemperatureSensor` class exposing a `ReadingChanged` event carrying the new temperature (use `EventHandler<T>` or a custom delegate — your choice, justify in a comment)
- [ ] A `MotionSensor` class exposing a `MotionDetected` event carrying a room name and timestamp (define a custom `EventArgs` subclass for this payload)
- [ ] Sensors raise events using the null-safe `?.Invoke` pattern
- [ ] A `HeatingSystem` subscriber that turns on below a configurable threshold and off above another (with a printed state change only when the state actually flips)
- [ ] A `LightController` subscriber that turns on the light in a room upon motion and schedules it off after N ticks without motion
- [ ] A `SecurityAlarm` subscriber that is armed/disarmed; when armed, motion triggers an alert
- [ ] An `ActivityLogger` that subscribes to **all** events and records every occurrence into an in-memory `List<string>` with timestamps, printable as a report at the end
- [ ] A `SimulationClock` (or main loop) that advances in ticks; each tick feeds the sensors new values (mix of scripted values for predictability and random drift)
- [ ] At least one subscriber is attached as a **lambda**, at least one as a **method group**, and at least one is **unsubscribed mid-simulation** (e.g., the alarm gets disarmed by unsubscribing) — demonstrating you understand handler identity
- [ ] Nothing in any sensor class references any subscriber type (verify: sensors compile in isolation)
- [ ] The simulation runs a fixed number of ticks and ends with the logger's full report and summary counts per event type (use a `Dictionary<string, int>`)

## Hints

- Design order matters: write the sensors first and prove they raise events with a throwaway lambda before building real subscribers.
- Custom `EventArgs`: a small class `MotionEventArgs : EventArgs { public string Room; public DateTime At; }` — properties, constructor, done.
- Unsubscribing a lambda requires holding it in a variable first (Chapter 14 pitfall #2). Let the alarm store its own handler delegate so `Disarm()` can do `sensor.MotionDetected -= _handler;`.
- The "lights off after N ticks" logic needs the light controller to also observe clock ticks — consider giving your clock its own `Tick` event that anything can subscribe to. Suddenly your architecture is uniform.
- Keep console output prefixed per component (`[HEAT]`, `[LIGHT]`, `[ALARM]`, `[LOG]`) so interleaved reactions stay readable.
- If two subscribers must react in a fixed order, that's a design smell in pub/sub — restructure so order doesn't matter.

## Stretch Goals

- Add a `DoorSensor` and a rule that motion + door-open while armed escalates the alert level.
- Make thresholds configurable at startup from user input, validated with your Chapter 13 patterns.
- Give the logger the ability to filter its report with lambdas passed as `Func<string, bool>` (e.g., only alarm lines).
- Implement a generic `EventBus<T>` class: `Subscribe(Action<T>)`, `Publish(T)` — then route one of your event types through it and compare with C# events in a comment.
- Save the activity log to a file at the end once you've read Chapter 16.
