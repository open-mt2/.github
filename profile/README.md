# OpenMT2 Project

## Disclaimer

This project is a personnal "sandbox" project that have no real goal but to test limits of the current state of Agentic AI for big codebase modifications. It has not goal to become a stable, production projet. So yes, usage of AI for code analysis and evolution is assumed, at least for the first big refactoring stages.

The project main purpose is to port the game client to linux and modernize it's architecture and build processes to better and more maintainable code. see the Procedure and Specification for more details about it.

## Client Packages

[![EterBase CI](https://github.com/open-mt2/pkg_eterbase/actions/workflows/ci.yml/badge.svg)](https://github.com/open-mt2/pkg_eterbase/actions)
[![EterLocale CI](https://github.com/open-mt2/pkg_eterlocale/actions/workflows/ci.yml/badge.svg)](https://github.com/open-mt2/pkg_eterlocale/actions)
[![EterPack CI](https://github.com/open-mt2/pkg_eterpack/actions/workflows/ci.yml/badge.svg)](https://github.com/open-mt2/pkg_eterpack/actions)
[![EterImageLib CI](https://github.com/open-mt2/pkg_eterimagelib/actions/workflows/ci.yml/badge.svg)](https://github.com/open-mt2/pkg_eterimagelib/actions)
[![SphereLib CI](https://github.com/open-mt2/pkg_spherelib/actions/workflows/ci.yml/badge.svg)](https://github.com/open-mt2/pkg_spherelib/actions)
[![CWebBrowser CI](https://github.com/open-mt2/pkg_cwebbrowser/actions/workflows/ci.yml/badge.svg)](https://github.com/open-mt2/pkg_cwebbrowser/actions)
[![EterLib CI](https://github.com/open-mt2/pkg_eterlib/actions/workflows/ci.yml/badge.svg)](https://github.com/open-mt2/pkg_eterlib/actions)
[![MilesLib CI](https://github.com/open-mt2/pkg_mileslib/actions/workflows/ci.yml/badge.svg)](https://github.com/open-mt2/pkg_mileslib/actions)
[![EterGrnLib CI](https://github.com/open-mt2/pkg_etergrnlib/actions/workflows/ci.yml/badge.svg)](https://github.com/open-mt2/pkg_etergrnlib/actions)
[![SpeedTreeLib CI](https://github.com/open-mt2/pkg_speedtreelib/actions/workflows/ci.yml/badge.svg)](https://github.com/open-mt2/pkg_speedtreelib/actions)
[![PRTerrainLib CI](https://github.com/open-mt2/pkg_prterrainlib/actions/workflows/ci.yml/badge.svg)](https://github.com/open-mt2/pkg_prterrainlib/actions)
[![EffectLib CI](https://github.com/open-mt2/pkg_effectlib/actions/workflows/ci.yml/badge.svg)](https://github.com/open-mt2/pkg_effectlib/actions)
[![ScriptLib CI](https://github.com/open-mt2/pkg_scriptlib/actions/workflows/ci.yml/badge.svg)](https://github.com/open-mt2/pkg_scriptlib/actions)
[![EterPythonLib CI](https://github.com/open-mt2/pkg_eterpythonlib/actions/workflows/ci.yml/badge.svg)](https://github.com/open-mt2/pkg_eterpythonlib/actions)
[![GameLib CI](https://github.com/open-mt2/pkg_gamelib/actions/workflows/ci.yml/badge.svg)](https://github.com/open-mt2/pkg_gamelib/actions)
[![UserInterface CI](https://github.com/open-mt2/pkg_userinterface/actions/workflows/ci.yml/badge.svg)](https://github.com/open-mt2/pkg_userinterface/actions)

# Linux Port Procedure & Specifications (Hybrid C++ -> Rust Strategy)

This document is the authoritative specification and progress tracker for porting and modernizing the Metin2 client on Linux.

## Staged Evolution Strategy (Option A: Pragmatic Hybrid)

To guarantee a reliable, testable, and robust end result without introducing untestable regressions, the project follows a two-stage hybrid migration path:

1. **Stage 1 — Linux C++ Baseline**:
   * Port the existing C++ codebase to 64-bit Linux ELF using CMake, Ninja, SDL2, and DXVK-Native (Vulkan).
   * Goal: Achieve a fully functional, verifiable game client running natively on Linux that connects to the server, loads maps, plays audio, and executes UI scripts. This establishes the essential "ground truth" reference.

2. **Stage 2 — Incremental Rust Migration**:
   * Progressively rewrite modular, self-contained packages into high-performance, memory-safe Rust crates (`cargo` / `corrosion-rs`), linked into the client via C-FFI:
     * **`EterPack` $\rightarrow$ `eterpack-rs`**: Memory-safe Virtual File System, zero-copy archive extraction, LZO/zstd decompression, TEA/AES decryption.
     * **`MilesLib` $\rightarrow$ `miles-rs`**: Modern cross-platform audio engine (built on `rodio` / `kira` / `cpal`).
     * **Network Packet Handling $\rightarrow$ `netproto-rs`**: Type-safe packet parsing eliminating buffer overflows and malformed packet crashes.
   * Progressively evaluate rewriting higher-level engine systems into Rust once the underlying foundation is secure.

Python 2.7 UI dependencies remain intact during Stage 1 to preserve UI functionality, and may be modernized or replaced in later phases.

---

## Confirmed Target Architecture & Decisions

* **Target Architecture**: 64-bit Linux ELF (`x86_64`).
* **Graphics Backend**: Vulkan via DXVK-Native (compiles Direct3D 9 API calls directly into Vulkan on Linux).
* **Audio Engine**: Modern cross-platform backend (Stage 1: miniaudio/SDL2_mixer; Stage 2: `miles-rs` via `rodio`/`kira`).
* **Granny 3D (`EterGrnLib`)**: Open-source `.gr2` parser/loader solution.
* **Window & Input**: SDL2 replacing Win32 `MSWindow` and DirectInput.
* **Build System**: Modular CMake + Ninja, integrated with Cargo for Rust crates.
* **Package Standard**: Modern Pitchfork layout (`include/<Pkg>/`, `src/`, `tests/`), standalone CMake + GitHub Actions CI matrix (GCC & Clang, Debug & Release), and CTest unit suites.

---

## Package Porting Progress by Layer

**Overall Progress**: **17 / 18** packages completed (**100% of Tier 1 Base Layer**, **100% of Tier 2 Core Engine Layer**, **100% of Tier 3 Layer**, **1 / 2 of Tier 4 Application Layer**, **22/22 CTest suites passing**).

### Tier 1 — Base & Foundation Layer (7 / 7 Completed)

Self-contained foundational utilities, math, VFS, image decoders, and OS compatibility shims with zero dependencies on the rendering or scripting engine.

| Package | Standalone Repo | Description | Status | Tests & CI | Key Porting Notes & Completed Actions |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **[`Platform`](file:///home/straut/storage/dev/mt2client_linux/src/Platform)** | Bundled in each pkg | Win32 scalar types (`DWORD`, `HANDLE`, `RECT`), POSIX `CRITICAL_SECTION` (`pthread_mutex_t`), `CreateDirectory`, and `DirectXMathCompat.h` (`D3DXVECTOR3`, `D3DXMATRIX`, `D3DXPLANE`). | **Completed** | Verified across all suites | Header-only compatibility layer (`Platform.h` & `DirectXMathCompat.h`) bundled into each standalone package for dual standalone/monorepo builds. |
| **[`EterBase`](file:///home/straut/storage/dev/mt2client_linux/src/EterBase)** | `pkg_eterbase` | Core data structures, TEA cipher, CRC32, LZO compression, `MappedFile` (`mmap`), `FileDir`, monotonic `Timer`, `Random`, `Debug`, and `Poly` math parser. | **Completed** | 7 CTest suites + GH Actions CI | Ported `MappedFile` to POSIX `mmap()`, `Timer` to `steady_clock`, `FileDir` to `std::filesystem`, `error.cpp` to `backtrace`. Fixed 64-bit `uint32_t` TEA words, `intptr_t` `Singleton`, and 64-bit `lzo_uint` stack smashing in `CLZObject`. |
| **[`EterLocale`](file:///home/straut/storage/dev/mt2client_linux/src/EterLocale)** | `pkg_eterlocale` | Internationalization, codepage translation (CP949, CP1250, CP1252, Shift-JIS, UTF-8, Vietnamese CP1258), Arabic/Japanese shaping. | **Completed** | `test_locale` + GH Actions CI | Implemented POSIX `iconv`-based `MultiByteToWideChar` & `WideCharToMultiByte` supporting all Metin2 codepages plus custom Vietnamese CP1258 codecs. |
| **[`EterPack`](file:///home/straut/storage/dev/mt2client_linux/src/EterPack)** | `pkg_eterpack` | Virtual File System (`CEterPackManager`) supporting `FOLDER` and `ZIP` (`libzip`) asset containers and `d:/ymir work/` virtual path mapping. | **Completed** *(Stage 2 Rust target)* | `test_eterpack` + GH Actions CI | Ported `Folder` to `std::filesystem` with `realPathMap` for case-insensitive virtual lookups on Linux filesystems. Normalized `\` $\rightarrow$ `/` separators; removed dead `EterPackPolicy_CSHybridCrypt`. |
| **[`EterImageLib`](file:///home/straut/storage/dev/mt2client_linux/src/EterImageLib)** | `pkg_eterimagelib` | 2D ARGB surface manipulation (`CImage`), TGA uncompressed/RLE codec (`CTGAImage`), and DDS (`DXT1`/`DXT3`/`DXT5`) decompressor (`CDXTCImage`). | **Completed** | `test_image` + GH Actions CI | Replaced `<d3d.h>` with packed 124-byte `DDSURFACEDESC2` (`static_assert` verified on 64-bit). Fixed 64-bit `(DWORD)` pointer truncation in `DecompressDXT1/3/5` (`uintptr_t`) and `CTGAImage` RLE bounds. |
| **[`SphereLib`](file:///home/straut/storage/dev/mt2client_linux/src/SphereLib)** | `pkg_spherelib` | 3D bounding spheres (`Sphere`), frustum culling (`Frustum`), and hierarchical spatial tree (`SpherePack`, `SpherePackFactory`). | **Completed** | `test_sphere` + GH Actions CI | Refactored into Pitchfork layout with self-contained `DirectXMathCompat.h`, guarded MSVC `#pragma warning` directives, and verified ray-sphere & range queries. |
| **[`CWebBrowser`](file:///home/straut/storage/dev/mt2client_linux/src/CWebBrowser)** | `pkg_cwebbrowser` | In-game web view C interface (`WebBrowser_Startup`, `Show`, `Move`, `Hide`, `IsVisible`, `GetRect`). | **Completed** | `test_webbrowser` + GH Actions CI | Replaced 1,700-line Win32 IE ActiveX (`IWebBrowser2`) control with portable state-tracking implementation. |

---

### Tier 2 — Core Engine & Subsystems Layer (5 / 5 Completed)

Low-level rendering, windowing, input, networking, audio, 3D skeletal animation, foliage, and terrain subsystems.

| Package | Standalone Repo | Description | Status | Dependencies | Planned Porting Actions |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **[`EterLib`](file:///home/straut/storage/dev/mt2client_linux/src/EterLib)** | `pkg_eterlib` | Core engine library: D3D9 device (`GrpDevice`), `StateManager`, shaders/buffers, fonts, sprites, camera, collision, window/input (`MSWindow`), and network sockets (`NetStream`). | **Completed** (`EterLib_SmokeTests` + GH Actions CI) | Tier 1 + `libjpeg`, `Threads`, `SDL2`, `OpenGL` | 1. Ported threads/semaphores (`_beginthreadex`, `CreateSemaphore`) & Berkeley sockets (`NetStream`, `NetDatagram`, `select(m_sock + 1, ...)`).<br>2. Fixed 64-bit `CPooledObject::operator new(size_t)`, `GetWindowLongPtr`/`SetWindowLongPtr`, and `CRay::operator=`.<br>3. Guarded Win32 IMM32/TSF COM in `CIME` while preserving cross-platform text/tag handling.<br>4. Implemented native SDL2 windowing and OpenGL 2D/3D rendering pipeline (`SDL_GLContext`, hardware DXT1/3/5 decompression, dynamic camera/view/projection matrices, `glClipControl` depth clipping, FVF stream unpacking, depth test/mask, alpha blend/test, culling, and texture stage transforms). |
| **[`MilesLib`](file:///home/straut/storage/dev/mt2client_linux/src/MilesLib)** | `pkg_mileslib` | Audio engine (`CSoundManager`, 2D/3D positional audio, BGM streaming) originally wrapping RAD Miles Sound System (`MSS32.DLL`). | **Completed** (`MilesLib_SmokeTests` + GH Actions CI, *Stage 2 Rust target*) | `EterBase`, `EterPack`, `EterLib` | 1. **Stage 1**: Replaced proprietary `<miles/MSS.H>` / `MSS32.DLL` dependency with cross-platform `MSSCompat.h` backed by `miniaudio`, supporting VFS callbacks (`CMappedFile`), 2D/3D spatial audio samples, and MP3 BGM streaming while preserving `CSoundManager`.<br>2. **Stage 2**: Rewrite in Rust (`miles-rs` via `rodio`/`kira`). |
| **[`EterGrnLib`](file:///home/straut/storage/dev/mt2client_linux/src/EterGrnLib)** | `pkg_etergrnlib` | 3D character/monster mesh, bone hierarchy, skinning, and skeletal animation library wrapping Granny 3D (`.gr2`). | **Completed** (`EterGrnLib_SmokeTests` + GH Actions CI) | `EterBase`, `EterPack`, `EterLib` | 1. Removed proprietary `granny2.dll` / `granny24.lib` / `granny29.lib` dependency via clean-room 64-bit `GrannyCompat.h`.<br>2. Implemented 4×4 bone hierarchy `WorldPose` math, animation control clock/blending, and 4-weight CPU Linear Blend Skinning (`GrannyDeformVertices`).<br>3. Implemented native Oodle1 decompression (`Gr2Decompress.c`) and `.gr2` container parser (`Gr2Reader.cpp`), verified on real Metin2 models (rigid meshes, deformable skinned meshes, skeletons, animations, and materials). |
| **[`SpeedTreeLib`](file:///home/straut/storage/dev/mt2client_linux/src/SpeedTreeLib)** | `pkg_speedtreelib` | Outdoor vegetation and foliage rendering library (`CSpeedTreeForest`, `SpeedTreeWrapper`) wrapping SpeedTreeRT. | **Completed** (`SpeedTreeLib_SmokeTests` + GH Actions CI) | `EterBase`, `EterPack`, `EterLib` | 1. Removed proprietary `SpeedTreeRT.dll` / `SpeedTreeRT.lib` dependency via clean-room 64-bit `SpeedTreeRTCompat.h`.<br>2. Implemented SPT texture scanning (bark, composite map, self-shadow, leaves, fronds) and trunk collision cylinder generation.<br>3. Verified `CSpeedTreeForest`, `CSpeedTreeDirectX`, and `CSpeedTreeWrapper` instancing, bounds, and bounding spheres. |
| **[`PRTerrainLib`](file:///home/straut/storage/dev/mt2client_linux/src/PRTerrainLib)** | `pkg_prterrainlib` | Outdoor terrain heightmaps, quadtree patches, water textures, shadow maps, and ground splatting. | **Completed** (`PRTerrainLib_SmokeTests` + GH Actions CI) | `EterBase`, `EterPack`, `EterImageLib`, `EterLib` | 1. Refactored into Pitchfork layout with cross-platform math macros replacing x86 inline assembly (`PR_FLOAT_TO_INT`, `PR_FLOAT_TO_FIXED`, `PR_FLOAT_ADD_TO_INT`).<br>2. Tested VFS heightmap, packed attrmap, and watermap loaders with `CEterPackManager`; verified texture coordinate scaling matrices in `CTextureSet`. |

---

### Tier 3 — High-Level Scripting, Effects & Gameplay Layer (4 / 4 Completed)

Visual effects, Python 2.7 host runtime, Python UI bindings, and game world/actor mechanics.

| Package | Standalone Repo | Description | Status | Dependencies | Planned Porting Actions |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **[`EffectLib`](file:///home/straut/storage/dev/mt2client_linux/src/EffectLib)** | `pkg_effectlib` | Particle emitters, mesh effects, effect script (`.mse`) loaders, texture animations, billboarding, and trails. | ✅ Ported | Tier 1, `EterLib`, `MilesLib` | Fixed 64-bit pointer math in update decorators, added `D3DXQuaternionRotationAxis` to `DirectXMathCompat`, normalized includes, CTest smoke suite passing. |
| **[`ScriptLib`](file:///home/straut/storage/dev/mt2client_linux/src/ScriptLib)** | `pkg_scriptlib` | Embedded Python 2.7 runtime host (`CPythonLauncher`), custom `.pyc` marshal loader (`PythonMarshal`), and resource manager bridge. | ✅ Ported | Tier 1, `EterLib`, `Python 2.7` | Embedded Python 2.7 runtime host (`Py_NoSiteFlag = 1`), resolved C++17 `register` storage class issues (`-Wno-register`), fixed 64-bit `Py_ssize_t` in `PyObject_AsCharBuffer`, decoupled cyclic `GameLib` dependency via `RegisterDestroyHandler`, CTest smoke suite passing. |
| **[`EterPythonLib`](file:///home/straut/storage/dev/mt2client_linux/src/EterPythonLib)** | `pkg_eterpythonlib` | Python C extension bindings for 2D/3D rendering and UI widgets (`CWindow`, buttons, editboxes, item slots, grids, bars). | ✅ Ported | `EterLib`, `ScriptLib`, `Python 2.7` | Fixed 64-bit address truncation for window/graphic handles in Python C-API, replaced Windows Shell API screenshot pathing with POSIX standard, added DirectX compatibility stubs for gamma ramps and surface descriptions, full bi-directional Python <-> C++ test suite passing. |
| **[`GameLib`](file:///home/straut/storage/dev/mt2client_linux/src/GameLib)** | `pkg_gamelib` | Core gameplay and entity engine: actors (`ActorInstance`), items (`ItemManager`), maps (`MapOutdoor`, `MapIndoor`, `Area`), physics, and monster spawns. | ✅ Ported | Tier 1, Tier 2, `EffectLib` | Reorganized into Pitchfork layout, fixed 64-bit pointer truncation in `CMapManager::ApplyLight`, added missing DirectX math (`D3DXVec3Scale`, `D3DXVec3Add`, `D3DXVec3Subtract`, `D3DXVec3Lerp`, `D3DXVec3Transform`, `D3DXQuaternionRotationYawPitchRoll`), corrected non-const reference binding on temporary `std::for_each` return, fixed nested class qualification in `MapManager.h`, and added comprehensive smoke tests covering math, item manager, race manager, property manager, and actor instances. |

---

### Tier 4 — Top-Level Application Layer (1 / 2 Completed)

Final executables combining all engine libraries, network protocol handlers, and configuration utilities.

| Package | Standalone Repo | Description | Status | Dependencies | Planned Porting Actions |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **[`UserInterface`](file:///home/straut/storage/dev/mt2client_linux/src/UserInterface)** | [pkg_userinterface](https://github.com/open-mt2/pkg_userinterface.git) | Main game client executable (`CPythonApplication`), network packet stream (`PythonNetworkStream`), player/guild/quest modules, and master game loop. | ✅ **Ported** (Linux Static Lib & `metin2client` ELF Executable) | All Tier 1–3 libraries | 1. Implemented standard Linux `main(int argc, char** argv)`.<br>2. Pitchfork layout + standalone CMake with dual-mode library/executable.<br>3. DirectShow stubbed, 64-bit packet alignment audited, Smoke & CTest tests passing. |
| **[`Config`](file:///home/straut/storage/dev/mt2client_linux/src/Config)** | Optional / Deferred | Standalone configuration utility (`config.exe` writing `metin2.cfg` resolution, audio, and graphics settings). | Deferred | Standalone | Keep deferred during client bring-up (`metin2.cfg` is plain text); optionally replace MFC GUI with lightweight SDL2/ImGui or CLI tool later. |
