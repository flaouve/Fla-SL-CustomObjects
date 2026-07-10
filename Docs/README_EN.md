# SL-CustomObjects Documentation (English)

Welcome to the official developer documentation for **SL-CustomObjects**, a specialized Unity-side editor framework designed to work seamlessly with **MapEditorReborn (MER)** (LabAPI & EXILED editions) for *SCP: Secret Laboratory*.

---

## 📖 Architecture & Design Overview

SL-CustomObjects allows you to design custom rooms, structures, and interactive maps in Unity and compile them into JSON schemas (`.json` for objects, `-Rigidbodies.json` for physics, and `-Teleports.json` for teleporters) that the server-side MapEditorReborn plugin can spawn in real-time.

```
+---------------------------------------+
|              Unity Editor             |
|   (Define GameObject Hierarchies)    |
+---------------------------------------+
                    |
                    v (Compile Schematic)
+---------------------------------------+
|        JSON Serialization Files       |
|    - [Name].json (All Blocks)         |
|    - [Name]-Rigidbodies.json          |
|    - [Name]-Teleports.json            |
+---------------------------------------+
                    |
                    v (Transfer to Server)
+---------------------------------------+
|          SCP: SL Game Server          |
|       (MapEditorReborn Spawns)        |
+---------------------------------------+
```

---

## 🛠️ Codebase & File Breakdown

### 1. Core Classes

#### 🔹 [Schematic.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/Schematic.cs)
The root component of any custom map structure. It resides on the root GameObject and manages the compiling process.
* **`CompileSchematic()`**: Scans all children containing a `SchematicBlock` component, compiles them into list structures, extracts Animator/Animations into AssetBundles, handles Rigidbodies/Teleporters, serializes the data to JSON files, and optionally packages them into a `.zip` file for distribution.
* **`Update()`**: Enforces project constraints (e.g., preventing root scale changes and renaming directories to remove spaces).

#### 🔹 [SchematicBlock.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/BlockComponents/SchematicBlock.cs)
The abstract base class for all custom objects. It inherits from Unity's `MonoBehaviour`.
* **`Compile(SchematicBlockData block)`**: Virtual method that extracts basic GameObject parameters (Name, instance ID, parent instance ID, Local Position, Local Rotation, Local Scale, and whether it's marked as `Static`).
* **`Decompile(...)`**: Virtual method allowing the system to reconstruct the Unity hierarchy from a serialized JSON file.

---

### 2. Block Components (`Assets/DONT TOUCH/Scripts/BlockComponents/`)

Each custom object in your Unity hierarchy is attached to one of these components to determine how it translates into the game:

* **[PrimitiveComponent.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/BlockComponents/PrimitiveComponent.cs)**: Represents standard Unity shapes (Cubes, Spheres, Cylinders, Planes, etc.) converted into server-side game primitives. Handles collision properties, material colors, and physics flags.
* **[LightComponent.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/BlockComponents/LightComponent.cs)**: Emits points of light (Directional, Point, Spot, Area). Controls intensity, color, shadows, and range.
* **[TeleportComponent.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/BlockComponents/TeleportComponent.cs)**: Defines teleporter logic, allowing players or objects to teleport to target coordinates or IDs.
* **[TextComponent.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/BlockComponents/TextComponent.cs)**: Spawns floating 3D text (text toys) in-game with custom font size, content, and color.
* **[InteractableComponent.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/BlockComponents/InteractableComponent.cs)**: Adds custom player interactions (buttons/sensors) with triggers, timers, and game events. Supports interactive animation triggering by referencing a target `Animator` component and specifying state names (including a second state for toggling between states on subsequent interactions).
* **[WaypointComponent.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/BlockComponents/WaypointComponent.cs)**: Positions path nodes or navigation markers for custom scripts.
* **[WorkstationComponent.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/BlockComponents/WorkstationComponent.cs)**: Spawns in-game weapon modification workstations.
* **[PickupComponent.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/BlockComponents/PickupComponent.cs)**: Defines spawnpoints for weapons, keycards, or items with custom properties (ammo attachments, spawn chances, infinite pickups, etc.).
* **[LockerComponent.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/BlockComponents/Locker/LockerComponent.cs)**: Manages in-game lockers, chests, and their loot pools (using `LockerChamber.cs` and `LockerItem.cs`).
* **[CameraComponent.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/Scripts/CameraComponent.cs)**: Places functional SCP-079 cameras.
* **[EmptyComponent.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/BlockComponents/EmptyComponent.cs)**: A dummy/empty node used to group child objects.

---

### 3. Editor Extensions (`Assets/DONT TOUCH/Scripts/Editors/`)

* **[RightClickMenuExtended.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/Editors/RightClickMenuExtended.cs)**: Creates the **"🛠️ MER Blocks"** menu item in the Unity right-click/GameObject menu. This is the recommended way to spawn primitives, lights, and custom interactive objects under your active selection.
* **[SchematicEditor.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/Editors/SchematicEditor.cs)**: Implements custom Inspector windows for the `Schematic` component, adding the "Compile Schematic" button.

---

## ⚙️ How to Compile a Schematic

1. Open your Unity Project.
2. Build your schematic structure by right-clicking in the **Hierarchy** -> **🛠️ MER Blocks** and adding components.
3. Keep all components nested under a single parent object containing the `Schematic` component.
4. Select the root object, navigate to its Inspector window, and click **Compile Schematic**.
5. Find the compiled JSON/AssetBundle output files inside your configured export path (or Desktop by default).
6. Copy these files to your SCP: SL server directory under:
   `LabAPI/configs/ProjectMER/Schematics/[YourSchematicName]/`
