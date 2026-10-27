# TartanDrive 2.0 — Canaux et signaux du dataset

Ce document décrit tous les canaux disponibles dans TartanDrive 2.0, sous ses deux formes :

1. **Dataset au format KITTI** (`kitti/all_topics/<run>/`) : signaux extraits, ré-échantillonnés et synchronisés.
2. **Rosbags bruts** (`bags/<run>/*.bag`) : tous les topics ROS enregistrés sur le véhicule.

> **Méthode.** Les formes, types et plages de valeurs ci-dessous ont été mesurés sur le run
> `2023-10-26-14-42-35_turnpike_afternoon_fall`. Ils viennent de `data_extraction.log`, des fichiers `.npy`
> et des en-têtes des `.tar`. Les points marqués **⚠️** n'ont pas de documentation officielle :
> ce sont des interprétations tirées des valeurs observées, à vérifier avant usage.

Véhicule : Yamaha Viking (ATV). Capteurs : Multisense S21 (stéréo + IMU), 2× Velodyne, Livox, Novatel (GNSS/INS),
Racepak (roues / amortisseurs / commandes), moteur de direction Talon. Calibration : [calibration/extrinsics.yaml](calibration/extrinsics.yaml),
[calibration/multisense_intrinsics.txt](calibration/multisense_intrinsics.txt).

---

## 1. Synchronisation temporelle (format KITTI)

- **Horloge de référence : 10 Hz** (`main freq`). Chaque modalité est extraite à un multiple de cette fréquence (1×, 5×, 10× ou 40×).
- Les signaux sont ré-alignés sur une grille régulière. Une frame absente est **remplie avec la frame précédente** (*filled missing frame*, voir `data_extraction.log`).
- Chaque dossier de signal contient `timestamps.txt` : une ligne par échantillon, en temps Unix (secondes, float). Le nombre de lignes est égal à la 1ʳᵉ dimension du `.npy`.
- Dans les archives `.tar`, les frames s'appellent `NNNNNN.<ext>` (6 chiffres). L'indice `i` correspond à l'échantillon `i` de la grille à 10 Hz (même grille que `cmd/timestamps.txt`).
- Exemple (turnpike_afternoon_fall) : 504 s → 5041 frames à 1×, 25205 à 5×, 50410 à 10×, 201640 à 40×.

---

## 2. Signaux temporels (`.npy` + `timestamps.txt`)

Tous les tableaux sont en `float64`. N = nombre d'échantillons.

| Dossier | Fichier | Topic ROS source | Fréq. | Forme | Colonnes | Unités / plage observée | Description |
|---|---|---|---|---|---|---|---|
| `cmd` | `twist.npy` | `/cmd` | 10 Hz | (N, 2) | `[throttle, steering]` | throttle ∈ [0, 1], steering ∈ [-1, 1] | Commande envoyée au véhicule (joystick ou contrôleur), normalisée. |
| `controls` | `control.npy` | `/controls` | 50 Hz | (N, 2) | `[c0, c1]` ⚠️ | c0 ≈ 4–21, c1 ≈ 0–6550 | Mesures Racepak des actionneurs (probablement position d'accélérateur et pression de frein). Ordre et unités ⚠️. |
| `steering_angle` | `float.npy` | `/ros_talon/steering_angle` | 10 Hz | (N,) | angle | ≈ -307 … 415 (deg ⚠️) | Angle de direction mesuré par le moteur Talon. |
| `current_position` | `float.npy` | `/ros_talon/current_position` | 10 Hz | (N,) | position | ≈ -283 … 413 | Position du moteur de direction Talon (topic natif à ~6 Hz, donc beaucoup de frames répétées). |
| `wheel_rpm` | `encoder.npy` | `/wheel_rpm` | 50 Hz | (N, 4) | 4 roues (FL, FR, RL, RR ⚠️) | rpm, ≈ 0–370 (pics aberrants possibles, ex. 10327) | Vitesse de rotation des roues (Racepak). |
| `shock_pos` | `shock.npy` | `/shock_pos` | 50 Hz | (N, 4) | 4 amortisseurs (FL, FR, RL, RR ⚠️) | ≈ 4.5–6.9 (pouces ⚠️) ; colonnes 0–1 à 0 sur ce run | Course des suspensions (Racepak). |
| `novatel_imu` | `imu.npy` | `/novatel/imu/data` | 100 Hz | (N, 6) | `[ωx, ωy, ωz, ax, ay, az]` | rad/s, m/s² | IMU du Novatel (GNSS/INS). |
| `multisense_imu` | `imu.npy` | `/multisense/imu/imu_data` | 400 Hz | (N, 6) | `[ωx, ωy, ωz, ax, ay, az]` | rad/s, m/s² | IMU intégrée à la tête stéréo Multisense. |
| `gps_odom` | `odometry.npy` | `/odometry/filtered_odom` | 50 Hz | (N, 13) | `[x, y, z, qx, qy, qz, qw, vx, vy, vz, ωx, ωy, ωz]` | position en m (coordonnées globales type UTM, x ≈ 4.48e6), quaternion, m/s, rad/s | Odométrie GNSS/INS Novatel filtrée. Elle sert de vérité terrain globale. |
| `super_odom` | `odometry.npy` | `/integrated_to_init` | 100 Hz | (N, 13) | idem | position en m dans le repère local du départ, vitesses dans le repère du véhicule | Odométrie LiDAR-inertielle SuperOdometry. |
| `tartanvo_odom` | `odometry.npy` | `/tartanvo_odom` | 10 Hz | (N, 13) | idem (vitesses toujours à 0) | m, quaternion | Odométrie visuelle stéréo TartanVO (pose seule). |
| `traversability_cost` | `float.npy` | `/traversability_cost` | 100 Hz | (N,) | coût | ≈ 0.01–1.7 | Coût de traversabilité scalaire, calculé à partir de la rugosité ressentie (algo `roughness_cost`). |
| `traversability_breakdown` | `float.npy` | `/traversability_breakdown` | 100 Hz | (N, 8) | 8 composantes ⚠️ | c0, c3, c4 ∈ [0, 3] ; c1, c2, c5, c6 ≈ 0–0.04 ; c7 = 0 | Décomposition du coût de traversabilité. Signification des composantes ⚠️. |

Format de chaque odométrie (13 colonnes) :

| Index | 0–2 | 3–6 | 7–9 | 10–12 |
|---|---|---|---|---|
| Contenu | position `x, y, z` | orientation quaternion `qx, qy, qz, qw` | vitesse linéaire `vx, vy, vz` | vitesse angulaire `ωx, ωy, ωz` |

---

## 3. Modalités par frame (archives `.tar`, 10 Hz)

Chaque archive contient un fichier (ou une paire de fichiers) par frame, nommé par l'indice de frame.
Les tailles sont celles du run de référence (~8 min 24 s de données extraites).

| Archive | Topic ROS source | Fichiers par frame | Format | Description | Taille (exemple) | Dispo. |
|---|---|---|---|---|---|---|
| `image_left.tar` | `/multisense/left/image_rect` | `NNNNNN.png` | PNG niveaux de gris `L`, 544×1024, uint8 | Image rectifiée, caméra gauche Multisense. | 1.6 GB | 55/55 |
| `image_right.tar` | `/multisense/right/image_rect` | `NNNNNN.png` | PNG `L`, 544×1024, uint8 | Image rectifiée, caméra droite (baseline ≈ 0.21 m). | 1.7 GB | 55/55 |
| `image_left_color.tar` | `/multisense/left/image_rect_color` | `NNNNNN.png` | PNG `RGB`, 544×1024×3, uint8 | Image couleur rectifiée, caméra gauche. | 5.1 GB | 54/55 |
| `depth_left.tar` | — (post-traitement stéréo) | `NNNNNN.npy` + `NNNNNN.jpg` | npy float32 (512, 640) + JPG RGB 512×640 (visualisation) | Carte de profondeur stéréo de la caméra gauche (m ⚠️). Le JPG en est une visualisation colorisée. | 6.8 GB | 55/55 |
| `velodyne_0.tar` | `/velodyne_1/velodyne_points` | `NNNNNN.npy` + `NNNNNN_intensity.npy` | xyz float32 (P, 3) + intensité uint8 (P,) | Nuage du Velodyne n°1. | 2.3 GB | 55/55 |
| `velodyne_1.tar` | `/velodyne_2/velodyne_points` | `NNNNNN.npy` + `NNNNNN_intensity.npy` | xyz float32 (P, 3) + intensité uint8 (P,) | Nuage du Velodyne n°2 (incliné ~10°, cf. extrinsèques). | 3.1 GB | 55/55 |
| `livox.tar` | `/livox/lidar` | `NNNNNN.npy` + `NNNNNN_intensity.npy` | xyz float32 (P, 3) + intensité uint8 (P,) | Nuage du LiDAR Livox (FOV frontal, ~10k pts, portée ≈ 70 m). | 0.7 GB | 55/55 |
| `full_cloud.tar` | `/full_cloud` | `NNNNNN.npy` | xyz float32 (P, 3), ~80k pts | Fusion des nuages des deux Velodyne (*lidar_stitching*). | 5.3 GB | 54/55 |
| `stereo_colored_point_cloud.tar` | `/deep_cloud` ⚠️ | `NNNNNN.npy` + `NNNNNN_color.npy` | xyz float32 (P, 3) + RGB uint8 (P, 3), ~150k pts | Nuage stéréo dense colorisé (portée 2.5–100 m). | 11.7 GB | 55/55 |
| `stereo_colored_point_cloud_gmf.tar` | — (post-traitement) | `NNNNNN.npy` + `NNNNNN_color.npy` | xyz float32 (P, 3) + RGB uint8 (P, 3) | Nuage stéréo colorisé filtré (*gmf*, filtre ⚠️), plus épars. | 2.9 GB | 55/55 |
| `height_map.tar` | `/local_height_map_inflate` | `NNNNNN.npy` | float32 (600, 600, 4) | Carte de hauteur locale centrée sur le véhicule. Canaux ⚠️ : `[h_min, h_max, h_moy, écart]` en m ; cellules inconnues à ±1e6. | 29 GB | 12/55 |
| `rgb_map.tar` | `/local_rgb_map_inflate` | `NNNNNN.png` | PNG `RGB`, 600×600×3, uint8 | Carte RGB vue de dessus (BEV), alignée sur `height_map`. | 3.9 GB | 12/55 |
| `local_gridmap.tar` | `/local_gridmap` | `NNNNNN_data.npy` | float32 (16, 480, 480), canaux en premier | Grille BEV locale à 16 couches (géométrie + couleur), voir ci-dessous. | 74 GB | 55/55 |
| `local_dino_map.tar` | — (post-traitement) | `NNNNNN_data.npy` | float32 (16, 480, 480) | Grille BEV de 16 descripteurs visuels DINO (réduits, valeurs ≈ [-0.6, 0.6]), même grille que `local_gridmap`. | 74 GB | 55/55 |

P = nombre de points, variable d'une frame à l'autre.

### Couches de `local_gridmap` (plages observées, noms non fournis ⚠️)

| Couche | Plage | Interprétation probable ⚠️ |
|---|---|---|
| 0–3 | ≈ -14.6 … 4.9 | Hauteurs (min / moyenne / max / …), en m |
| 4 | 0 … 2.5 | Écart de hauteur / épaisseur |
| 5 | ≈ -14.9 … 2.4 | Hauteur du sol / terrain estimé |
| 6 | 0 … 1.41 | Pente |
| 7 | 0 … 0.31 | Rugosité |
| 8 | {0, 1} | Masque binaire (cellule connue / obstacle) |
| 9–11 | 0 … 1 | Couleur RGB normalisée |
| 12–13 | -1 … 1 | Composantes de normale ou de pente (x, y) |
| 14 | 0 … 500 | Nombre de points / densité |
| 15 | 0 … 6.2 | Autre statistique géométrique |

### Fichiers annexes par run

| Fichier | Format | Description |
|---|---|---|
| `kitti/.../data_extraction.log` | texte | Journal d'extraction : topics, fréquences, frames ré-appariées ou remplies. |
| `bags/<run>/info.yaml` | YAML | Métadonnées : durée, vitesse max/moyenne, conditions, parcours, pilote, capteurs, algorithmes actifs. |
| `bags/<run>/config.yaml` | YAML | Configuration de la collecte (launch files, topics par capteur). |
| `bags/<run>/gps.npy` | npy | Trace GPS du run (utilisée par la GUI pour l'aperçu sur la carte). |
| `bags/<run>/*_topics.yaml` | YAML | Fréquence et nombre de messages de chaque topic du bag. |

---

## 4. Topics ROS bruts (rosbags)

Fréquences mesurées sur `turnpike_afternoon_fall_0.bag` (~2 min). Les types de messages sont ceux attendus pour chaque driver ou algorithme ⚠️ (non listés dans les métadonnées). **✔** = extrait dans le format KITTI.

### Capteurs

| Topic | Fréq. | Type probable | KITTI | Description |
|---|---|---|---|---|
| `/multisense/left/image_rect` | 10 Hz | `sensor_msgs/Image` | ✔ `image_left` | Image gauche rectifiée, niveaux de gris |
| `/multisense/left/image_rect_color` | 10 Hz | `sensor_msgs/Image` | ✔ `image_left_color` | Image gauche rectifiée, couleur |
| `/multisense/right/image_rect` | 10 Hz | `sensor_msgs/Image` | ✔ `image_right` | Image droite rectifiée |
| `/multisense/imu/imu_data` | ~400 Hz | `sensor_msgs/Imu` | ✔ `multisense_imu` | IMU Multisense |
| `/velodyne_1/velodyne_points` | 10 Hz | `sensor_msgs/PointCloud2` | ✔ `velodyne_0` | Nuage Velodyne 1 |
| `/velodyne_2/velodyne_points` | 10 Hz | `sensor_msgs/PointCloud2` | ✔ `velodyne_1` | Nuage Velodyne 2 |
| `/velodyne_1/velodyne_packets`, `/velodyne_2/velodyne_packets` | 10 Hz | `velodyne_msgs/VelodyneScan` | | Paquets bruts Velodyne |
| `/velodyne_1/scan`, `/velodyne_2/scan` | 10 Hz | `sensor_msgs/LaserScan` | | Projection 2D (laserscan) des Velodyne |
| `/velodyne_cloud_2` | 10 Hz | `sensor_msgs/PointCloud2` | | Nuage Velodyne pré-traité |
| `/livox/lidar` | 10 Hz | `sensor_msgs/PointCloud2` | ✔ `livox` | Nuage Livox |
| `/novatel/imu/data` | 125 Hz | `sensor_msgs/Imu` | ✔ `novatel_imu` | IMU Novatel |
| `/odometry/filtered_odom` | 62 Hz | `nav_msgs/Odometry` | ✔ `gps_odom` | Odométrie GNSS/INS filtrée |
| `/odom` | 62 Hz | `nav_msgs/Odometry` | | Odométrie Novatel non filtrée |
| `/span/status` | 50 Hz | Novatel | | Statut INS SPAN Novatel |
| `/vehicle_state/velocity` | 62 Hz | — | | Vitesse du véhicule |
| `/wheel_rpm` | 60 Hz | `std_msgs/Float64MultiArray` | ✔ `wheel_rpm` | Vitesse des 4 roues (Racepak) |
| `/shock_pos` | 60 Hz | `std_msgs/Float64MultiArray` | ✔ `shock_pos` | Course des 4 amortisseurs (Racepak) |
| `/controls` | 60 Hz | `std_msgs/Float64MultiArray` | ✔ `controls` | Actionneurs mesurés (Racepak) |
| `/received_messages`, `/sent_messages` | 250 Hz | `can_msgs/Frame` | | Trames CAN reçues / envoyées |

### Commande et pilotage

| Topic | Fréq. | Type probable | KITTI | Description |
|---|---|---|---|---|
| `/cmd` | 124 Hz | `geometry_msgs/TwistStamped` | ✔ `cmd` | Commande (throttle, steering) |
| `/joy` | 83 Hz | `sensor_msgs/Joy` | | Joystick brut |
| `/mux/joy` | 31 Hz | `sensor_msgs/Joy` | | Joystick après multiplexeur |
| `/mux/intervention` | 31 Hz | `std_msgs/Bool` | | Indique une reprise en main par l'opérateur |
| `/ros_talon/steering_angle` | 31 Hz | `std_msgs/Float32` | ✔ `steering_angle` | Angle de direction |
| `/ros_talon/current_position` | 6 Hz | `std_msgs/Float32` | ✔ `current_position` | Position du moteur de direction |
| `/ros_talon/status` | 20 Hz | — | | Statut du contrôleur Talon |
| `/dynamixel_state`, `/goal_dynamixel_position` | 4 / 31 Hz | — | | État et consigne du servo Dynamixel (frein / accélérateur ⚠️) |
| `/joint_states` | 4 Hz | `sensor_msgs/JointState` | | États articulaires |

### Odométrie et SLAM

| Topic | Fréq. | Type probable | KITTI | Description |
|---|---|---|---|---|
| `/integrated_to_init` | 124 Hz | `nav_msgs/Odometry` | ✔ `super_odom` | SuperOdometry, odométrie IMU intégrée haute fréquence |
| `/aft_mapped_to_init` | 10 Hz | `nav_msgs/Odometry` | | SuperOdometry, pose après mapping |
| `/lidar_odometry` | 10 Hz | `nav_msgs/Odometry` | | Odométrie LiDAR pure |
| `/super_odometry_stats`, `/superodom/feature_info` | 10 Hz | — | | Statistiques internes de SuperOdometry |
| `/edge_points`, `/planar_points` | 10 Hz | `sensor_msgs/PointCloud2` | | Points caractéristiques (arêtes / plans) |
| `/velodyne_cloud_registered_with_features` | 10 Hz | `sensor_msgs/PointCloud2` | | Nuage recalé avec ses features |
| `/tartanvo_odom` | 10 Hz | `nav_msgs/Odometry` | ✔ `tartanvo_odom` | Odométrie visuelle TartanVO |
| `/tartanvo_pose`, `/tartanvo_transform` | 10 Hz | `PoseStamped` / `TransformStamped` | | Pose TartanVO, sous deux formes |

### Perception, cartographie, traversabilité

| Topic | Fréq. | Type probable | KITTI | Description |
|---|---|---|---|---|
| `/full_cloud` | 10 Hz | `sensor_msgs/PointCloud2` | ✔ `full_cloud` | Nuage fusionné des deux Velodyne |
| `/deep_cloud` | 10 Hz | `sensor_msgs/PointCloud2` | ✔ `stereo_colored_point_cloud` ⚠️ | Nuage stéréo dense (réseau profond) |
| `/local_height_map`, `/local_height_map_inflate` | 10 Hz | `sensor_msgs/Image` (multicanal) | ✔ `height_map` (inflate) | Carte de hauteur locale (brute / dilatée) |
| `/local_rgb_map`, `/local_rgb_map_inflate` | 10 Hz | `sensor_msgs/Image` | ✔ `rgb_map` (inflate) | Carte RGB locale (brute / dilatée) |
| `/local_gridmap` | 10 Hz | `grid_map_msgs/GridMap` | ✔ `local_gridmap` | Grille locale multicouche |
| `/debug/ground/{raw, estimated, inflated, noise, object, roughness, slope_magnitude, slope_x, slope_y, svd1, svd2, svd3}` | 1 Hz | `sensor_msgs/Image` / GridMap | | Couches de debug de l'estimation du sol (hauteur brute / estimée, bruit, objets, rugosité, pentes, valeurs singulières SVD) |
| `/debug/voxel/density` | 1 Hz | — | | Densité de la grille voxel |
| `/local_mapping_node/parameter_*` | — | `dynamic_reconfigure` | | Paramètres du nœud de mapping |
| `/traversability_cost` | 124 Hz | `std_msgs/Float32` | ✔ `traversability_cost` | Coût de traversabilité |
| `/traversability_breakdown` | 124 Hz | `std_msgs/Float32MultiArray` | ✔ `traversability_breakdown` | Composantes du coût |

### Système

| Topic | Fréq. | Description |
|---|---|---|
| `/tf`, `/tf_static` | — / 5 Hz | Arbre des transformations (voir aussi [calibration/extrinsics.yaml](calibration/extrinsics.yaml)) |
| `/clock` | 250 Hz | Horloge ROS |
| `/diagnostics` | 6 Hz | Diagnostics des drivers |
| `/rosout`, `/rosout_agg` | — | Logs ROS |
| `/velodyne_*/…_nodelet_manager*/…` | — | Bond et paramètres des nodelets Velodyne |

---

## 5. Repères et calibration (résumé)

Extrait de [calibration/extrinsics.yaml](calibration/extrinsics.yaml) : transformations depuis `vehicle` vers `multisense/head`, `velodyne_1`, `velodyne_2`, `livox` et `novatel/imu_frame`, plus `multisense/head → multisense/left_camera_optical_frame`.

Intrinsèques Multisense (images 544×1024, rectifiées), tirées de [calibration/multisense_intrinsics.txt](calibration/multisense_intrinsics.txt) :
`fx = fy = 477.60`, `cx = 499.5`, `cy = 252.0`, baseline = 100.15 / 477.60 ≈ **0.210 m**.
