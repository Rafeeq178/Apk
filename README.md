# Vethos Native Android

Native Android app (Kotlin + Jetpack Compose) inspired by the uploaded Vethos UI. It does not use WebView. The local inference bridge is C++/JNI and uses llama.cpp for GGUF models.

## Features in this source build

- Native Compose screens: Home, Models, Personalities, Chat, Insights and Settings.
- Choose a `.gguf` model using Android's system file picker.
- The selected model is copied into the app's private `files/models` directory so llama.cpp can open it by path. Ensure enough free storage for a second copy of the model.
- Load/unload model to manage RAM.
- CPU inference via llama.cpp and basic token streaming-ready JNI boundary (current JNI method returns the completed response in one result).
- Built-in personalities: General Assistant, Coder, Teacher, Math Mentor, Writer, Research Pal, Finance Pro and Health Guide.

## Important limitations

- This is the full **source project**, not a prebuilt APK. Android SDK/NDK downloads and compilation are required.
- The uploaded original was a web app; the Compose UI is a native recreation of its visual style and core navigation, not a pixel-perfect one-to-one port of every web-only interaction.
- This first source build supports one single-file GGUF model. Split GGUF shards are not supported.
- Inference uses a generic ChatML-style prompt. Some models require a different chat template and may respond poorly until model-specific prompt formatting is added.
- Inference is CPU-only; GPU/Vulkan acceleration is not configured.
- llama.cpp's API changes over time. The native bridge targets the current upstream API; if upstream breaks an API, pin a compatible llama.cpp revision or adjust `native-lib.cpp`.
- A model's memory use is larger than its file size. Start with a small 1B–3B Q4 model on mobile, and avoid loading models larger than available RAM.

## Build on Termux

Use a reasonably recent Android device and enough free storage. Android SDK/NDK setup is the most difficult part of building from Termux; native compilation on a phone can take a long time.

1. Install packages:

   ```sh
   pkg update
   pkg install git openjdk-17 gradle cmake ninja python
   ```

2. Install Android SDK command-line tools and Android NDK (r27 or compatible) using your preferred SDK setup. Ensure platform 35 and build-tools are installed. Set environment variables, for example:

   ```sh
   export ANDROID_HOME="$HOME/android-sdk"
   export ANDROID_SDK_ROOT="$ANDROID_HOME"
   export ANDROID_NDK_HOME="$ANDROID_HOME/ndk/27.2.12479018"
   export PATH="$PATH:$ANDROID_HOME/platform-tools:$ANDROID_HOME/cmdline-tools/latest/bin"
   ```

3. Put this project folder in Termux storage, enter the project folder, and fetch llama.cpp:

   ```sh
   bash scripts/fetch-llama.sh
   ```

4. Build the debug APK:

   ```sh
   bash scripts/build-termux.sh
   ```

   If Gradle/SDK versions are already configured, the APK is copied to `output/Vethos-debug.apk`.

5. Install it:

   ```sh
   termux-open output/Vethos-debug.apk
   ```

   Or install with `adb install output/Vethos-debug.apk` if ADB is available.

### SDK notes

- Android Gradle Plugin 8.7.3 needs JDK 17 and Android SDK platform 35.
- Android NDK and CMake must be installed through Android SDK Manager; Termux's system CMake alone does not provide the Android NDK toolchain.
- Some devices/Termux environments cannot run a full Android SDK/NDK build locally. If the Gradle build fails before compilation, build the same project on a Linux PC or CI with Android SDK/NDK installed.

## Model suggestions

Choose a GGUF quantized model compatible with your phone, typically Q4_K_M. Start small. The app does not download model weights from Hugging Face itself; choose a GGUF file already downloaded on the phone.

## Project layout

- `app/src/main/java/com/vethos/app/MainActivity.kt` — native UI, storage picker, chat and app state.
- `app/src/main/java/com/vethos/app/LlamaBridge.kt` — JNI declarations.
- `app/src/main/cpp/native-lib.cpp` — llama.cpp model loading and generation.
- `app/src/main/cpp/CMakeLists.txt` — native build integration.
- `scripts/fetch-llama.sh` — fetch upstream llama.cpp source.
- `scripts/build-termux.sh` — build and copy APK.
