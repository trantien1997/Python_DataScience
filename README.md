```markdown
# WebRTC Build Setup for Windows


```
## 1. Create `asix` Folder
```bash
mkdir asix
```

## 2. Install depot_tools
### 2.1 Clone depot_tools from the official repository:
```bash
cd asix
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
```

### 2.2 Add depot_tools to the PATH environment variable:
- Open Environment Variables on Windows:
  - Press `Windows Key + S`, search for "Edit the system environment variables" and open it.
  - In the System Properties window, click on `Environment Variables`.
  - Find the `Path` variable under `System variables`, click `Edit`.
  - Add the path to the `depot_tools` folder you just cloned:
    ```
    C:\Users\PC\Desktop\asix\depot_tools
    ```

## 3. Install Python 3
- Visit the [Python Downloads](https://www.python.org/downloads/) page and choose Python 3.11.8.
- Install Python 3.
- Add Python to the PATH:
  - Open Environment Variables on Windows as described above.
  - Add the path to the Python folder:
    ```
    C:\Users\PC\AppData\Local\Microsoft\WindowsApps
    ```

## 4. Install Visual Studio Community 2022
- Download [Visual Studio Community](https://visualstudio.microsoft.com/visual-cpp-build-tools/) (free version).
- During installation, select the following Workloads:
  - Desktop Development with C++.
  - MSVC v143 or v142.
  - Windows 10/11 SDK.
  - CMake tools for Windows (optional).

## 5. Create Checkout Directory
```bash
cd asix
mkdir libwebrtc_build
cd libwebrtc_build
```

## 6. Create `.gclient` File
Create a `.gclient` file with the following content:
```python
solutions = [
  {
    "name"        : 'src',
    "url"         : 'https://github.com/webrtc-sdk/webrtc.git@m125_release',
    "deps_file"   : 'DEPS',
    "managed"     : False,
    "custom_deps" : {},
    "custom_vars": {},
  },
]
target_os  = ['win']
```

## 7. Synchronize Source Code
```bash
gclient sync
```

## 8. Compile Steps for libwebrtc
- Ensure you're currently in the `src` directory:
```bash
git clone https://github.com/webrtc-sdk/libwebrtc
```

## 9. Modify `src/BUILD.gn` File
Add `libwebrtc` to the `default` group in `src/BUILD.gn`:
```diff
--- a/BUILD.gn
+++ b/BUILD.gn
@@ -29,7 +29,7 @@ if (!build_with_chromium) {
   # 'ninja default' and then 'ninja all', the second build should do no work.
   group("default") {
     testonly = true
-    deps = [ ":webrtc" ]
+    deps = [ ":webrtc","//libwebrtc", ]
     if (rtc_build_examples) {
       deps += [ "examples" ]
     }
```

## 10. Set Environment Variables
```bash
set DEPOT_TOOLS_WIN_TOOLCHAIN=0
set GYP_MSVS_VERSION=2022
set GYP_GENERATORS=ninja,msvs-ninja
set GYP_MSVS_OVERRIDE_PATH=C:\Program Files\Microsoft Visual Studio\2022\Community
```

## 11. Generate Build Files
```bash
cd src
gn gen out-debug/Windows-x64 --args="target_os=\"win\" target_cpu=\"x64\" is_component_build=false is_clang=true is_debug=true rtc_use_h264=true ffmpeg_branding=\"Chrome\" rtc_include_tests=false rtc_build_examples=false libwebrtc_desktop_capture=true" --ide=vs2022
```

## 12. Build libwebrtc
```bash
ninja -C out-debug/Windows-x64 libwebrtc
```
```

This `README.md` provides clear instructions for setting up and building `libwebrtc` on Windows, formatted for easy reading and use in a project repository.
