# Implementation Plan: Stop Streaming Output Button

## Approach
This feature will introduce a new button in the chat interface that allows users to stop the generation of streaming output from the Ollama server. The core idea is to use a global flag that the streaming function checks periodically. When the button is pressed, this flag is set, causing the streaming process to terminate gracefully. This approach offers a direct and user-friendly way to manage AI responses, especially for long or undesirable outputs.

## Steps
1.  **Introduce a flag to control streaming (15 min)**
    *   **Files to modify:** `lib/main.dart`, `lib/worker/sender.dart`
    *   **Description:** Add a new global boolean variable (e.g., `_stopStreaming`) in `lib/main.dart` that defaults to `false`. This flag will be set to `true` when the stop button is pressed.
    *   **Code Change (Conceptual - `lib/worker/sender.dart`):**
        ```dart
        // Inside the stream processing loop in send() function
        await for (final res in stream) {
            if (_stopStreaming) { // Check the new flag
                // Clean up and break the stream
                break;
            }
            // Existing stream processing logic
        }
        // After the loop, reset _stopStreaming to false
        _stopStreaming = false;
        ```

2.  **Add a Stop Button to the Chat UI (20 min)**
    *   **Files to modify:** `lib/main.dart`
    *   **Description:** Implement an `IconButton` within the `AppBar`'s `actions` list in the `_MainAppState` widget.
    *   **Visibility:** This button should only be visible (`AnimatedOpacity` or conditional rendering) when an Ollama response is actively streaming (i.e., when `chatAllowed` is `false` as per current logic in `main.dart`). It should disappear once the streaming stops or is manually interrupted.
    *   **Icon and Action:** Use an appropriate icon (e.g., `Icons.stop_rounded`). The `onPressed` callback will set the `_stopStreaming` flag to `true` and trigger a UI rebuild (`setState`) to ensure the change is reflected.

3.  **Refine UI/UX and State Management (15 min)**
    *   **Files to modify:** `lib/main.dart`, `lib/worker/sender.dart`
    *   **Description:**
        *   Ensure the `_stopStreaming` flag is correctly reset to `false` in `lib/worker/sender.dart` once the stream naturally completes or is interrupted by the new button.
        *   Verify that `chatAllowed` transitions back to `true` correctly after stopping, allowing new messages to be sent.
        *   Add a tooltip to the stop button for clarity.
        *   Consider disabling the text input field while streaming to prevent new input during an active response.

4.  **Testing (20 min)**
    *   **Test cases:**
        *   Verify the stop button appears only when a message is actively streaming.
        *   Send a long query and click the stop button. Observe that the streaming output ceases and the UI returns to an interactive state.
        *   Send a query and let it complete naturally. Verify the stop button disappears.
        *   Test sending a query and immediately clicking stop.
        *   Ensure that after stopping a stream, new messages can be sent normally.

## Timeline
| Phase | Duration |
|-----------------------------|----------|
| Introduce streaming flag    | 15 min   |
| Add Stop Button to Chat UI  | 20 min   |
| Refine UI/UX and State Mgmt | 15 min   |
| Testing                     | 20 min   |
| **Total**                   | **1 hour 10 min** |

## Rollback Plan
In case of issues, the changes can be reverted by checking out the previous versions of `lib/main.dart` and `lib/worker/sender.dart` using git. The introduction of a new flag and a UI element are isolated changes that should be easy to undo without affecting other parts of the application.

## Security Checklist
-   [ ] Input validation: Not directly applicable to this feature.
-   [ ] Auth checks: Not directly applicable to this feature.
-   [ ] Rate limiting: Not directly applicable to this feature.
-   [x] Error handling: Ensure the `send` function's `catch` block for Ollama API errors gracefully handles cases where `_stopStreaming` might be active. The UI should always recover to a usable state.