Add SODP state write code (set, patch, or set_in) to this project.

`$ARGUMENTS` can be:
- A key: `game.score`
- A key and operation: `game.score patch`
- A key, operation, and value: `game.player set {"name":"Alice","health":100}`

If arguments are missing, ask the user:
1. Which state key to write to
2. Which operation: `set` (replace full value), `patch` (merge fields), or `set_in` (nested field by JSON Pointer)

## What to do

1. **Detect the language** from project files.

2. **Add the write code in the appropriate location** — the currently open file, or a logical place in the codebase.

3. **Generate the correct code for the chosen operation:**

   ### set — replace full value

   **TypeScript:**
   ```typescript
   await sodp.set("$KEY", { /* your value */ });
   ```

   **Python:**
   ```python
   await client.set("$KEY", { /* your value */ })
   ```

   **Java:**
   ```java
   client.set("$KEY", yourValue).get();
   ```

   ### patch — shallow merge (only changed fields sent to watchers)

   **TypeScript:**
   ```typescript
   await sodp.patch("$KEY", { fieldToUpdate: newValue });
   ```

   **Python:**
   ```python
   await client.patch("$KEY", {"field_to_update": new_value})
   ```

   **Java:**
   ```java
   client.patch("$KEY", Map.of("fieldToUpdate", newValue)).get();
   ```

   ### set_in — set a single nested field by JSON Pointer

   **TypeScript:**
   ```typescript
   await sodp.state("$KEY").setIn("/nested/field", value);
   ```

   **Python:**
   ```python
   await client.state("$KEY").set_in("/nested/field", value)
   ```

   **Java:**
   ```java
   client.setIn("$KEY", "/nested/field", value).get();
   ```

4. **Explain the difference** between the three operations:
   - `set` — replaces the entire value; all watchers receive a delta with every changed field
   - `patch` — merges only the specified fields; untouched fields are preserved
   - `set_in` — atomically updates a single nested path; nothing else changes

5. **If the SODP client does not exist yet**, tell the user to run `/sodp-setup` first.
