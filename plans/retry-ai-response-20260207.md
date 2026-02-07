# Implementation Plan: Retry AI Response Option

## Approach
The goal is to provide a "retry" option for AI messages, allowing users to regenerate a response. The chosen approach will involve introducing a dedicated function in the backend logic (`sender.dart`) to handle the regeneration process. This function will effectively roll back the chat history to the user message preceding the message to be retried, then re-initiate the message sending process. A corresponding UI button will be added next to each AI message to trigger this function. This solution is modular, leveraging existing message sending capabilities, and provides a clear user interaction.

## Steps
1.  **Introduce a `retryMessage` function in `sender.dart` (30 min)**
    *   **Files to modify:** `lib/worker/sender.dart`
    *   **Description:**
        *   Create a new asynchronous function, `retryMessage(String messageId, Function setState)`, in `lib/worker/sender.dart`.
        *   This function will identify the `messageId` of the AI message to be retried within the global `messages` list.
        *   It will then remove the target AI message and all subsequent messages from the `messages` list.
        *   The function will locate the last remaining user message (the one immediately preceding the now-removed AI message).
        *   It will then call the existing `send` function, passing the content of this last user message to trigger a new AI response.
        *   Crucially, it will update the UI by calling the provided `setState` callback.
        *   The `send` function in `sender.dart` may need minor adjustments to accept an optional `previousMessageContent` or similar to facilitate the retry logic cleanly, or the `retryMessage` function will directly manipulate the `messages` list before calling `send`. For simplicity, `retryMessage` will manipulate `messages` and then call `send` as if it were a new user input.

2.  **Add a Retry Button to AI Messages in the Chat UI (30 min)**
    *   **Files to modify:** `lib/main.dart`
    *   **Description:**
        *   Locate the `textMessageBuilder` property within the `Chat` widget in the `_MainAppState` class.
        *   Inside this builder, for messages where `p0.author == assistant`, add an `IconButton` (e.g., using `Icons.refresh_rounded` or `Icons.replay_rounded`).
        *   The `onPressed` callback for this button will trigger the `retryMessage` function created in Step 1, passing `p0.id` (the ID of the current AI message) and `setState` from `_MainAppState`.
        *   The button should be positioned appropriately within the message bubble layout to be accessible without clutter.

3.  **Update Chat History Management (20 min)**
    *   **Files to modify:** `lib/worker/sender.dart`
    *   **Description:**
        *   Ensure that after `retryMessage` modifies the global `messages` list (by removing messages), the updated chat history is persistently saved using the `saveChat` function. This should happen immediately after `messages` is modified and before calling `send`.

4.  **Refine UI/UX and Localization (15 min)**
    *   **Files to modify:** `lib/main.dart`, `lib/l10n/app_en.arb` (and other `.arb` files if translated)
    *   **Description:**
        *   Add a clear tooltip to the retry button (e.g., "Regenerate response").
        *   Add a new key-value pair for the tooltip in `app_en.arb` (e.g., `"tooltipRetryMessage": "Regenerate response"`).
        *   Run `flutter pub get` or `flutter gen-l10n` to regenerate localization files.
        *   Ensure that the retry button is appropriately styled and doesn't interfere with other message interactions.

5.  **Testing (30 min)**
    *   **Manual Testing:**
        *   Send a user message, receive an AI response. Click the retry button. Verify the current AI message is removed, and a new one is generated.
        *   Conduct a multi-turn conversation. Click retry on an AI message that is not the last one. Verify that the target AI message and all subsequent messages are removed, and the AI regenerates its response based on the previous user message.
        *   Test the retry functionality when the AI generates an image or multimodal content.
        *   Ensure chat history is saved correctly after multiple retries across sessions.
        *   Verify that clicking retry is disabled if an AI message is currently being streamed.

## Timeline
| Phase | Duration |
|-------------------------------------|----------|
| Introduce `retryMessage` function   | 30 min   |
| Add Retry Button to AI Messages     | 30 min   |
| Update Chat History Management      | 20 min   |
| Refine UI/UX and Localization       | 15 min   |
| Testing                             | 30 min   |
| **Total**                           | **2 hours 5 min** |

## Rollback Plan
Rollback involves reverting changes made to `lib/worker/sender.dart`, `lib/main.dart`, and the localization files (`.arb` and generated `.dart`). These changes are relatively self-contained to the chat message handling and should not affect other parts of the application if reverted using Git.

## Security Checklist
-   [ ] Input validation: Not directly applicable as it re-uses existing validated messages.
-   [ ] Auth checks: Not applicable.
-   [ ] Rate limiting: Re-sending messages increases API calls. Ensure existing `ollama_dart` client handles rate limiting gracefully or consider adding retry-specific rate limiting if this becomes an issue.
-   [x] Error handling: Ensure the `retryMessage` function includes robust error handling for API calls, similar to the `send` function, and updates the UI state accordingly if the regeneration fails.