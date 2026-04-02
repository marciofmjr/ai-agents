# Stack Selection

Choose the lightest approach that matches the interaction.

## CSS Only

Use when the effect is a hover state, focus state, microinteraction, simple entrance, accordion, tab switch or modal transition.

Good fit:
- button hover and press states
- card lift or depth shift
- simple fade, slide or scale
- component enter and exit with existing classes or states

## IntersectionObserver plus CSS

Use when motion should start on scroll but remain simple.

Good fit:
- reveal on enter
- section fade-up
- lightweight stagger
- image and copy entering together

## WAAPI or Small Vanilla JS

Use when the animation needs sequencing, interruption, state awareness or runtime values, but does not justify a new heavy dependency.

Good fit:
- coordinated component states
- timeline-lite interactions
- counters, progress cues or gesture-linked feedback

## Framework-Native Motion

Use when the codebase already relies on a built-in or established motion layer.

Examples:
- Angular animation primitives already in the app
- React codebases already using Motion or Framer Motion
- Vue transitions already integrated into the component model

## GSAP or ScrollTrigger

Use only when the experience depends on advanced scroll storytelling.

Good fit:
- scrubbed timelines
- pinned scenes
- synchronized multi-element choreography
- layered reveals with precise timing control

Do not use just to reveal text blocks or cards.

## Default Rule

If two approaches can deliver the same perceived result, choose the simpler one.
