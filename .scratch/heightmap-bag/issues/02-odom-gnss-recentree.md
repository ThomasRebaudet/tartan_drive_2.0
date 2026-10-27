# 02: Odom GNSS recentrée sur l'origine du run

**What to build:** dans le bag généré, l'odométrie GNSS est exprimée relativement au premier fix du run, et non plus en coordonnées globales type UTM (x ≈ 4.48e6, y ≈ -6.0e5). Le nœud heightmap travaille ainsi avec des petites valeurs, sans perte de précision en float32. L'origine est conservée dans un fichier annexe à côté du bag, pour pouvoir revenir aux coordonnées globales (par ex. pour superposer sur la carte gascola).

**Blocked by:** 01 (Bag ROS1 minimal, fusionné et filtré)

**Status:** ready-for-agent

- [ ] La première pose d'odom du bag généré vaut (0, 0, 0) en position.
- [ ] Pour chaque message, origine + position recentrée redonne la position d'origine (à la précision float64 près).
- [ ] L'orientation, le twist, les covariances, les stamps et les `frame_id` sont inchangés.
- [ ] Le fichier annexe contient l'origine (x, y, z), le topic et le `frame_id` source, et le stamp du fix de référence.
- [ ] On peut désactiver le recentrage avec une option.
