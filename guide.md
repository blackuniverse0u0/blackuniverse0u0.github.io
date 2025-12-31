# Planning & Control


- Perception & segformer train(DDP)
```
@projects/perception_planning_control/SideWalker/jplanner/zzz_semantic.py

@projects/perception_planning_control/SideWalker/segformer
```

- Planner & controller
```
@projects/perception_planning_control/SideWalker/jplanner/bev_mppi/collision_detector_runner.py
```
- MPPI test 
```
@projects/perception_planning_control/SideWalker/mppi
```


- Iplanner test(ETH,Zurich paper) & Visual transformer navigation(UC bercly paper)
```
@projects/perception_planning_control/SideWalker/iplanner_v60_deploy
```

- myself custom End-to-End(Behavior Cloning)
```
@projects/perception_planning_control/AutoNavigation/scripts/train2.py
```

# Locomotion
아래 경로에서 학습했어.

Rigid body motion 
- so(3),se(3),PoE,screw theory,jacobian ...etc

Forwrad kinematics 
```
@projects/locomotion_RL_dynamics/physics_to_robot/mujoco_physics/froward_kinematics

@projects/locomotion_RL_dynamics/physics_to_robot/mujoco_physics/robot/fk.py
```

Inverse Kinematics 
```
@projects/locomotion_RL_dynamics/physics_to_robot/mujoco_physics/robot/ik.py
```

quadruped trot
```
@projects/locomotion_RL_dynamics/physics_to_robot/mujoco_physics/trot/a1_trot/mj_a1_trot.py
```


Reinforcement Learning
```
@projects/locomotion_RL_dynamics/go1_joystick/baseline

@projects/locomotion_RL_dynamics/krm_loco/mujoco_playground/learning/train_rsl_rl.py
```
여기를 참고해. 

# Camera AI API & Docker 

-  robot 내장 카메라 Detection server 및 배포 
```
@projects/vision_ptz_api_docker/detection/Detection_Inference

@projects/vision_ptz_api_docker/detection/krm_object_detect
```

- PTZ camera API 구성 및 AI (detection & tracking 기능 추가)
아래 내용은 3가지로 구성됨. EO/IR 기능 추가

1. API를 통한 카메라 제어 (제일 중요 특히 detection을 통한 PD control tracking)
2. MediaMTX를 통한 프로토콜 변경 통신 
3. UI를 통한 시각화 

```
@projects/vision_ptz_api_docker/Gremsy_Box
```

