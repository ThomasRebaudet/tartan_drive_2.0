# 04: Sortie ROS2 Jazzy

**What to build:** la même commande peut produire, en plus du bag ROS1 ou à sa place, un bag ROS2 Jazzy (rosbag2, stockage mcap) avec le même contenu : nuage, odom recentrée, `/tf_static`. Le nœud heightmap version ROS2 peut ainsi être rejoué avec `ros2 bag play --clock`. Les messages sont convertis au typestore Jazzy (le `seq` des headers ROS1 disparaît). `/tf_static` est enregistré avec une QoS `transient_local`, sinon un nœud qui démarre après le début du rejeu ne reçoit jamais les extrinsèques.

La fonction de conversion de `rosbags` traduit déjà le latching ROS1 en `transient_local`. Partir du bag ROS1 généré est donc le chemin le plus court.

**Blocked by:** 01 (Bag ROS1 minimal, fusionné et filtré), 03 (`/tf_static` propre avec les extrinsèques capteurs)

**Status:** ready-for-agent

- [ ] Une option choisit la cible : ROS1, ROS2 ou les deux.
- [ ] Le bag ROS2 est en version de format 9, stockage mcap, typestore `ros2_jazzy`.
- [ ] `ros2 bag info` (Jazzy) liste les mêmes topics, avec les mêmes nombres de messages que le bag ROS1.
- [ ] Le profil QoS de `/tf_static` dans les métadonnées est `transient_local`.
- [ ] Avec un nœud lancé après le début de `ros2 bag play`, `tf2_echo vehicle velodyne_1` répond.
