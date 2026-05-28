# Isaac Sim 6.0 Compatibility Branches

## Branches

| Repo | Branch | Base | Patches |
|------|--------|------|---------|
| robocasa | `isaac-sim-6.0` | `origin/main` (be22d65) | 5 commits |
| robosuite | `isaac-sim-6.0` | v1.5.2 (PyPI wheel) | 5 commits |

## RoboCasa Patches (5 files changed)

### 1. `robocasa/__init__.py` - Remove version assertions
Removed hard assertions for `mujoco==3.3.1` and `numpy==2.2.5`.
Isaac Sim 6.0 ships MuJoCo 3.5 and numpy 2.2.x (backward compatible).

### 2. `robocasa/environments/kitchen/kitchen.py` - MuJoCo 3.5 compat
Removed `enable_multiccd=True` and `enable_sleeping_islands=False` from
world creation. These options were removed in MuJoCo 3.5.

### 3. `robocasa/environments/kitchen/atomic/kitchen_doors.py` - Default fixture_id
Added `fixture_id=FixtureType.CABINET_WITH_DOOR` as default for
`ManipulateDoor` and `ManipulateLowerDoor` constructors.

### 4. `robocasa/utils/env_utils.py` - Filter kwargs for non-kitchen envs
`create_env()` now only passes RoboCasa-specific kwargs (camera names,
layout_ids, etc.) to Kitchen subclasses. Base robosuite environments
(Door, Lift, etc.) get default camera names.

### 5. `setup.py` - Relax pinned dependencies
Changed exact version pins to minimum version constraints for numpy,
numba, scipy, mujoco, tianshou, and lerobot.

## Robosuite Patches (6 files changed, +14 -2 lines)

### 1. `robosuite/__init__.py` - load_controller_config shim
RoboCasa references `load_controller_config` which exists in robosuite
master but was renamed in v1.5.2. Added compatibility shim.

### 2. `robosuite/controllers/parts/controller_factory.py` - JOINT_VELOCITY_LEGACY
Accept both `JOINT_VELOCITY` and `JOINT_VELOCITY_LEGACY` in
`mobile_base_controller_factory` for PandaOmron config compatibility.

### 3. `robosuite/environments/base.py` - getattr guard
Changed `if self.sim is not None` to `if getattr(self, 'sim', None) is not None`
in `_destroy_sim()` to handle pre-init cleanup calls.

### 4. `robosuite/environments/manipulation/manipulation_env.py` - **kwargs
Added `load_model_on_init=True` parameter and `**kwargs` to constructor
to accept RoboCasa's extra environment arguments.

### 5. `robosuite/environments/manipulation/door.py` + `lift.py` - **kwargs
Added `**kwargs` to Door and Lift constructors to handle RoboCasa's
`create_env()` passing extra arguments.

## Installation

```bash
# In Isaac Sim 6.0 venv
cd /path/to/robosuite_isaac6 && pip install --no-deps -e .
cd /path/to/robocasa_isaac6 && pip install --no-deps -e .
```

## Test Results
- 391/396 environments: create + reset + step + close -- PASS
- 5 expected failures (not regressions):
  - PickPlace: abstract base class (NotImplementedError in _get_obj_cfgs)
  - TwoArmHandover/Lift/PegInHole/Transport: require bimanual robot, PandaOmron is single-arm
- Visual verification: 8 environments with 50-step rollouts confirmed
- Robot end-effector displacement: 0.2-0.5m (realistic for random actions)
- Assets: 6/6 packages downloaded (~10GB)
- Test duration: 52 minutes for full 396-env suite
