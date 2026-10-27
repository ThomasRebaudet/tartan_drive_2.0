# 01: Bag ROS1 minimal, fusionné et filtré

**What to build:** une commande qui prend le dossier d'un run TartanDrive 2.0 téléchargé (les rosbags découpés en morceaux `_0` … `_N` d'environ 2 min) et qui produit un seul bag ROS1 Noetic. Ce bag ne contient que le nuage fusionné `/full_cloud` (`sensor_msgs/PointCloud2`, repère `velodyne_1`) et l'odométrie GNSS `/odometry/filtered_odom` (`nav_msgs/Odometry`, `map → gps_frame`). On doit pouvoir le rejouer directement avec `rosbag play --clock` pour alimenter le nœud heightmap. La lecture et l'écriture passent par la lib Python `rosbags` (pas besoin d'installer ROS), ajoutée aux dépendances du projet.

À noter : le typestore Noetic de `rosbags` ne connaît pas certains types présents dans les bags (par ex. `tf2_msgs/TFMessage`). Les types doivent donc être enregistrés à partir des définitions de messages embarquées dans le bag.

**Blocked by:** None (can start immediately)

**Status:** ready-for-agent

- [ ] La commande prend en entrée le dossier d'un run et un chemin de sortie, et trie les morceaux par leur indice numérique (pas par ordre alphabétique : `_10` vient après `_9`).
- [ ] Le bag produit ne contient que `/full_cloud` et `/odometry/filtered_odom`, avec les types, md5 et définitions de messages d'origine.
- [ ] Pour chaque topic, le nombre de messages est égal à la somme sur tous les morceaux.
- [ ] Les messages sont écrits dans l'ordre chronologique et les timestamps d'enregistrement sont monotones.
- [ ] Les topics source sont paramétrables, avec `/full_cloud` et `/odometry/filtered_odom` par défaut.
- [ ] Le bag se lit sans erreur avec `rosbag info` et `rosbag play` (Noetic).
