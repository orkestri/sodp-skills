Add a SODP state watch to this project.

`$ARGUMENTS` should contain the state key to watch, e.g. `game.score` or `room.players`.
If no key is given, ask the user which key to watch.

## What to do

1. **Parse arguments.** Extract:
   - `key` — the SODP state key (e.g. `game.score`)
   - Optional: a type name if the user specified one (e.g. `game.score as Score`)

2. **Detect the language** from project files (package.json, pyproject.toml, pom.xml, build.gradle).

3. **Find where to add the watch** — look at the currently open file or the most relevant existing file for the detected framework.

4. **Add the watch code:**

   **TypeScript:**
   ```typescript
   import { sodp } from "@/lib/sodp"; // adjust path if needed

   const unsub = sodp.watch<YourType>("$ARGUMENTS", (value, meta) => {
     if (!meta.initialized) return; // key not yet written
     console.log("$ARGUMENTS updated:", value, "version:", meta.version);
   });

   // Call unsub() to stop watching
   ```

   **React hook:**
   ```tsx
   import { useSodpState } from "@sodp/react";

   function YourComponent() {
     const [value, meta] = useSodpState<YourType>("$ARGUMENTS");

     if (!meta?.initialized) return <div>Loading...</div>;
     return <div>{JSON.stringify(value)}</div>;
   }
   ```

   **Python:**
   ```python
   def on_update(value, meta):
       if not meta.initialized:
           return  # key not yet written
       print(f"$ARGUMENTS updated: {value}, version: {meta.version}")

   unsub = client.watch("$ARGUMENTS", on_update)
   # Call unsub() to stop watching
   ```

   **Java:**
   ```java
   Runnable unsub = client.watch("$ARGUMENTS", YourType.class, (value, meta) -> {
       if (!meta.initialized()) return; // key not yet written
       System.out.println("$ARGUMENTS updated: " + value + " v" + meta.version());
   });
   // Call unsub.run() to stop watching
   ```

5. **If the SODP client singleton does not exist yet**, tell the user to run `/sodp-setup` first, or offer to set it up now.

6. **Explain** what `meta.initialized` means (false = key has never been written to the server) and what `meta.source` values (`"cache"`, `"init"`, `"delta"`) mean.
