# Accessibility Guidelines

The source specs set concrete rules but name no standard. Proposed: meet **WCAG 2.2 level AA** and verify each release.

## Rules from the specs
- Tap targets 48 px or larger; Counter tap targets 48 px or larger with Pay at the bottom.
- Text scales to the largest system size; layouts do not clip.
- Readable contrast in light and dark themes; colour never carries meaning alone (status chips have an icon or word).
- Every rail icon has a screen-reader name matching its caption; captions stay visible on touch screens (tooltips do not work there).
- Keyboard: each list is one tab stop and arrow keys move inside it; Tab order is rail → panel → body with a visible focus ring; Enter opens a row; Esc closes the top layer; single-key shortcuts work only when no field has focus.
- Skip links "Skip to list" and "Skip to record" are the first tab stops; when a row loads a screen reader hears the record's title.
- Masked values announce "masked, ending 1234"; a reveal announces "revealed" and the screen says "Reveal is logged".
- Messages are plain and name the next step.

## Release check
A screen-reader pass (TalkBack, VoiceOver, and a desktop reader) and the largest text size on every release, in addition to the 48 px targets. Golden screenshots cover seven widths in light and dark.

## Open
Choose the reader list per platform; add a contrast check to the design-token package.
