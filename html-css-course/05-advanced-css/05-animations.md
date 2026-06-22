# Animations

Transitions are great for animating a single change, like a hover color. But what if you want something to move on its own, loop forever, or step through several stages? That's where **CSS animations** shine. With `@keyframes`, you describe a sequence of steps, and the browser plays it. Spinners, fade-ins, pulses, and bounces all come from this one tool.

## What You'll Learn

- How to define a sequence of steps with `@keyframes`
- The difference between `from`/`to` and percentage keyframes
- All the `animation-*` properties and the handy shorthand
- How to loop an animation forever
- How to build a spinner, a fade-in, a pulse, and a bounce
- How to respect users who prefer less motion
- When to use a transition versus an animation

## Defining Keyframes

An **`@keyframes`** rule is a named sequence of styles. It describes what the element should look like at different points during the animation. You give it a name, then list the steps inside.

```css
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
```

This defines an animation called `fadeIn` that goes from fully transparent to fully visible. By itself, `@keyframes` does nothing. You have to attach it to an element with the `animation` property.

```css
.box {
  animation: fadeIn 1s ease;
}
```

Now `.box` fades in over one second when it appears.

## `from`/`to` vs Percentage Keyframes

There are two ways to write the steps.

**`from` and `to`** mark the start and end. They are shortcuts for `0%` and `100%`. Use them for simple two-stage animations.

```css
@keyframes slideIn {
  from { transform: translateX(-100%); }
  to   { transform: translateX(0); }
}
```

**Percentages** let you add stops in between for multi-stage animations. `0%` is the start, `100%` is the end, and you can place any number of points along the way.

```css
@keyframes bounce {
  0%   { transform: translateY(0); }
  50%  { transform: translateY(-30px); }
  100% { transform: translateY(0); }
}
```

This goes up at the halfway point and back down, creating a bounce. Percentages give you full control over the journey.

## The Animation Properties

The `animation` shorthand is built from several properties. Here's what each one does.

### `animation-name`

The name of the `@keyframes` rule to play.

```css
.box { animation-name: fadeIn; }
```

### `animation-duration`

How long one cycle takes, in `s` or `ms`. Required — without it, the animation has zero length and never appears.

```css
.box { animation-duration: 2s; }
```

### `animation-timing-function`

The pace of the motion, the same values you learned for transitions: `ease`, `linear`, `ease-in-out`, `cubic-bezier(...)`. For a smooth spinner, use `linear` so it turns at a steady speed.

```css
.box { animation-timing-function: linear; }
```

### `animation-delay`

How long to wait before starting.

```css
.box { animation-delay: 0.5s; }
```

### `animation-iteration-count`

How many times to play. A number, or `infinite` to loop forever.

```css
.spinner { animation-iteration-count: infinite; }
```

### `animation-direction`

Which way to play each cycle.

- `normal` — start to end each time.
- `reverse` — end to start.
- `alternate` — forward, then backward, then forward... great for smooth back-and-forth motion.
- `alternate-reverse` — the same, starting backward.

```css
.pulse { animation-direction: alternate; }
```

### `animation-fill-mode`

What styles the element keeps **before** and **after** the animation runs.

- `none` — no leftover styles (the default).
- `forwards` — keep the final keyframe's styles after it ends. Useful so things don't snap back.
- `backwards` — apply the first keyframe's styles during any delay.
- `both` — do both.

```css
.box { animation-fill-mode: forwards; }
```

### `animation-play-state`

Whether the animation is `running` or `paused`. You can pause on hover, for example.

```css
.spinner:hover { animation-play-state: paused; }
```

## The Animation Shorthand

Writing all those properties separately is a lot. The **`animation` shorthand** packs them into one line. A common order is: name, duration, timing-function, delay, iteration-count, direction, fill-mode.

```css
.box {
  animation: fadeIn 1s ease 0s 1 normal forwards;
}
```

In practice you only include what you need:

```css
.spinner {
  animation: spin 1s linear infinite;
}
```

That reads as: play `spin`, one second per turn, steady speed, forever.

## Examples

### Spinner

A classic loading spinner. A ring that rotates endlessly.

```css
.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid #e5e7eb;
  border-top-color: #2563eb;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
```

Only the top of the border is colored, so as the whole ring spins, you see a moving arc. `linear` keeps it perfectly smooth, and `infinite` keeps it going.

### Fade-In

Make content appear gently as the page loads.

```css
.fade-in {
  animation: fadeIn 0.6s ease forwards;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to   { opacity: 1; transform: translateY(0); }
}
```

This fades in while sliding up slightly. `forwards` makes it stay visible at the end.

### Pulse

A soft "breathing" effect that draws attention, good for a notification dot or a call-to-action.

```css
.pulse {
  animation: pulse 1.2s ease-in-out infinite alternate;
}

@keyframes pulse {
  from { transform: scale(1);   opacity: 1;   }
  to   { transform: scale(1.1); opacity: 0.7; }
}
```

With `alternate`, it grows and fades, then shrinks and brightens, smoothly back and forth forever.

### Bounce

A playful bounce, looping forever.

```css
.ball {
  width: 40px;
  height: 40px;
  background: #f59e0b;
  border-radius: 50%;
  animation: bounce 1s ease-in-out infinite;
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50%      { transform: translateY(-40px); }
}
```

Notice `0%, 100%` share the same style — the ball is on the ground at the start and end of each cycle, and at its peak in the middle.

## Accessibility: `prefers-reduced-motion`

Some people get dizzy or uncomfortable from on-screen motion. Operating systems have a "reduce motion" setting, and you should respect it. The **`prefers-reduced-motion`** media query lets you turn animations off for those users.

```css
.fade-in {
  animation: fadeIn 0.6s ease forwards;
}

@media (prefers-reduced-motion: reduce) {
  .fade-in {
    animation: none;
  }
}
```

A common, kind approach is to disable or shorten all animations at once:

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

This is one of the few good uses of `!important` — making sure your accessibility override always wins. Always consider this for looping or large motion effects.

## Transitions vs Animations: Which to Use?

Both create motion, but they fit different jobs.

**Use a transition when:**

- You're reacting to a state change like `:hover`, `:focus`, or a class toggle.
- The motion goes between exactly two states (normal and changed).
- You want simple, "respond to the user" smoothness.

**Use an animation when:**

- You need motion to start on its own, with no user interaction (a fade-in on load).
- You want it to **loop** (a spinner, a pulse).
- You need **multiple steps** (a bounce that goes up then down).
- You want fine control over timing across several keyframes.

A simple way to remember it: a **transition** is a one-way trip triggered by a change; an **animation** is a self-running routine you can loop and choreograph.

## Common Mistakes

- Forgetting `animation-duration`, so nothing happens (it defaults to zero).
- Forgetting `infinite` when you wanted a loop, so it plays only once.
- Expecting the element to keep its end state without `animation-fill-mode: forwards`.
- Animating `width`/`top`/`left` instead of `transform`, causing jank.
- Ignoring `prefers-reduced-motion`, which can make some users uncomfortable.
- Using a heavy animation where a simple `:hover` transition would do.

## Exercises

1. Build a loading spinner using `@keyframes` and `animation: spin 1s linear infinite`. Then pause it on hover with `animation-play-state`.
2. Create a fade-in that also slides up, using percentage or `from`/`to` keyframes, and keep the final state with `forwards`.
3. Make a pulsing notification dot using `scale` and `opacity` with `infinite alternate`.
4. Animate a bouncing ball using `0%, 50%, 100%` keyframes and `ease-in-out` timing.
5. Add a `prefers-reduced-motion: reduce` media query that disables your animations, and test it by turning on "reduce motion" in your system settings.

## Key Takeaways

- `@keyframes` defines a named sequence of styles to animate through.
- Use `from`/`to` for two stages, or percentages for multi-step animations.
- The `animation` properties control name, duration, timing, delay, repeats, direction, fill, and play state.
- The `animation` shorthand combines them; `infinite` loops forever.
- `animation-fill-mode: forwards` keeps the final state after the animation ends.
- Spinners, fade-ins, pulses, and bounces are all built from these pieces.
- Respect `prefers-reduced-motion` to keep your site comfortable for everyone.
- Use transitions for state changes; use animations for self-running, looping, or multi-step motion.

---

[← Back to Course Index](../README.md) | [Next: Organizing Your CSS →](../06-professional-skills/01-organizing-css.md)
