# Motion Analysis Template

Use this template when planning motion for a page, flow or component set.

## Context

- Surface or route:
- Primary audience:
- Primary business goal:
- Primary user action:
- Existing stack:
- Existing motion patterns:
- Constraints:

## Area Template

- Area name:
- Content purpose:
- User task:
- Current friction:
- Motion goal:
- Trigger:
- Entry behavior:
- Continuous behavior:
- Exit or reset behavior:
- Desktop behavior:
- Mobile behavior:
- Reduced-motion fallback:
- Complexity:
- Implementation choice:
- Success criteria:

## Example

```md
Area name: Pricing comparison
Content purpose: Help the user compare plans quickly
User task: Scan differences and choose a plan
Current friction: Dense table, low visual hierarchy
Motion goal: Reveal comparison progressively without delaying scan
Trigger: Section enters viewport
Entry behavior: Table fades up, columns stagger lightly
Continuous behavior: None
Exit or reset behavior: None
Desktop behavior: 60ms stagger by column
Mobile behavior: No stagger, single fade-up
Reduced-motion fallback: Immediate visibility
Complexity: Low
Implementation choice: IntersectionObserver + CSS classes
Success criteria: Faster scan, CTA remains immediately visible
```
