# Survive RPG — notas para Claude Code

Juego de supervivencia sandbox 2D (vista lateral), un solo archivo: `index.html` (antes `sobrevivir.html`) (canvas HTML5, sin dependencias).
Se juega sobre todo en iPhone en horizontal; también en iPad y Mac. El autor es Santiago y habla español: responde en español.
Quiere código completo y listo para usar, y opiniones honestas (señala fallos, no le des la razón por darla).

## Cómo probar
- Abre `index.html` en el navegador o sírvelo con `npx serve .` / `python3 -m http.server`.
- Guardado local: localStorage con clave `sobrevivir2d_v4` (no cambiarla: rompería las partidas guardadas) (copias de respaldo en `_prev` y `_broken`).
- Guardado en la nube: solo funciona cuando corre como artifact de claude.ai (`claude.use('db')`/`claude.use('user')`). Fuera de ahí, el código debe seguir funcionando sin nube.

## Arquitectura (resumen)
- Constantes: `TS=16, H=140, SEA=58, DAY=480`. Escala de píxel entera: `S=round(short*dpr/(TS*14))`, y `imageSmoothing` desactivado.
- Mundo infinito por chunks: `CW=32`; claves de mapa `K(x,y)=(x+OFF)*256+y` con `OFF=1<<20`. Generación determinista por semilla (`genChunk`, `noise2`, `hash`).
  - Biomas: desierto, pradera, bosque, montaña y nieve.
  - Solo se guardan los chunks modificados (`c.mod`, codificados en base64). `unloadFar` descarga los chunks lejanos que no se han modificado.
- Bloques en `B` (0–32): menas de cobre (26), oro (27) y diamante (28). Ítems 100–145 y monedas 200. Niveles de herramienta en `TIER_OF`, durabilidad `DUR=[60,130,200,320,900]`.
- IA: `planPath` (BFS); los zombis rompen paredes y puertas desde el día 2 (`hitBlock`, `BHP`); las arañas trepan.
  - Límite de hostiles: 3 normalmente; en hordas 4 el día 1 y 6 después.
- Progresión: XP, niveles y habilidades (`prog`), nivel de mobs (`mobLvl`) con barras de vida, durabilidad (`wear`), reparación en el mercader.
- Modo historia: Anselmo (`guide`, `initGuide`, `drawGuide`) espera cerca del spawn. Las misiones (`QUESTS`) se aceptan y se entregan hablando con él (`questState`, `acceptQuest`, `turnInQuest`, `renderGuide`); `flags.qa` = misión aceptada, `qi` = índice. Recompensa: monedas, XP y objetos. `#quest` solo es un rastreador de la misión activa. Misión 1: mesa+espada+pico; misión 2: 3 ovejas (`sheep`) y 3 de lana (id 146). Los guardados antiguos se migran en `loadFrom` (`flags.story`).
- Expansiones y razas: `EXPANSIONS` (por ahora solo «Raíces del Mundo», niveles 1–20, tope `LVL_CAP`). Razas `RACES` = normal y druida (`flags.race`, se elige en `raceBegin`, permanente). `ABILS` define las habilidades por nivel: pasivas (`m`, se suman con `rmod(k)`) y activas (`act`, botón `#tx` o tecla Q → `useAbility`, equipada en `flags.ab`).
- Monstruos (`MT`, `hostile`): zombi, araña, lobo (nieve/bosque), escorpión (desierto, veneno), esqueleto (montaña/cuevas, flechas en `arrows`), limo (salta) y zombi bruto. Reglas de aparición por bioma y nivel en `spawner`; drops en `mobDie`. Ítems nuevos 147–149.
- Refugio estilo "Asylum": NPC con roles, despensa, economía diaria (`newDay`), mejoras `UPG` y 15 misiones (`QUESTS`) con recompensas.
- Controles táctiles:
  - Un toque mina o usa.
  - Un toque corto (<0.3 s) interactúa con puerta, cama, cofre, mesa, fogata u horno.
  - El botón de acción `#ta` solo aparece con una herramienta o espada en la mano.
- Teclado en Mac: `grabFocus()` y `#focusHint`.

## Siguiente paso planeado: arte
El arte dibujado por código se ve de baja resolución. El plan es pasar a sprites reales:
- Opción 1: un pack de assets, por ejemplo Sunny Land (CC0) de ansimuz en itch.io.
- Opción 2: generarlos con el MCP de PixelLab (`npx mcp-pixellab --secret=...`).

Requisitos del arte nuevo:
- Héroe animado, tiles con bordes, enemigos y fondo con parallax.
- Hacer primero un prototipo aparte y después integrarlo en `index.html`.

## Ideas ofrecidas, aún sin confirmar
- Mobs por bioma: lobos en la nieve, escorpiones en el desierto, esqueletos en la montaña.
- Daño por contacto con cactus.
- Animales en la nieve y el desierto.

## Repo
- GitHub: `Danielca90/survive-rpg` (antes `tallgrass`). El proyecto Java anterior está en la rama `legacy-tallgrass`.
