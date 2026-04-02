# Motion Principles

## Purpose

- Use motion to clarify hierarchy, causality, feedback and narrative.
- Make the interface feel intentional, not busy.
- Treat motion as part of UX and brand tone, not decoration.

## Pacing

- Favor short, readable entrances.
- Make exits faster than entrances when possible.
- Use stagger to guide scanning, not to delay reading.
- Keep continuous motion rare and low-amplitude.

## Spatial Logic

- Maintain directional consistency.
- Tie movement to layout and interaction logic.
- Avoid arbitrary diagonals or exaggerated travel distance.
- Preserve orientation during state changes and crossfades.

## Mobile

- Reduce distance, layers and simultaneous movement.
- Re-check every scroll effect on touch devices.
- Assume hover does not exist.

## Accessibility

- Support `prefers-reduced-motion`.
- Keep information hierarchy intact even when animation is removed.
- Avoid flashes, aggressive zooms and motion that competes with reading or focus.

## Performance

- Prefer `transform` and `opacity`.
- Be cautious with `filter`, `backdrop-filter`, `box-shadow`, `clip-path` and expensive masks.
- Avoid scroll-linked work on the main thread unless the effect truly requires it.
- Use `will-change` sparingly and only near the interaction window.
