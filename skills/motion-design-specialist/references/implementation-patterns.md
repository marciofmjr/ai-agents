# Implementation Patterns

## Reveal and Stagger

Use for sections that need clearer scan order.

- Keep offsets small.
- Stagger by groups, not every tiny child.
- Remove stagger entirely on small screens when it slows reading.

## State Transition

Use for tabs, accordions, filters, drawers and modals.

- Preserve orientation between states.
- Animate container and content with a shared rhythm.
- Keep input responsiveness more important than visual flourish.

## Scroll Emphasis

Use to highlight a key story beat while the user is already reading.

- Favor one dominant motion cue.
- Avoid combining parallax, scale and blur without a strong reason.
- Ensure the user can still scan the copy while the effect runs.

## Sticky Storytelling

Use only when the narrative truly benefits from step-by-step progression.

- Keep each step semantically meaningful.
- Provide clear exit points.
- Simplify heavily on mobile or replace with static progression.

## Feedback Motion

Use for confirmations, validation, loading and completion states.

- Reinforce system status.
- Keep loops subtle and temporary.
- End the motion once the state is understood.
