# Starling 2 Max + Parallel-Jaw Gripper — Isaac Sim Asset

**Author:** Nver | **For:** AirVLA-style pick-and-place in Isaac Sim / OmniDrones

## Contents

```
gripper.urdf          full articulated model (Starling body + gripper)
meshes/
  starling.stl        official Starling 2 Max CAD (200k faces, Y-up->Z-up fixed)
  adapter_plate.stl   95x80x3 mm mount plate (4x M3 holes at +-32,+-32)
  gripper_frame.stl   gripper body 70x50x40, servo pockets, hinge barrel
  finger_left/right.stl   sliding jaws (V-groove + pad recess, 6 deg inward tip)
  pad_left/right.stl      soft pads (2 mm)
  servo_mg90s.stl / pinion.stl / camera_block.stl   visual placeholders
  gripper_assembly.step   deployed-pose assembly reference (CAD)
  *.step              per-part CAD for editing in Fusion/FreeCAD
README.md             this file
```

## Coordinate frame & units
- Z-up, metres in URDF. Meshes are mm (URDF `scale="0.001"`).
- Drone body floor = z 0; landing-leg tips reach z = -0.0676 m.
- Mass: 0.67 kg total (0.566 drone + 0.104 gripper). Inertias are estimates — see CALCS.md.

## Joints (drivable)
| Joint | Type | Axis | Range | Meaning |
|-------|------|------|-------|---------|
| `stow` | revolute | -X | 0 .. 1.6 rad | 0 = deployed (hangs down), ~1.55 = folded under belly for landing |
| `servo_left` | prismatic | -X | 0.015 .. 0.060 m | jaw offset from centre |
| `servo_right` | prismatic | +X | 0.015 .. 0.060 m | jaw offset from centre |

Jaw gap = 2 x joint value, so 30-120 mm opening. A 40 mm object is grasped at ~0.020.

## Cameras (visual meshes — add Isaac `Camera` prims at these locations)
- Forward: `drone_body` frame, `xyz 0 0.052 0`, lens faces +Y.
- Downward: `gripper_frame` front face, `xyz 0 0.0295 -0.03`, faces +Y/down.

## Importing into Isaac Sim
1. File > Import > select `gripper.urdf`.
2. Options: `collision_from_visuals` OFF (primitive collision proxies are already defined — the real Starling STL is visual only, legs/body have box colliders). Enable *Allow Self-Collision* OFF is fine.
3. Output USD can then be referenced by OmniDrones with a `.yaml` drone config.

## Known limitations (honest)
- Starling mesh is **decimated to 200k faces** — good for sim, not a manufacturing file. Original STEP remains the CAD source.
- Mounting hole pattern (+-32,+-32) is our best reading of the STEP's clamp blocks; verify against the physical mounts before real hardware.
- Inertia tensors are box approximations. Gripper link masses are estimates (PLA + MG90S).
- Propellers/rotors are part of the static visual mesh — spin them via OmniDrones' own rotor visuals or leave static; thrust comes from the controller, not the mesh.
- Flight dynamics come from OmniDrones/PX4-sim on your side; this package is the geometry + articulation + masses.
