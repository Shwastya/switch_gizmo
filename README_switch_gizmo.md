# switch_gizmo

A small, editor-focused package that provides **two ImGuizmo variants** side-by-side:

- **modern/**: intended for newer ImGui docking branches (e.g., ~1.9x era)
- **legacy/**: a classic ImGuizmo version closer to older behavior (e.g., ~1.6x era)

This repository exists to let an engine/editor **toggle ImGuizmo implementation at compile time** to compare behavior, verify regressions, and keep a stable fallback while integrating newer releases.

---

## Why this repo exists

In some setups, the **modern ImGuizmo line** may behave differently while dragging/rotating/scaling compared to older builds (for example, axes visibility or interaction patterns under certain UI layouts). Keeping a legacy variant available makes it easy to:

- isolate whether an issue is a regression vs integration-related
- validate changes quickly
- keep production workflows stable

---

## Contents

This repository intentionally keeps **only the minimal files** required for each variant:

```
modern/
  ImGuizmo.h
  ImGuizmo.cpp
  LICENSE
  README.md

legacy/
  ImGuizmo.h
  ImGuizmo.cpp
  LICENSE
  README.md
```

Each subfolder preserves its original license and notes.

---

## ImGui compatibility note

The legacy variant may require small compatibility shims if you are compiling against newer ImGui versions.

A common fix pattern is:

```cpp
#if defined(IMGUI_VERSION_NUM) && IMGUI_VERSION_NUM >= 18900
    // Modern ImGui
    ImGui::SetNextFrameWantCaptureMouse(true);
#else
    // Older ImGui
    ImGui::CaptureMouseFromApp(true);
#endif
```

Your project can carry this patch inside `legacy/ImGuizmo.cpp` as needed.

---

## Integration (CMake example)

This repo is designed to be consumed as source.

Assuming you also have ImGui available as a target `imgui::imgui`:

```cmake
option(SCF_USE_IMGUIZMO_LEGACY "Use legacy ImGuizmo" OFF)

set(SWITCH_GIZMO_DIR "${CMAKE_SOURCE_DIR}/third_party/switch_gizmo")

add_library(imguizmo_modern STATIC
  ${SWITCH_GIZMO_DIR}/modern/ImGuizmo.cpp
)
target_include_directories(imguizmo_modern PUBLIC
  ${SWITCH_GIZMO_DIR}/modern
)
target_link_libraries(imguizmo_modern PUBLIC imgui::imgui)

add_library(imguizmo_legacy STATIC
  ${SWITCH_GIZMO_DIR}/legacy/ImGuizmo.cpp
)
target_include_directories(imguizmo_legacy PUBLIC
  ${SWITCH_GIZMO_DIR}/legacy
)
target_link_libraries(imguizmo_legacy PUBLIC imgui::imgui)

set(_IMGUIZMO_TARGET imguizmo_modern)
if (SCF_USE_IMGUIZMO_LEGACY)
  set(_IMGUIZMO_TARGET imguizmo_legacy)
endif()

# Example consumer
target_link_libraries(my_editor PRIVATE ${_IMGUIZMO_TARGET})
target_compile_definitions(my_editor PRIVATE
  SCF_USE_IMGUIZMO_LEGACY=$<BOOL:${SCF_USE_IMGUIZMO_LEGACY}>
)
```

---

## Usage expectations

Recommended per-frame usage (typical pattern):

```cpp
ImGuizmo::BeginFrame();
ImGuizmo::SetRect(x, y, w, h);
ImGuizmo::Manipulate(view, proj, operation, mode, matrix);
```

(Exact drawlist selection and viewport overlay strategy is left to the host editor.)

---

## Origins & attribution

- **modern** variant is based on ImGuizmo by **Cedric Guillemet**.
- **legacy** variant is based on a historical ImGuizmo line (commonly used in older editor setups).
- Both are distributed under the MIT license, preserved in each subfolder.

This repository does not claim authorship of ImGuizmo itself; it only provides a structured, switchable packaging for editor workflows.

---

## Scope

- This package is intended for **editor tooling**.
- It is not meant to be pulled into engine/core libraries that aim to stay UI-agnostic.
- Prefer including and linking it only from client/editor targets.

---

## License

See:
- `modern/LICENSE`
- `legacy/LICENSE`
