# AxAPI – patched for Minecraft / Paper 26.3 (protocol 777)

## Changes

1. **New NMS module `nms/v26_3`**
   - Based on `v26_2`, packages/group renamed to `v26_3`
   - `paperweight.paperDevBundle("26.3.build.+")`
   - Java 25 (same as other 26.x modules)

2. **`Version` enum** (`paper/.../utils/Version.java`)
   - Added `v26_3(777, "v26_3", Collections.singletonList("26.3"), true)`
   - Protocol 777 = official Java Edition 26.3 protocol

3. **Root `build.gradle`**
   - `java25NmsModules` includes `"v26_3"`
   - Authenticated publish to `repo.artillex-studios.com` disabled (no longer needs `MAVEN_USERNAME`/`MAVEN_PASSWORD` or `maven_username`/`maven_password` for a local build)

4. **`settings.gradle`**
   - Includes `nms:v26_3`

## Build requirements

- JDK 21 (main project) + JDK 25 (v26_* NMS modules)
- Several GB of RAM – paperweight userdev is memory-intensive when setting up multiple Minecraft versions
- Network access to PaperMC / Mojang

```bash
./gradlew clean build -x test
```

Main artifact: shadow JAR in `build/libs/`. The 26.x NMS handlers use `requiresDownload = true` and are fetched at runtime the same way as 26.1/26.2.

## Notes

- Minimal forward-port of the 26.2 NMS code. If Mojang introduced breaking NMS changes in 26.3, further edits inside `nms/v26_3` may be needed.
- Original: https://github.com/dpembo/AxAPI

## GitHub Actions – build & release

Workflow: `.github/workflows/build-and-release.yml`

| Event | What happens |
|--------|----------------|
| Push / PR to `master` or `main` | Build only; JARs uploaded as Actions artifacts |
| Push tag `v*` (e.g. `v2.1.9`) | Build + **GitHub Release** with the JARs attached |

### Create a release

```bash
# after committing the 26.3 changes
git tag v2.1.9
git push origin v2.1.9
```

The `release` job uses `GITHUB_TOKEN` (no extra secrets required for the GitHub Release itself).  
If you still want to publish to the private Artillex Maven repo, restore the credentials block in `build.gradle` and add a separate publish step with `MAVEN_USERNAME` / `MAVEN_PASSWORD` secrets (as in the original workflow).

### Requirements on the runner

- Ubuntu latest
- JDK 21 + JDK 25 (both installed by the workflow)
- ~4 GB heap for Gradle/paperweight

## 26.3 NMS API fixes (compile errors)

Paper/Minecraft 26.3 broke several APIs that the 26.2 NMS module used:

| Broken API | Fix applied |
|------------|-------------|
| `CraftItemStack.asCraftMirror(nms)` | `nms.asBukkitMirror()` (Paper method on NMS ItemStack) |
| `ClientboundMoveEntityPacket.PosRot(id, short, short, short, byte, byte, boolean)` | `PosRot(id, new VecDelta.Linear(xa, ya, za), yRot, xRot, onGround)` |
| `new SignText()` / `setMessage` / `DIRECT_CODEC` | `new SignText(messages, filtered, DyeColor, glowing)` + `SignText.CODEC` |
| `ClientboundOpenSignEditorPacket(pos, boolean)` | `ClientboundOpenSignEditorPacket(pos, SignTextSlot.FRONT)` |

If `SignText.CODEC` fails to resolve at compile time, try `SignText.STREAM_CODEC` only for network; NBT `store` needs a `Codec` — check the Paper 26.3 mapped name in your IDE.
