# 05: Relier `gps_frame` à `vehicle`

**What to build:** l'odom GNSS donne la pose de `gps_frame`, mais `gps_frame` n'apparaît nulle part dans l'arbre TF des bags. Le nœud heightmap ne connaît donc pas le décalage entre le point suivi par le GNSS et le repère du nuage (`vehicle`/`velodyne_1`). Hypothèse ⚠️ : `gps_frame` correspond à `novatel/imu_frame`, à environ (-0.508, -0.05, -0.152) m de `vehicle`. Ce ticket confirme ou infirme l'hypothèse, puis rend le bag cohérent, au choix : ajouter `vehicle → gps_frame` dans `/tf_static`, ou exprimer l'odom directement pour `vehicle`. La décision est à noter dans le ticket.

**Blocked by:** 03 (`/tf_static` propre avec les extrinsèques capteurs)

**Status:** ready-for-agent

- [ ] L'hypothèse sur `gps_frame` est confirmée ou infirmée à partir des données (comparaison avec la trajectoire SuperOdom `/integrated_to_init` et la TF `sensor_init → vehicle`, ou avec la config de collecte du run), et la conclusion est notée ici.
- [ ] En ligne droite, le cap déduit de l'orientation de l'odom est cohérent avec la direction de la vitesse (écart médian < 5°).
- [ ] Le bag généré permet de résoudre la pose de `vehicle` dans le repère de l'odom (via TF ou via l'odom elle-même).
- [ ] Le comportement est identique dans les sorties ROS1 et ROS2.
