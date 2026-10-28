# Starling 2 Max gripper - design calculations

## 1. Platform (ModalAI datasheet, docs.modalai.com/starling-2-max-datasheet)
| | |
|---|---|
| Take-off weight (no payload) | 566 g |
| Payload capacity | 500 g (all-up ~1066 g; flight time drops) |
| Motors / props | 4x 2203.5 1500 kV, 180 mm tri-blade, 4S (16.8 V) |
| Motor-to-motor | 260 x 190 mm |
| Landing-leg height (from STEP) | 67.6 mm |
| Belly mount (from STEP) | 4 arm-clamp blocks at (+-32, +-32), bottoms at -16 mm |

Max thrust estimate: ~650 g per motor -> ~2.6 kgf total (ESTIMATE from motor class, not datasheet).
Thrust-to-weight at full payload: 2.6 / 1.07 = 2.4 (ok; >2 is the usual minimum).

## 2. Payload budget
    payload capacity        500 g
  - gripper (target)       -120 g   frame ~60 g + 2x MG90S 27 g + SG90 stow 9 g + camera 10 g + fasteners
  = max object             ~380 g

Every gram saved on the gripper is a gram more object. AirVLA's gripper is heavier (est. 150-200 g)
-> lighter gripper with equal grip strength = higher usable payload than the reference design.

## 3. Grip force (friction grasp, two parallel jaws)
    F_jaw >= m * g * (1 + a/g) / (2 * mu)
    m = 0.38 kg, a = 3 m/s^2 (controller accel limit), mu = 0.6 (rubber pad on steel/wood)
    F_jaw >= 0.38 * 9.81 * 1.31 / 1.2 = 4.1 N      -> with safety factor 2.5: ~10 N per jaw

Servo -> jaw force via pinion radius r:  F = tau / r
    MG90S: tau = 0.22 N*m, r = 6 mm -> 36 N   (3.6x margin)
    SG90:  tau = 0.18 N*m, r = 6 mm -> 30 N   (3.0x margin)
=> small 9-14 g servos are sufficient; heavy servos only cost payload.

## 4. Grasp geometry (verified in MuJoCo, drone/sim/)
- Hinged V-paddles: squeezing an object on a table wedges the drone UP ~45 mm; object ends up at the
  paddle tips and slips. Rejected.
- Parallel jaws (UMI style): object held mid-paddle, zero creep with proper contact solver. Chosen.
- Jaw opening 0-70 mm: covers cubes to 60 mm and hammer handles (25-35 mm). Grasp a hammer by the HANDLE.
- V-groove + 2 mm rubber in the paddle face: self-centres cylinders, raises mu to ~0.9.
- Stow hinge (90 deg fold under belly): paddles (90 mm) are longer than the legs (67.6 mm),
  so the gripper must fold for take-off/landing. Same solution as the AirVLA video.

## 5. Flight-controller note
A pure PD altitude loop cannot lift an unknown payload (sim: 350 g object -> drone stuck on the table).
Adding an altitude INTEGRAL term (as PX4 does) recovers it; the "payload sag" seen in the sim
(1.00 -> 0.97 m at 350 g, -> 0.84 m at 450 g) is exactly the effect AirVLA's Payload-Aware Guidance
compensates for.

## 6. Sim results (fly_pick_place.py --dryrun, OBJ_MASS env var)
| object | lifted | min height during lift | in bin |
|---|---|---|---|
| 50 g  | yes | 1.05 m (overshoot) | yes |
| 350 g | yes | 0.97 m | yes |
| 450 g | yes (marginal) | 0.84 m | yes |
