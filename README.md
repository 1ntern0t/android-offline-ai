# ⚡ ANDROID OFFLINE AI — CLEAN SETUP GUIDE
Run GGUF AI models *fully offline* on a **rooted Android device** using cross-compiled `llama.cpp` binaries.

## Requirements
Anyone following this setup **must install the Android NDK**, because the NDK provides:

---

## Demo

![Offline AI demo screenshot](IMG_8709.jpg)

[[▶️ Watch the offline AI demo](sample%20video.mp4)](https://x.com/1nternot/status/1994772971576070355/video/1)

---

- Toolchains (clang, linker, etc.)  
- Android system headers  
- CMake integration files  
- Cross-compile environment  
- `arm64-v8a` architecture support  

Without the NDK, this command **will not work**:

```bash
📦 Model Used for This Demo
I used the mistral-7b-instruct-v0 GGUF model from HuggingFace.

You can choose any GGUF model you want:
https://huggingface.co/

You MUST cross-compile your own binaries to match your device’s architecture.

1️⃣ Set Your NDK Path
Replace the directory with your own on your Linux machine:

export ANDROID_NDK=(WRITE YOUR DIRECTORY PATH HERE)/Android/Sdk/ndk/27.0.12077973
Example: export ANDROID_NDK=/home/Desktop/xirion/Android/Sdk/ndk/27.0.12077973

2️⃣ Cross-Compile llama.cpp for Android Clone the repository:
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp

Main build command:
cmake -DCMAKE_TOOLCHAIN_FILE=$ANDROID_NDK/build/cmake/android.toolchain.cmake \
      -DANDROID_ABI=arm64-v8a \
      -DANDROID_PLATFORM=android-28 \
      -DGGML_OPENMP=OFF \
      -B build-android

If that fails, use fallback:
cmake -DCMAKE_TOOLCHAIN_FILE=$ANDROID_NDK/build/cmake/android.toolchain.cmake \
      -DANDROID_ABI=arm64-v8a \
      -DANDROID_PLATFORM=android-28 \
      -DGGML_OPENMP=OFF \
      -DLLAMA_CURL=OFF \
      -B build-android

Compile the binaries:
cmake --build build-android --config Release -j$(nproc)

3️⃣ Move the Compiled Binaries After building, your files will be located in:
llama.cpp/build-android/bin/

For convenience, I moved the entire bin folder to my Desktop and renamed it:
android_llama_binaries
I also placed my .gguf model inside that folder.

4️⃣ Push Everything to Android (Root Required) Check device connection:
adb devices

Expected output example:
f32b1ec0   device

Create the directory:
adb shell "mkdir -p /data/local/tmp/llama"

Verify:
adb shell ls -l /data/local/tmp/llama

Push binaries:
adb push android_llama_binaries/* /data/local/tmp/llama/

Push your GGUF model:
adb push mistral-7b-instruct-v0.2.Q4_K_S.gguf /data/local/tmp/llama/

Confirm files:
adb shell ls -lh /data/local/tmp/llama/

5️⃣ Set Permissions (Important)
Some binaries won’t run without executable permissions:
adb shell "chmod +x /data/local/tmp/llama/llama-*"

Optional full sweep:
adb shell "chmod +x /data/local/tmp/llama/*"

6️⃣ Run llama-cli on Android Start root mode:
adb root
adb shell

Or using SU inside shell:
su cd ..

Basic test run:
LD_LIBRARY_PATH=/data/local/tmp/llama /data/local/tmp/llama/llama-cli -m /data/local/tmp/llama/mistral-7b-instruct-v0.2.Q4_K_S.gguf -p "What is the capital of Mars?"

7️⃣ Faster / Better Tuning Example
LD_LIBRARY_PATH=/data/local/tmp/llama /data/local/tmp/llama/llama-cli -m /data/local/tmp/llama/mistral-7b-instruct-v0.2.Q4_K_S.gguf --ctx-size 2048 --threads 8 --no-warmup --n-predict 128 --top-k 20 --top-p 0.9 --temp 0.7 -p "Explain dark matter in 1 sentence."
