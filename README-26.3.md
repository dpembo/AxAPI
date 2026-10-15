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
