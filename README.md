# Starling 2 Max + Parallel-Jaw Gripper — Isaac Sim Asset
 
Articulated URDF model of the **ModalAI Starling 2 Max** carrying an underbelly
**parallel-jaw gripper** with a fold-up (stow) joint — built for AirVLA-style
aerial pick-and-place in **Isaac Sim / OmniDrones**.
 
## What's inside
 
```
gripper.urdf          articulated model: drone body + plate + hinged frame + 2 sliding jaws
meshes/
  starling.stl            official Starling 2 Max CAD, decimated to 200k faces (Y-up→Z-up)
  adapter_plate.stl/step  95×80×3 mm mount plate, 4× M3 holes, centre window, hinge ears
  gripper_frame.stl/step  70×50×40 mm body, MG90S servo pockets, hinge barrel, rail slot
  finger_*/pad_*.stl/step sliding jaws — 2 mm pad recess, V-groove, 6° inward tips
  servo_mg90s / pinion / camera_block  visual placeholders
  gripper_assembly.step   core assembly (deployed pose)
  gripper_full.step       all 10 parts, separate bodies (open in Fusion/SolidWorks)
CALCS.md              mass budget, grip-force and servo sizing, controller notes
```
 
## Coordinates & units
 
- **Z-up, SI units** in the URDF; meshes are authored in mm (`scale="0.001"`).
- Drone body floor at `z=0`; landing-leg tips at `z=-0.0676` m.
- Total mass **0.67 kg** (0.566 airframe + 0.104 gripper). Inertias are box
  approximations — refine before precise controller work.
 
## Joints (the three drives Grik's controller commands)
 
| Joint | Type | Axis | Range | Meaning |
|-------|------|------|-------|---------|
| `stow` | revolute | −X | 0 → 1.6 rad | 0 = deployed below belly · ~1.55 = folded for landing |
| `servo_left` | prismatic | −X | 0.015 → 0.060 m | jaw offset from centre |
| `servo_right` | prismatic | +X | 0.015 → 0.060 m | jaw offset from centre |
 
Jaw gap = 2 × joint value → 30–120 mm opening; a 40 mm object is held at ≈0.020.
 
## Cameras
 
Place Isaac `Camera` prims at the green placeholder meshes:
 
- **Forward:** `drone_body` frame, `xyz 0 0.052 0`, looks +Y
- **Downward:** `gripper_frame`, `xyz 0 0.0295 -0.03`, looks +Y/down (grasp view)
 
## Import into Isaac Sim
 
1. **File → Import →** select `gripper.urdf`.
2. Leave *collision from visuals* **off** — primitive colliders are included
   (body box + 4 legs); the dense Starling mesh is visual-only.
3. Save as USD and reference from an OmniDrones `.yaml` drone config.
 
## Validated
 
- Loads clean in **MuJoCo 3.17** — 6 links, all 3 joints within limits.
- Full mission script (deploy → grasp → carry → release → stow → land) ran
  end-to-end on this model before mesh swap; see `CALCS.md`.
 
## Known limitations (honest)
 
- Starling mesh is a **sim-visual decimation**, not a manufacturing file.
- Mount-hole pattern (±32 mm) is best-effort from the official STEP — verify
  against hardware before printing a real mount.
- Inertia tensors and link masses are estimates.
- Rotors are static visuals; propulsion is modelled by the controller, not the mesh.
 
## License
 
MIT — use it, extend it, keep the credit line.
 
---
 
*Designed by Nver — first drone payload, built with AI tooling, tested in MuJoCo
before handing to Isaac Sim.*
