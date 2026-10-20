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

## Hunter pet spells — approved GitHub activation candidate (2026-10-19)

The optional Hunter pet SQL files have been moved from `data/sql/db-paused/` into `data/sql/db-world/` in the activation pull request. This is a **repository change only**. The 2026-10-16 production deployment record above remains historical; do not interpret merging this PR as a live database migration, a build, or a server restart.

- Scope: **all Hunters** receive Tame Beast (1515), Call Pet (883), Dismiss Pet (2641), Feed Pet (6991), and Revive Pet (982) at character creation. In the Brambletown production core checkout, `PlayerStorage.cpp` calls `LearnCustomSpells()` during `Player::LoadFromDB()`, so existing Hunters should also acquire missing spells on login after the server has loaded the migrations; this needs in-game confirmation. This intentionally bypasses the level-10 pet quest gate for standard and newly supported races.
- Dependencies: `PlayerStart.CustomSpells = 1` (already configured in production); install **both** paired SQL migrations in the same release. The `spell_dbc` overlay lowers server-side spell BaseLevel/SpellLevel to 1.
- Deployment: update the deployed module checkout to the reviewed merged commit, back up the affected world DB tables, restart the worldserver during an approved maintenance window, and verify both migrations were applied. Until that restart, the live server remains unchanged.
- Regression checks: new level-1 Human Hunter and original-race Hunter learn all five spells and can tame/use a pet; existing Human Hunter gets missing spells on next login; pet behavior, ordinary Hunter talents/spells, and a conventional level-10 Hunter quest chain remain functional. Validate client UI/casting in game.
- Rollback preflight: **before deployment**, back up `playercreateinfo_spell_custom` and `spell_dbc` and query `spell_dbc` for IDs `1515`, `883`, `2641`, `6991`, and `982`. The Brambletown world DB contained **zero** matching `spell_dbc` rows when checked on 2026-10-19; confirm again at deployment. The supplied paired `data/sql/db-uninstall/mod_uac_hunter_pet_*` scripts delete those five `spell_dbc` IDs rather than restoring previous row values. Use the DBC uninstall unchanged **only if the five IDs were absent before installation**; otherwise selectively restore each pre-existing row from the captured backup instead of deleting it, preserving other modules' overrides. The custom-spell install **deletes and replaces** rows keyed by `(racemask, classmask, Spell)` for `(0, 4, 1515)`, `(0, 4, 883)`, `(0, 4, 2641)`, `(0, 4, 6991)`, and `(0, 4, 982)`. Before installing, query and separately save any matching `playercreateinfo_spell_custom` rows **including their original `Note` values**; these may belong to other modules. If matching rows exist, confirm overwriting them is acceptable before deployment. On rollback, execute the custom-spell uninstall to remove the five UAC grants, **then restore only the exact pre-install matching rows** from the saved backup, preserving all original columns and leaving unrelated rows untouched. If no rows previously existed for those keys, the custom-spell uninstall needs no restoration. Moving SQL files back does **not** undo already-applied migrations. Characters that learned spells are **not automatically unlearned** by reverting database rows; do not globally remove spells without separate approval.
- No gameplay executable or client patch is changed in this PR. No live deployment, restart, or database changes are authorized by a GitHub merge.
