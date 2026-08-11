# Chat UI Auto-Scroll and New Message Indicator

**Originating Issue:** [#20 — Automatic Scrolldown in Agent Chats for progress message + New Message Hint](https://github.com/onecx/tasks/issues/20)
**Target Repository:** `onecx-chat-ui` (https://github.com/onecx/onecx-chat-ui)
**Date:** 2026-08-11

---

## 1. Scroll-State Behavior

### Auto-Scroll Rules

1. **User at bottom** — When the user's scroll position is at (or very near) the bottom of the message list, the chat container must automatically scroll to the bottom whenever new content arrives. This applies to:
   - Incoming assistant/agent messages (including streaming responses).
   - Progress/loading messages that appear immediately after the user sends a message.
   - Incoming human-to-human messages.

2. **User scrolled up** — When the user has manually scrolled away from the bottom, the chat container must NOT force-scroll. The user retains full control of their scroll position.

3. **Smooth scrolling** — All automated scroll actions must use smooth scroll transitions (e.g., `scroll-behavior: smooth` or equivalent programmatic smooth scrolling via `scrollTo({ behavior: 'smooth' })`). No jarring instant jumps.

4. **"At bottom" detection** — The threshold for "at bottom" should be forgiving (e.g., within 100px of the scroll end) so that minor layout shifts or content expansion do not trigger the "scrolled up" state unexpectedly.

---

## 2. Progress / Loading Message Visibility

1. **Immediate visibility** — In agent chats, the loading/progress message must become visible to the user immediately after they send a message. If auto-scroll is active (user was at bottom), the view must scroll down to reveal it.

2. **Persistence** — The loading/progress message remains in view until it is replaced by the actual assistant response. It must not disappear prematurely.

3. **Replacement** — When the assistant response arrives, the loading/progress message is replaced with the actual message content. The scroll position should be maintained naturally (no forced scroll if user has scrolled up).

---

## 3. Unread Indicator Behavior

### Appearance

- Show a circular indicator positioned at the bottom of the chat area (floating, non-intrusive) when:
  - The user is scrolled up (not at the bottom), AND
  - One or more new messages have arrived since the user last scrolled to the bottom.

- The indicator must include:
  - A **down-arrow icon** centered inside the circle.
  - An **animated border** with a modern color effect (e.g., a rotating gradient or pulsing glow).
  - An **unread message count** badge displayed adjacent to or integrated into the indicator, showing how many new messages are unread.

### Dismissal Rules

1. **Manual scroll to bottom** — The indicator must disappear automatically when the user scrolls to the bottom of the chat (or within the "at bottom" threshold).

2. **Indicator click** — Clicking the indicator must:
   - Smoothly scroll the chat container to the bottom.
   - Disappear immediately after the scroll completes (or concurrently with the scroll animation).

3. **No messages** — If no unread messages exist (all messages have been viewed), the indicator must not render at all.

---

## 4. Human-to-Human Chat Behavior

1. **No loading message** — Unlike agent chats, human-to-human chats do not display a loading/progress message. This section applies only to non-agent chat types.

2. **Auto-scroll when at bottom** — If the user is at the bottom, new messages should trigger automatic smooth scrolling to reveal them.

3. **Unread indicator when scrolled up** — If the user has scrolled up, the unread indicator (per Section 3) must appear for new messages.

---

## 5. Implementation Constraints

1. **Structured component reuse** — Reuse existing chat components and utility functions where possible. New components (e.g., the unread indicator) should be self-contained and follow the existing component architecture.

2. **Responsive layout** — The implementation must work correctly across all supported viewport sizes. The unread indicator position and size should adapt to smaller screens without overlapping message content or controls.

3. **Smooth automated scrolling** — All scroll actions must be smooth. No instant `scrollTop` assignments without a smooth transition.

4. **No regression to existing message rendering** — The auto-scroll and indicator features must not alter the visual appearance, layout, or interaction behavior of existing message rendering. Only scroll behavior and the new indicator are affected.

5. **Chat type detection** — The implementation must correctly distinguish between agent chats and human-to-human chats to apply the appropriate behavior (loading message visibility vs. no loading message).

---

## 6. Acceptance Criteria

- [ ] **AC1:** When the user is at the bottom of an agent chat and sends a message, the chat scrolls down smoothly to reveal the user's message.
- [ ] **AC2:** When the user is at the bottom of an agent chat and sends a message, the loading/progress message becomes immediately visible via auto-scroll.
- [ ] **AC3:** The loading/progress message remains visible until replaced by the assistant response.
- [ ] **AC4:** When the user is scrolled up in an agent chat and new messages arrive, the chat does NOT force-scroll.
- [ ] **AC5:** When the user is scrolled up and new messages arrive, the unread indicator (circular down-arrow with animated border and count badge) appears at the bottom of the chat area.
- [ ] **AC6:** Clicking the unread indicator smoothly scrolls the chat to the bottom and causes the indicator to disappear.
- [ ] **AC7:** Scrolling manually to the bottom of the chat causes the unread indicator to disappear.
- [ ] **AC8:** In a human-to-human chat, new messages trigger auto-scroll when the user is at the bottom.
- [ ] **AC9:** In a human-to-human chat, the unread indicator appears when the user is scrolled up and new messages arrive.
- [ ] **AC10:** All automated scroll transitions use smooth scrolling (no instant jumps).
- [ ] **AC11:** The unread indicator displays the correct count of unread messages.
- [ ] **AC12:** The implementation is responsive — the indicator and scroll behavior work correctly on mobile, tablet, and desktop viewports.
- [ ] **AC13:** Existing message rendering (styling, layout, interactions) is not affected by the auto-scroll or indicator changes.

---

## 7. Verification Workflow

Execute the following steps in the `onecx-chat-ui` repository after implementing the code changes:

1. **Install dependencies:** Run the project's dependency installation command (e.g., `npm install` or `npm ci`).

2. **Run unit tests:** Execute the full test suite (`npm test` or equivalent) and confirm all tests pass, including any new tests added for the auto-scroll and unread indicator behavior.

3. **Manual UI verification — Agent chat (at bottom):**
   - Open an agent chat.
   - Ensure the view is at the bottom.
   - Send a message.
   - Confirm: the view scrolls down smoothly to show the message and the loading/progress indicator becomes visible.
   - Confirm: the loading indicator is replaced by the assistant response when it arrives.

4. **Manual UI verification — Agent chat (scrolled up):**
   - Open an agent chat and scroll up to view earlier messages.
   - Send a message.
   - Confirm: the view does NOT force-scroll.
   - Confirm: the unread indicator appears at the bottom with the correct count.
   - Click the indicator and confirm: the view scrolls to the bottom smoothly and the indicator disappears.

5. **Manual UI verification — Human-to-human chat (at bottom):**
   - Open a human-to-human chat with the view at the bottom.
   - Receive (or send) a new message.
   - Confirm: the view scrolls down smoothly to reveal the new message.

6. **Manual UI verification — Human-to-human chat (scrolled up):**
   - Open a human-to-human chat and scroll up.
   - Receive a new message.
   - Confirm: the unread indicator appears with the correct count.
   - Scroll manually to the bottom and confirm: the indicator disappears.

7. **Responsive verification:**
   - Resize the browser window to mobile, tablet, and desktop sizes.
   - Confirm: the unread indicator is visible and functional at all viewport sizes.
   - Confirm: no layout breakage or overlap occurs.

8. **Regression check:**
   - Browse existing chat conversations.
   - Confirm: message rendering, styling, and interactions are unchanged.
