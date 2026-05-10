Integrate the SODP (State-Oriented Data Protocol) client into this project.

SODP is a WebSocket protocol for continuous state sync. Clients subscribe to named keys and receive only the changed fields as deltas — no polling, no full objects on every update.

## What to do

1. **Detect the language/framework** by inspecting the project files:
   - `package.json` with React → TypeScript + React
   - `package.json` without React → TypeScript/JavaScript
   - `pyproject.toml` or `requirements.txt` → Python
   - `pom.xml` → Java (Maven)
   - `build.gradle` or `build.gradle.kts` → Java (Gradle)

   If `$ARGUMENTS` specifies a language (e.g. "python", "typescript", "react", "java"), use that instead.

2. **Install the dependency:**

   - TypeScript/JS: `npm install @sodp/client`
   - React: `npm install @sodp/client @sodp/react`
   - Python: `pip install sodp-client`
   - Java Maven — add to `pom.xml`:
     ```xml
     <dependency>
       <groupId>site.orkestri</groupId>
       <artifactId>sodp-client</artifactId>
       <version>0.2.2</version>
     </dependency>
     ```
   - Java Gradle — add to `build.gradle`:
     ```groovy
     implementation 'site.orkestri:sodp-client:0.2.2'
     ```

3. **Determine the server URL.** Look for it in:
   - Environment variables: `SODP_URL`, `NEXT_PUBLIC_SODP_URL`, `VITE_SODP_URL`
   - Existing config files
   - If not found, use `ws://localhost:7777` and leave a comment

4. **Create a SODP client file** appropriate for the project structure:

   **TypeScript** — create `src/lib/sodp.ts` (or nearest equivalent):
   ```typescript
   import { SodpClient } from "@sodp/client";

   const SODP_URL = process.env.SODP_URL ?? "ws://localhost:7777";

   export const sodp = new SodpClient(SODP_URL);
   ```

   **React** — create `src/providers/SodpProvider.tsx` and wrap the app:
   ```tsx
   import { SODPProvider } from "@sodp/react";

   const SODP_URL = import.meta.env.VITE_SODP_URL ?? "ws://localhost:7777";

   export function AppSodpProvider({ children }: { children: React.ReactNode }) {
     return <SODPProvider url={SODP_URL}>{children}</SODPProvider>;
   }
   ```
   Then show the user where to add it (usually `main.tsx` or `App.tsx`).

   **Python** — create `sodp_client.py` (or `src/sodp_client.py`):
   ```python
   import os
   from sodp import SodpClient

   SODP_URL = os.getenv("SODP_URL", "ws://localhost:7777")

   client = SodpClient(SODP_URL)
   ```

   **Java** — create `src/main/java/.../SodpConfig.java` using the dominant framework:

   Spring Boot:
   ```java
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import site.orkestri.sodp.SodpClient;

   @Configuration
   public class SodpConfig {
       @Bean
       public SodpClient sodpClient(@Value("${sodp.url:ws://localhost:7777}") String url) {
           return SodpClient.builder(url).build();
       }
   }
   ```

   Plain Java:
   ```java
   import site.orkestri.sodp.SodpClient;

   public class SodpConfig {
       public static final SodpClient CLIENT =
           SodpClient.builder(System.getenv().getOrDefault("SODP_URL", "ws://localhost:7777"))
               .build();
   }
   ```

5. **Write a usage example** in a comment or a separate example file showing:
   - How to watch a key
   - How to set/patch state
   - How to unsubscribe

6. **Tell the user** what was created, what env var to set (`SODP_URL`), and how to start a local server:
   ```
   The SODP server can be run locally:
     Docker:  docker run -p 7777:7777 ghcr.io/orkestri/sodp-server:latest
     Binary:  ./sodp-server 0.0.0.0:7777
   ```
