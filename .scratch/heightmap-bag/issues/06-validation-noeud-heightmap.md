# 06: Validation de bout en bout avec le nœud heightmap

**What to build:** le nœud heightmap de l'utilisateur tourne sur les bags générés, sous ROS1 Noetic et sous ROS2 Jazzy, et produit des heightmaps cohérentes. Ticket human-in-the-loop : le nœud n'est pas dans ce repo. Si le nœud attend d'autres noms de topics, de frames ou de champs (le champ d'intensité de `/full_cloud` s'appelle `i` et non `intensity`), on les note ici et on ouvre un ticket de remapping.

**Blocked by:** 02 (Odom GNSS recentrée), 03 (`/tf_static` propre), 04 (Sortie ROS2 Jazzy). Le ticket 05 est optionnel, nécessaire seulement si le décalage d'environ 0.5 m GNSS/véhicule pose problème.

**Status:** needs-human

- [ ] Le nœud tourne sans erreur de TF ni de type, avec `use_sim_time`, sur le bag Noetic.
- [ ] Idem sur le bag Jazzy.
- [ ] La heightmap produite est comparée visuellement à `/local_height_map` (la heightmap TartanDrive, présente dans les bags bruts) sur au moins un run. Pas de dérive visible ni de décalage systématique.
- [ ] Les écarts de nommage attendus par le nœud (topics, frames, champs) sont listés ici.
