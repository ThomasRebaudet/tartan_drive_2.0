# 03: `/tf_static` propre avec les extrinsèques capteurs

**What to build:** le bag généré fournit l'arbre TF dont le nœud heightmap a besoin pour ramener `/full_cloud` (repère `velodyne_1`) dans le repère véhicule, et rien de plus. Le `/tf` d'origine n'est pas recopié : il contient la pose SuperOdometry (`sensor_init → vehicle`, `map → sensor_init`, `multisense_init → …`), qui contredirait l'odom GNSS. À la place, on écrit un `/tf_static` latched au début du bag. Il est construit à partir des transformations `vehicle → capteur` observées dans le bag (`velodyne_1`, `velodyne_2`, `livox_frame`, `multisense/head`, `novatel/imu_frame`) et des statiques Multisense d'origine.

Dans le run de référence, `vehicle → velodyne_1` est l'identité (cohérent avec l'extrinsics.yaml du repo). Ces transformations sont publiées en boucle sur `/tf` alors qu'elles devraient être constantes, et il faut le vérifier.

**Blocked by:** 01 (Bag ROS1 minimal, fusionné et filtré)

**Status:** ready-for-agent

- [ ] Le bag généré ne contient aucune transformation qui fait intervenir `map`, `sensor_init`, `sensor_init_rot` ou `multisense_init`.
- [ ] Un unique message `/tf_static` latched est écrit au premier timestamp du bag, et contient au minimum `vehicle → velodyne_1`.
- [ ] Le script émet un avertissement si une extrinsèque varie au cours du run (au-delà de 1 mm ou 0.1°).
- [ ] Au rejeu sous Noetic, `tf_echo vehicle velodyne_1` répond dès le premier nuage.
