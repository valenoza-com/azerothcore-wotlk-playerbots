# Downstream patches applied at image build time

These patches live outside the `modules/` submodules so the submodule pins stay
identical to upstream. `apps/docker/Dockerfile` applies every `*.patch` in this
directory with `patch -p1` right after the sources are copied into the image.

Keep patches small, and delete them once the upstream module ships the change.

## `mod-npc-talent-template-console-apply.patch`

**Problem.** `mod-npc-talent-template` can only apply one of its 408 stored
class/spec templates from its gossip NPC: `.templatenpc reload|create` are
declared `Console::No`, and AzerothCore refuses `Console::No` commands for the
console and therefore for SOAP. A web store (this realm's portal) can mail gear
with `.send items` but has no way to set talents and glyphs.

**Change.** Adds two `Console::Yes` subcommands:

| Command | Effect |
| --- | --- |
| `.templatenpc apply "<TemplateName>" <PlayerName>` | Applies a stored template to an **online** player. Validates class match, equipped gear and spent talent points, and reports failures on the command result so the caller (SOAP) sees success or a reason. |
| `.templatenpc list [class filter]` | Lists `class \| spec \| category \| level range \| mask`, optionally filtered by a case-insensitive class substring. |

`apply` resolves the player with `ObjectAccessor::FindPlayerByName`, so it only
works for characters that are currently logged in — which is also the only state
in which talent changes can be persisted safely.

**When to drop it.** Once the module exposes a console-reachable apply command
upstream, delete this file; the Dockerfile iterates the directory and will
simply have nothing to apply.
