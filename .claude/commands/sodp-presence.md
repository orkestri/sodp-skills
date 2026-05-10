Add SODP presence to this project.

Presence binds a value to a session-scoped path. When the client disconnects the server automatically removes it and notifies all watchers — no ghost cursors, no stale "online" indicators.

`$ARGUMENTS` can be:
- A key: `collab.cursors`
- A key and path: `collab.cursors /user_alice`

If arguments are missing, ask the user:
1. Which state key holds presence data (e.g. `collab.cursors`, `room.members`)
2. What path identifies this session (e.g. `/user_${userId}`, `/instance_${hostname}`)
3. What payload to store (e.g. `{ name, color, line }` for a cursor)

## What to do

1. **Detect the language** from project files.

2. **Add the presence registration code** where the session starts (on connect, on mount, on login, etc.).

3. **Add the watcher** for the presence key so the user can render other sessions.

   **TypeScript:**
   ```typescript
   // Register this session's presence — auto-removed on disconnect
   await sodp.presence("$KEY", `/user_${userId}`, {
     name: userName,
     // add any fields you want visible to others
   });

   // Watch all presence entries
   const unsub = sodp.watch<Record<string, { name: string }>>("$KEY", (cursors, meta) => {
     const others = Object.entries(cursors ?? {}).filter(([id]) => id !== `user_${userId}`);
     renderPresence(others);
   });
   ```

   **React:**
   ```tsx
   import { useSodpState, useSodpClient } from "@sodp/react";
   import { useEffect } from "react";

   function usePresence(key: string, path: string, payload: unknown) {
     const client = useSodpClient();
     useEffect(() => {
       client.presence(key, path, payload);
       // no cleanup needed — server removes it on disconnect
     }, []);
   }

   function CollabLayer({ userId, userName }: { userId: string; userName: string }) {
     usePresence("$KEY", `/user_${userId}`, { name: userName });

     const [cursors] = useSodpState<Record<string, { name: string }>>("$KEY");
     return (
       <>
         {Object.entries(cursors ?? {})
           .filter(([id]) => id !== `user_${userId}`)
           .map(([id, data]) => (
             <Cursor key={id} name={data.name} />
           ))}
       </>
     );
   }
   ```

   **Python:**
   ```python
   # Register presence — auto-removed when client.close() is called or connection drops
   await client.presence("$KEY", f"/user_{user_id}", {
       "name": user_name,
   })

   # Watch all presence entries
   def on_presence(cursors, meta):
       others = {k: v for k, v in (cursors or {}).items() if k != f"user_{user_id}"}
       print("online:", others)

   unsub = client.watch("$KEY", on_presence)
   ```

   **Java:**
   ```java
   // Register presence
   client.presence("$KEY", "/user_" + userId,
       Map.of("name", userName)).get();

   // Watch all presence entries
   Runnable unsub = client.watch("$KEY", JsonNode.class, (cursors, meta) -> {
       if (cursors == null) return;
       cursors.fields().forEachRemaining(entry -> {
           if (!entry.getKey().equals("user_" + userId)) {
               renderCursor(entry.getKey(), entry.getValue());
           }
       });
   });
   ```

4. **Explain** that no cleanup is needed on the client side — the server broadcasts a REMOVE delta to all watchers automatically when the session disconnects.

5. **If the SODP client does not exist yet**, tell the user to run `/sodp-setup` first.
