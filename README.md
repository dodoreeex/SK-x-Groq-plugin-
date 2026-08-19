# Groq Skript Addon (custom build)

**Important:** this is *not* a recompiled copy of the original Skript-Groq — I didn't have
its source. This is a fresh addon I wrote that talks to Groq's API and gives you similar
chat + memory syntax, built fresh against Skript 2.16.2 / current Paper. Read it over before
you rely on it; it hasn't been tested on a live server.

---

## 1. Build the jar (no installs needed — GitHub Actions)

1. Go to https://github.com and make a **free** account if you don't have one.
2. Click the **+** in the top right → **New repository**. Any name (e.g. `groq-skript-addon`),
   set it to **Private** if you want, then **Create repository**.
3. On the new repo's page, click **Add file → Upload files**, then drag in *everything from
   this zip* (keep the folder structure — `src/`, `.github/`, `pom.xml`, etc. all need to
   land at the repo root). Commit the upload.
4. Click the **Actions** tab at the top of the repo. A "Build" workflow should already be
   running (or click **Run workflow** if not). Wait for the green checkmark (~1-2 minutes).
5. Click into the finished run → under **Artifacts**, download **GroqSkriptAddon** → unzip it.
   Inside is `GroqSkriptAddon.jar`. That's your plugin file.

If the build fails, click into the failed step to read the error and paste it back to me —
most likely cause is the `paper-api` version needing to be pinned (see Troubleshooting below).

### Alternative: building locally
If you do have JDK 21+ and Maven installed (or an IDE like IntelliJ IDEA Community, which
bundles both), you can instead just run this from the project folder:
```
mvn package
```
The jar will be at `target/GroqSkriptAddon.jar`.

---

## 2. Install on your server (Falix)

1. In the Falix file manager, go to your `plugins/` folder.
2. Upload `GroqSkriptAddon.jar` there (Skript must also already be installed and enabled).
3. Restart the server. It will generate `plugins/GroqSkriptAddon/config.yml` and then fail
   to actually work yet — that's expected, you still need step 4.
4. Open `plugins/GroqSkriptAddon/config.yml` and replace `YOUR-GROQ-API-KEY-HERE` with a
   real key from https://console.groq.com/keys (free to create).
5. Restart the server again. Check console for `Groq Skript addon enabled — model: ...`.

---

## 3. Using it in scripts

See `example-script.sk` for working examples. Quick reference:

**Effects**
- `send groq prompt <text> [to <conversation-id>] [with model <model>]`
- `clear groq history [of <conversation-id>]`
- `set groq system prompt [of <conversation-id>] to <text>`

**Expressions** (read-only)
- `last groq response [of <conversation-id>]`
- `groq error [of <conversation-id>]` — empty string if the last send succeeded
- `groq history [of <conversation-id>]` — list of `"role: content"` strings

`<conversation-id>` is any string you choose to keep separate memories per player/context
(e.g. `"%player%"`). It defaults to `"default"` (one shared conversation) if omitted.

---

## 4. Known limitation — please read

`send groq prompt` currently waits for Groq's response on the same thread that triggered it
(a command, an event, etc.). Groq is usually fast (under a second), so in practice this is
rarely noticeable, but if a request is ever slow or times out, your server will briefly pause
for up to `timeout-seconds` (default 15s) in `config.yml`. This matches how these small AI
chat addons have typically been built, but it's worth knowing about. If this becomes a
problem in practice, it can be reworked to run off-thread with a callback event — say the
word and I can take a pass at that version.

---

## Troubleshooting the build

**"Could not resolve dependency io.papermc.paper:paper-api"** — the version range in
`pom.xml` (`[26.1.build,)`) couldn't find a match. Go to
https://repo.papermc.io/service/rest/repository/browse/maven-public/io/papermc/paper/paper-api/
find a folder matching your server's exact version, and replace the `<version>` line in
`pom.xml` for the `paper-api` dependency with that exact string (e.g. `26.1.build.42-stable`).

**"Could not resolve dependency com.github.SkriptLang:Skript:2.16.2"** — double check that
version number is really what `/plugins` (or `/skript info`) reports on your server.
