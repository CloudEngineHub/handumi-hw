# HandUMI - Hardware

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-Apache_2.0-blue.svg" alt="License: Apache 2.0"></a>
  <a href="https://github.com/murobotics-ai/handumi-sw"><img src="https://img.shields.io/badge/Software-handumi--sw-4c8bf5.svg" alt="HandUMI software"></a>
  <a href="https://murobotics-ai.github.io/handumi-sw/"><img src="https://img.shields.io/badge/Docs-GitHub_Pages-AF0000.svg" alt="HandUMI documentation on GitHub Pages"></a>
  <a href="https://murobotics.ai/research/handumi"><img src="https://img.shields.io/badge/Project-Website-222222.svg" alt="HandUMI project website"></a>
  <a href="https://huggingface.co/NONHUMAN-RESEARCH"><img src="https://img.shields.io/badge/%F0%9F%A4%97-Data_%26_Checkpoints-FFD21E.svg" alt="HandUMI data and checkpoints on Hugging Face"></a>
</p>

<p align="center">
  <img src="media/paper-overview.png" alt="HandUMI overview: operation setup, labelled hand-worn device (fisheye camera, VR controller, servo encoder, swappable fingertips), PICO 4 Ultra or Meta Quest tracking, and recorded trajectories retargeted to seven bimanual arms" width="800">
</p>

A hand-worn, open-source variant of the Universal Manipulation Interface (UMI)
for collecting bimanual manipulation data without a robot in the loop, designed
for robot arms with parallel grippers. HandUMI mounts on the operator's thumb
and index/middle fingers, opens and closes with a natural pinch, and uses
interchangeable gripper tips to target different parallel-jaw robot grippers.
One unit costs roughly $104 in parts; a bimanual pair costs $207.11 in parts,
or about $557 with a shared Meta Quest 3S (PICO 4 Ultra and Meta Quest 3 are
also supported).

HandUMI demonstrations are robot-agnostic: collect the data once with the
wearable hardware, then use the [HandUMI software](https://github.com/murobotics-ai/handumi-sw)
to retarget it across different bimanual arms with parallel grippers. The
robot keeps its installed gripper; only the printed tip on HandUMI changes to
match it.

## Paper

**HandUMI: A Modular Hand-Worn Parallel-Jaw Interface for Robot-Free Bimanual
Data Capture**  
Brik Meza, Leonardo Perez, Alvaro Mendoza, Aniela Ubillus, Mitshell Ramos,
Raul Bastidas, Mauricio Olivera, Bryan Bastidas  
Mu Robotics · McGill University · PUCP · NONHUMAN · TUM · UPC

[Project website](https://murobotics.ai/research/handumi) ·
[Data and checkpoints](https://huggingface.co/NONHUMAN-RESEARCH) ·
[Software](https://github.com/murobotics-ai/handumi-sw) ·
[Quest app](https://github.com/murobotics-ai/handumi-quest-app)

On bimanual towel folding, 562 robot-free HandUMI demonstrations (111.9 min,
no robot-collected data) were retargeted to AgileX PiPER arms and used to train
policies that fold the towel on the real robot:

| Policy | Training data | Real-robot successes |
|---|---|---:|
| ACT | First 101 HandUMI episodes | 0/20 |
| ACT | 562 HandUMI episodes | 12/20 |
| π0.5 (base, no fine-tuning) | — | 0/20 |
| π0.5 (fine-tuned) | 562 HandUMI episodes | 7/20 |

<p align="center">
  <img src="media/paper-towel-folding.png" alt="Towel-folding task: left and right fisheye wrist views, ZED Mini top view, and a PiPER rollout from initial state to grasping edges to folding the towel" width="520">
  <img src="media/paper-policy-results.png" alt="Real-robot success rates on towel folding: ACT 101 episodes 0/20, ACT 562 episodes 12/20, pi0.5 base 0/20, pi0.5 562 episodes 7/20" width="300">
</p>

The same recordings retarget in simulation to seven bimanual arms with
sub-millimeter median IK position error; the list of supported arms lives in
the [software README](https://github.com/murobotics-ai/handumi-sw#supported-bimanual-embodiments).
See the paper for the capture-alignment, calibration and retargeting results.

## Why Teleoperation Is Not Scalable?

Traditional leader-follower teleoperation is expensive and lab-constrained.

It is expensive because collecting data for bimanual arms requires at least
four robotic arms: two follower arms plus two leader arms, or a VR
headset acting as the leader for both. Even low-cost leader arms like
[GELLO](https://wuphilipp.github.io/gello_site/), which replicate the
follower's kinematics in 3D-printed hardware to cut the leader cost to a few
hundred dollars each, still leave you duplicating both follower arms for
every operator who wants to collect data.

It is lab-constrained because the follower arm has to physically move to
wherever the data needs to be collected. Bolting an arm to a cart and hauling
it between rooms, buildings, or the environments you actually want to collect
data in is slow, heavy, and hard to scale beyond a single fixed setup.

I felt this pain firsthand: the NONHUMAN team and I collected over 2,000
episodes using this method
([link](https://huggingface.co/spaces/lerobot/visualize_dataset?path=%2FAutobrik%2Fbi_piper_pick_and_place_fruits_mantra%2Fepisode_0)).

<p align="center">
  <img src="media/Follower_Leader_setup.png" alt="Classical bimanual leader-follower teleoperation setup" width="720">
</p>

## Why HandUMI

HandUMI removes both the cost and the lab constraint by moving the collection
interface onto the operator's hand instead of onto a robot, the same way the
original UMI removed the lab constraint for its own gripper. This time, that
wearable concept is adapted from [Generalist](https://generalistai.com/)'s
approach and re-targeted at robot arms with parallel-jaw grippers, and it is
open-source and modular: the body, camera mount, servo, and tracker mounting
stay the same across robots, and only the detachable gripper tip changes to
target a new one. Demonstrations can then be captured directly from human
motion, anywhere, without a robot arm at all. Current target tips are AgileX
Piper, ARX X5 2023, Dream Gripper (TRLC), Trossen WidowX AI, and the original
UMI gripper.

Any robot with a comparable parallel-jaw gripper can be supported by designing
and printing a matching tip.

<p align="center">
  <img src="media/modular-handumi.png" alt="With HandUMI, you don't need robotic arms to collect data. Instead, use them for deployment! Data collection tips vs. deployment arms for AgileX Piper, ARX X5, UR/Dream Gripper, and ARX" width="800">
</p>

## What It Records

Each demonstration records the core signals needed for later deployment:

- SE(3) wrist pose from a VR headset (PICO 4 Ultra or Meta Quest 3/3S) and
  its controllers.
- Gripper width from a Feetech servo encoder.
- Wrist-view video from a small camera mounted on the device.
- Optionally, a workspace view from a ZED Mini.

## Direct Gripper-Width Sensing

Most UMI-style rigs estimate gripper aperture indirectly from fiducials or image
segmentation. HandUMI measures aperture directly with a Feetech servo encoder,
so the recorded width follows the mechanical opening frame by frame.

## Motion Tracking

Pose comes from a VR headset and its two controllers. Depending on the user's
preference, the headset can be a PICO 4 Ultra or a Meta Quest 3/3S. The headset
provides the world frame, while the controllers provide each hand trajectory.
Each HandUMI includes a printed controller support (the
`controller_support` parts in `hardware/STL/left_handumi/` and
`hardware/STL/right_handumi/`) that mounts the controller on the wrist.
This avoids an offline camera SLAM step and keeps the wrist camera focused on
visual observation.

## Wrist-View Camera

The wrist camera provides the observation used during training and deployment.
HandUMI uses the IMX335 USB camera (SVPRO, 5 MP) listed in the
[Bill of Materials](bom/README.md), a compact UVC module with a wide field of
view.

## Demo

<p align="center">
  <img src="media/HandUMIDemo.gif" alt="HandUMI data collection demo, showing wrist cameras, gripper-width tracking, and the recorded episode in simulation" width="800">
</p>

### Additional Example

Natural pinch control makes fine-motor tasks tractable that are notoriously
hard to teleoperate with leader-follower puppeteering rigs. Below, an operator
plugs a USB-C cable into a keyboard, a precision insertion task that benefits
directly from direct human dexterity rather than an intermediary robot arm.

<p align="center">
  <img src="media/hand_umi_real_demo.gif" alt="HandUMI demo: plugging a USB-C cable into a keyboard, a precision task that is hard to teleoperate with leader-follower puppeteering" width="300">
</p>

## Bill of Materials

The full bill of materials is available in [`bom/README.md`](bom/README.md).
It lists every part needed to build one HandUMI unit — mechanical, structural,
and electronic — with purchase links (Amazon and Alibaba) and per-unit prices.
One unit comes to roughly $104 in parts; a bimanual pair to $207.11. The
VR headset used for the shared tracking layer is a separate one-time purchase;
with a Meta Quest 3S the pair comes to about $557 (excluding the host computer,
tax and shipping).

## Quick Links

- [Hardware](hardware/README.md)
- [STL files](hardware/STL/)
- [Bill of Materials](bom/README.md)
- [Assembly Guide](hardware/HandUMI_Assembly_Guide.pdf)
- [Software](https://github.com/murobotics-ai/handumi-sw)
- [Documentation](https://murobotics-ai.github.io/handumi-sw/)
- [Quest app](https://github.com/murobotics-ai/handumi-quest-app)
- [Project website](https://murobotics.ai/research/handumi)
- [Data and checkpoints](https://huggingface.co/NONHUMAN-RESEARCH)

## Hand-Fit Design

The finger cradle geometry was designed from a 3D scan of the operator's hand.
That scan is used as a CAD reference surface for the thumb and index/middle
finger rings, and
the same workflow can be repeated to fit another operator.

<p align="center">
  <img src="media/3d_hunyuang_hand.png" alt="3D hand scan used as CAD reference for HandUMI" width="540">
</p>

## References

UMI pioneered in-the-wild data collection without a robot in the loop, and
YUBI brought that idea to a finger-driven V-shaped gripper.
[Generalist](https://generalistai.com/) built a proprietary hand-worn device
for a V-shaped gripper too. HandUMI is the open-source counterpart for robot
arms with parallel-jaw grippers.

- Cheng Chi, Zhenjia Xu, Chuer Pan, Eric Cousineau, Benjamin Burchfiel, Siyuan
  Feng, Russ Tedrake, and Shuran Song. "Universal Manipulation Interface:
  In-The-Wild Robot Teaching Without In-The-Wild Robots." *Robotics: Science
  and Systems (RSS)*, 2024. [https://umi-gripper.github.io/](https://umi-gripper.github.io/)
- Takehiko Ohkawa, Jumpei Arima, Yuki Noguchi, et al. "YUBI: Yielding
  Universal Bidigital Interface for Bimanual Dexterous Manipulation at Scale."
  *arXiv:2606.10244*, 2026. [https://yubi.airoa.io/](https://yubi.airoa.io/)

## Citation

If you use HandUMI, please cite:

```bibtex
@misc{meza2026handumi,
  title  = {{HandUMI}: A Modular Hand-Worn Parallel-Jaw Interface for
            Robot-Free Bimanual Data Capture},
  author = {Meza, Brik and Perez, Leonardo and Mendoza, Alvaro and
            Ubillus, Aniela and Ramos, Mitshell and Bastidas, Raul and
            Olivera, Mauricio and Bastidas, Bryan},
  year   = {2026},
  url    = {https://murobotics.ai/research/handumi}
}
```

## Acknowledgments

We thank the [NONHUMAN](https://huggingface.co/NONHUMAN-RESEARCH) community
for their collaboration on data collection and device testing.

## Contributing

This project is under active development, and contributions are welcome. Please
read [`CONTRIBUTING.md`](CONTRIBUTING.md) before opening a pull request for
bugs, improvements, documentation, BOM updates, or new gripper tips.

This field still has plenty of unsolved problems, and I'm confident the
open-source community can contribute a great deal toward getting robots to do
tasks we currently think are impossible for them.
