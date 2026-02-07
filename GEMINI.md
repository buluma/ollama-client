# Ollama App Project Overview

This document provides a concise overview of the Ollama App project, its architecture, key technologies, and development conventions to serve as a comprehensive reference for future interactions.

## Project Overview

The Ollama App is a modern and user-friendly client designed to interact with an Ollama server locally via its API. It aims to provide a seamless chat experience while keeping all data private and within the user's local network.

*   **Purpose:** To offer an intuitive client interface for interacting with local Ollama language models.
*   **Technologies:** The application is built using the **Flutter** framework and the **Dart** programming language, enabling a single codebase to run across multiple platforms.
*   **Key Features:**
    *   Interactive chat user interface.
    *   Connection to local Ollama API endpoints.
    *   Experimental voice mode for conversational interaction.
    *   Progressive Web App (PWA) support.
    *   Desktop support for Windows, Linux, and macOS platforms.
    *   Dynamic theming and localization.
*   **Core Dependencies (from `pubspec.yaml`):**
    *   `ollama_dart`: Facilitates communication with the Ollama API.
    *   `shared_preferences`: Used for local storage of user settings and chat history.
    *   `flutter_chat_ui`: Provides the foundational components for the chat interface.
    *   `speech_to_text`: Enables speech recognition for the voice mode.
    *   `flutter_tts`: Provides text-to-speech capabilities.
    *   `bitsdojo_window`: Used for custom window management and controls on desktop platforms.
    *   `pwa_install`: Adds PWA installation features for web deployments.

## Building and Running

### Prerequisites

*   **Flutter SDK:** Installed and configured.
*   **Dart SDK:** Included with Flutter SDK.
*   **Ollama Server:** A running Ollama server instance accessible from the client application.

### Installation

Pre-built binaries are available under the [releases tab](https://github.com/JHubi1/ollama-app/releases). The app can also be downloaded from various app stores as listed in the `README.md`.

### Setup

After installation, the primary setup involves configuring the Ollama host endpoint within the application's settings. This can be done via the "Settings" screen (`lib/screen_settings.dart`).

### Running (Development)

To run the application in development mode:

*   **General:** `flutter run`
*   **Specific Platform (e.g., Web):** `flutter run -d web`
*   **Specific Platform (e.g., Linux):** `flutter run -d linux`
*   **Specific Platform (e.g., Windows):** `flutter run -d windows`

### Building

To build the application for various platforms:

*   **Web:** `flutter build web`
*   **Android:** `flutter build apk` or `flutter build appbundle`
*   **iOS:** `flutter build ios`
*   **Windows:** `flutter build windows`
*   **Linux:** `flutter build linux`
*   **macOS:** `flutter build macos`

The project also contains a `scripts/build.dart` which might automate some build processes.

## Development Conventions

*   **Language:** Dart
*   **Framework:** Flutter
*   **Linting:** The project adheres to standard Flutter linting rules, as indicated by `analysis_options.yaml` including `package:flutter_lints/flutter.yaml`.
*   **Project Structure:**
    *   `lib/main.dart`: The main entry point of the application, responsible for app initialization, theming, and primary UI routing.
    *   `lib/worker/`: Contains core logic and utility functions, such as API communication (`sender.dart`), desktop-specific adaptations (`desktop.dart`), and other helper functions.
    *   `lib/screen_*.dart`: Individual UI screens of the application (e.g., `screen_settings.dart`, `screen_welcome.dart`).
    *   `lib/settings/`: Dedicated sub-screens for various configuration options within the settings.
    *   `assets/`: Stores static assets like images and icons.
    *   `l10n/`: Contains localization files (`.arb`) and generated Dart localization code.
*   **API Interaction:** All communications with the Ollama server are handled through the `ollama_dart` package, primarily orchestrated within `lib/worker/sender.dart`.
*   **State Management:** The application primarily uses Flutter's built-in `StatefulWidget`s and `Provider` (implicitly, via `SharedPreferences` for persistent settings) for managing application state.
*   **Desktop Integration:** Uses `bitsdojo_window` for enhanced desktop window control, with adaptive layouts managed by functions in `lib/worker/desktop.dart`.
