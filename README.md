# KaleidoVR Asset Organizer

<p align="center">
  <img src="Unity Organizer Tool/Editor/Icons/Kali_Logo.png" alt="KaleidoVR" width="300">
</p>

A Unity editor tool that sorts the assets of a VRChat avatar into a consistent folder structure, remaps their references, and writes a scene plus a packed prefab that point at those organized copies.

Drop an avatar FBX or prefab into the window, press **Organize Assets**, and its meshes, materials, textures, animations, controllers, menus, and parameters are collected and filed into one output folder.
<p align="center">
<img width="500" height="1101" alt="image" src="https://github.com/user-attachments/assets/2f1ed18f-d4e8-4115-9088-612ef5ede97e" />
</p>

- Unity **2022.3.22f1** or newer, including Unity 6 (6000.x)
- VRChat SDK3 Avatars is optional

## Install

1. Download the latest `.unitypackage` from [Releases](https://github.com/KaleidoVR/Asset-Organizer/releases).
2. In Unity, choose **Assets > Import Package > Custom Package...** and select the file.
3. Import everything. Files land in `Assets/KaleidoVR/Editor/`.
4. Open the tool from the menu bar: **KaleidoVR > Asset Organizer**.

To install from source instead, copy the `Editor` folder (including its `.meta` files) into `Assets/KaleidoVR/` in your project.

## Requirements

Unity 2022.3.22f1 or newer. The tool only uses editor APIs that exist in the 2022.3 LTS line, so it also compiles and runs on Unity 6.

The VRChat SDK3 Avatars package is only needed for the **Auto-Link FX & Menu** option. Everything else — organizing, copying, moving, and prefab creation — works in a plain Unity project. If the SDK is absent, the descriptor step is skipped with a warning instead of failing.

## Usage

1. Set **Output Directory** with **Select Folder**. It has to be inside `Assets`.
2. Drop your avatar into **Objects to Organize**. Project FBX/prefab assets work, and so do scene instances — those resolve back to their source asset.
3. You can set **Scene Name** and **Prefab Name**, and drop anything you want left out of the new prefab and scene into the **Ignore List**. I leave files that belong only to those objects, including unique materials and textures, out of the output.
4. Press **Organize Assets**.

**Scene Name** and **Prefab Name** fill from the first object you drop in. I always write a scene named after **Scene Name** and a packed prefab named after **Prefab Name** into `<output>/Prefabs/`. Nested hair, clothes, and the original avatar prefab unpack into that one prefab. I do not copy them as extra `.prefab` files.

Use the **Settings (Beta)** tab if you want to rename those output folders or send a type to a different folder. Copy, Move, and Ignore stay on Organize.

### Export List Options

Each asset type can be set to one of three actions:

- **Copy** — duplicate the asset into the output folder, leaving the original in place. This is the default.
- **Move** — relocate the original into the output folder. Use only when you intend to move your source files.
- **Ignore** — skip the type entirely.

Scripts, DLLs, shaders, and anything under `Packages/`, `Assets/Editor`, `Assets/KaleidoVR/Editor`, or `Assets/KaleidoVR/Generated` are always skipped, so the tool will not relocate Poiyomi, the VRChat SDK, Kaleido editor scripts, or Kaleido generated cache.

### Organize settings

- I always write the packed prefab into `<output>/Prefabs/` and place that prefab into the scene.
- **Rename Old / New Objects** — off by default. When on, leftover originals in the source scene get an Old suffix.
- I sort textures into subfolders: suffix maps (normal, emission, metallic, roughness, AO), VRChat menu icons (`Icons`), LilToon and Poiyomi Mask slots (`Masks`), and cubemaps (`CubeMaps`). Menu icons win over Masks. Masks win over CubeMaps. Those win over the suffix folders.
- I lock every new Poiyomi material after Organize. I unlock locked copies first so their textures can follow the new files. You need Poiyomi/Thry in the project for that. LilToon materials stay as they are.
- **Auto-Link FX & Menu** always runs on a single organized object: add or reuse a `VRCAvatarDescriptor` and assign the FX layer, expressions menu, and expression parameters.

Organize settings persist between sessions via `EditorPrefs`.

## Settings (Beta)

**Settings (Beta)** is a second tab for output folder names and where each asset type lands. It does not change Copy, Move, or Ignore — those stay on Organize.

### Folder layout

Parents are the folders under **Output Directory**. Child rows show the full path they land in, for example `Textures/Normals` or `3.0/Animations`.

Default names match the tree in **Output structure** below: Models (`FBX`), Materials, Textures (Normals, Emissions, Metallic, Roughness, AO, Icons, Masks, CubeMaps), Audio, Prefabs, Other, and the VRChat root (`3.0`) with Animations, Blend Trees, Avatar Masks, Controllers, Menus, and Parameters.

### Where each type goes

Each export type has a folder dropdown. That only picks the destination folder. Copy, Move, and Ignore are still set per type on Organize.

Shader, MonoScript, and DefaultAsset show an orange **Warning (Special use case)** because those types are special-use and default to Ignore on Organize.

The packed result always lands in the Prefabs folder. Texture suffix subfolders still apply when a texture name matches normal, emission, metallic, roughness, or AO. Menu icons, Mask-slot textures, and cubemaps take their own folders first. You can rename Icons, Masks, and CubeMaps here.

### These settings apply to

- **All organizes** — the folder layout is used for every output folder.
- **This output folder only** — the layout is stored for the current Output Directory and does not change other folders.

**Reset to defaults** restores the stock folder names and type destinations.

Settings (Beta) also persists via `EditorPrefs`.

## Output structure

```
<output folder>/
├── <Scene Name>.unity
├── FBX/
├── Materials/
├── Textures/
│   ├── Normals/
│   ├── Emissions/
│   ├── Metallic/
│   ├── Roughness/
│   ├── AO/
│   ├── Icons/
│   ├── Masks/
│   └── CubeMaps/
├── Audio/
├── Prefabs/
├── Other/
└── 3.0/
    ├── Animations/
    ├── BlendTrees/
    ├── Avatar Masks/
    ├── Controllers/
    ├── Menus/
    └── VRCExpressionParameters/
```

That tree is the default. **Settings (Beta)** can rename any of those folders or send a type somewhere else. Empty texture children are not kept if that organize did not write files into them.

## How references are kept intact

Copies are made through the Unity asset database rather than by copying files and their `.meta` on disk. Each copy therefore gets its own GUID, and the tool then rewrites the serialized references of the copied assets and the generated prefab to point at the new files.

This matters because duplicating a `.meta` file duplicates its GUID, which leaves two assets claiming the same identity and causes materials, controllers, and prefabs to resolve to the wrong file.

Every run writes a log of what was copied, moved, and ignored to `Logs/KaleidoVR/Organizer/`.

## Known limitations

Prefab generation expects the avatar root as a single object. Dropping several root objects nests them all under one new parent named after **Prefab Name**.

## Credits

Created and maintained by **KaleidoVR**.

- [kalivr.com](https://kalivr.com)
- [Discord](https://discord.com/invite/cRsufJssTA)

## License

Copyright (c) 2026 KaleidoVR. All rights reserved.

Free to use in your own projects, personal or commercial, including commission work. Please don't modify it, sell it, or bundle it into products or avatar downloads — link people to the repo instead so they get the current version.
