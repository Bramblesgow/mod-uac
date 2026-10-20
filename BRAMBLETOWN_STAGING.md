# Brambletown mod-uac deployment record — 2026-10-16

Status: SERVER-SIDE DEPLOYED / CLIENT PATCH NOT YET DISTRIBUTED

Production source tree: /root/brambletown-upgrade-work/core
Production build tree: /root/brambletown-upgrade-work/build-production
Production executable: /root/azerothcore-wotlk/env/dist/bin/worldserver
Module link in production source: modules/mod-uac -> /root/mod-uac-review-20261015
Upstream pinned commit: 1df8053 (2026-07-13)

Do not rebuild production from /root/azerothcore-wotlk/var/build/obj without checking module inclusion: that older source checkout lacks mod-brambletown-services. The actual deployed build includes Services, Playerbots, Dungeon Clear, mod-uac, and existing production core customizations.

Server activation:
- data/sql/db-world/ contains 14 active migration files.
- data/sql/db-paused/ retains the TWO optional hunter pet spell migrations; do not move them into db-world without deliberate approval.
- PlayerStart.CustomSpells = 1 in live worldserver.conf.
- No client-side patch-z.mpq was installed or sent.
- Client patch options are in client-patch/unlock-only, standard, enhanced; check WoW client's existing Data/patch-z.mpq and DBC overlay compatibility before selecting one.

Tests and verification:
- Combined production CMake build: PASS, 2026-10-16 10:08 CDT, exit 0.
- Binary includes mod-brambletown-services, mod-playerbots, mod-dungeon-clear, mod-uac.
- 14 mod_uac world SQL updates applied through AzerothCore DB updater.
- acore_world.playercreateinfo count 100 (baseline 62); new examples human druid, tauren paladin, draenei warlock each present.
- 2 custom warlock imp grants, 0 early hunter pet grants.
- Restart SUCCESS at 2026-10-16 10:12:51 CDT.
- Ports 3724 and 8085 listening; world ready; Playerbots and Dungeon Clear reported initialized; Errors.log empty.
- World initialization took 2 minutes 22 seconds after the full rebuild; causes not established.
- Human gameplay with unpatched client NOT YET TESTED; user to evaluate existing characters, standard character creation, NPC Services, playerbots/dungeons, connection/client errors before testing client patch.

Rollback assets: /root/backups/mod-uac-20261016/
- acore_world.before.sql.gz is a verified full predeployment world backup.
- worldserver.before is previous production binary.
- worldserver.conf.before is previous production config.
- Module includes paired manual uninstall SQL in data/sql/db-uninstall/.
- Do not restore full world backup blindly after gameplay changes; prefer selective SQL reversions and account for new race/class characters.
- Renaming db-world back to db-paused does NOT undo migrations already applied.

Generated record: server operator integration assessment and live checks.

## Deferred enhancement backlog — mod-uac secondary quest-hub trainers

- **Status:** Backlog / proposed improvement; NOT implemented or approved for deployment.
- **Purpose:** Bridge the level 5–10 training gap between each race's initial starting area and its capital city for newly unlocked race/class combinations.
- **Candidate secondary hubs:** Goldshire (Human), Kharanos (Dwarf/Gnome), Dolanaar (Night Elf), Azure Watch (Draenei), Razor Hill (Orc/Troll), Bloodhoof Village (Tauren), Brill (Undead), Falconwing Square (Blood Elf).
- **Scope:** Review the existing stock trainers at each hub and add only genuinely missing class trainers required by mod-uac combinations. Select faction-appropriate NPC trainer entries, ground-safe placement near existing trainer clusters, and appropriate gossip/training coverage. Avoid duplicates and changes to conventional race/class progression.
- **Validation:** Confirm each trainer's correct class spell offerings, spawn/location, level-range training, faction availability, guards/directions where applicable, and cross-client behavior (patched and unpatched). Validate against Brambletown Playerbots and Dungeon Clear.
- **Constraint:** Do not implement any trainer additions, introduce new trainer-introduction quests, change client patches, or restart the live server unless explicitly requested. Missing introductory trainer quests are intentionally acceptable.
