# The Stalker — mod de terror (Fabric, Minecraft 1.21.11)

Uma criatura que te observa de longe, se esconde quando você olha e, se você chega perto demais
(ou a encara por tempo demais), vira uma perseguição: **título "RUN!!!"**, **glitch na tela**,
sons perturbadores e um ataque de salto/agarrar.

## Estrutura

```
the-stalker/
├─ build.gradle / gradle.properties / settings.gradle
└─ src/
   ├─ main/java/com/stalker/
   │  ├─ TheStalkerMod.java        entrypoint comum
   │  ├─ ModEntities.java          registro do EntityType
   │  ├─ ModSounds.java            registro dos sons
   │  ├─ StalkerSpawner.java       spawn só em locais escuros / à noite
   │  ├─ StalkerUtil.java          chão válido, linha de visão, escuridão
   │  └─ entity/StalkerEntity.java entidade + IA (STALKING / CHASING / ataque)
   ├─ main/resources/
   │  ├─ fabric.mod.json
   │  └─ assets/stalker/ (lang, sounds.json, sounds/*.ogg, textures/entity/the_stalker.png, icon.png)
   └─ client/
      ├─ java/com/stalker/client/
      │  ├─ TheStalkerClient.java    entrypoint do cliente
      │  ├─ StalkerRenderer.java     renderer (modelo humanoide esticado + olhos brilhantes)
      │  ├─ StalkerClientState.java  intensidade 0..1 dos efeitos + funções de tremor
      │  └─ StalkerHud.java          tremor da HUD + overlay de glitch
      

```

## Como funciona

| Fase | O que acontece |
|---|---|
| **Spawn** | A cada 10 s, 30% de chance por jogador (Overworld, não-creative): aparece a 38–55 blocos, num ponto com luz efetiva ≤ 7 (noite, caverna, local escuro). Só um por jogador. Some se o ambiente ficar claro por 20 s ou após 10 min. |
| **Stalking** | Fica parada olhando para você. Se você a encara, ela "dá passos" (teleporta ~4 blocos mais perto, escondida atrás de árvore/pedra quando possível) e, após 5 s de encarada, se esconde. Quando você olha para outro lado ela muda de posição na sua periferia (nunca "pisca" dentro do seu campo de visão). Respiração, sussurros e batimento cardíaco crescem com a proximidade. |
| **Chasing** | Dispara se você chegar a < 12 blocos ou encarar por ~20 s acumulados. Envia o título **RUN!!!** (vermelho, negrito) + subtítulo, corre muito rápido, pisca para 7–14 blocos de você, salta em cima, faz sons. Desiste após 60 s ou se você fugir a 64+ blocos. |
| **Ataque** | Ao encostar: 10 de dano (5 corações), lentidão V + cegueira + náusea (atordoado), depois some para ~50 blocos e fica 30–60 s sem perseguir. |
| **Fim dos efeitos** | O estado "chasing" é sincronizado na entidade. Ao terminar a perseguição, a criatura sumir ou você morrer, o cliente zera glitch + tremor e o servidor limpa o título. |

Efeitos no cliente (`StalkerClientState` → `StalkerHud` / `CameraMixin`):
- **Sem tremor de tela**: removido a pedido; ficam só o título RUN!!! e o glitch.
- **Glitch**: vinheta vermelha pulsante, scanlines, barras horizontais com separação RGB, blocos de pixels corrompidos, ruído e flashes.

## Compilar

Requisitos: **JDK 21** e Gradle 9.x (ou o wrapper do template oficial).

1. **Confira as versões** em `gradle.properties` com https://fabricmc.net/develop (Minecraft 1.21.11):
   `loader_version`, `loom_version`, `fabric_version`. Os valores que deixei são estimativas.
2. Na pasta do projeto: `gradle wrapper` (uma vez) e depois `./gradlew build`.
3. O mod sai em `build/libs/the-stalker-1.0.0.jar` (use o que **não** termina em `-sources`).
4. Coloque o jar em `.minecraft/mods/` junto com o **Fabric API** 1.21.11.

Para desenvolver/testar direto: `./gradlew runClient`.

## Testar

Em um mundo com cheats:
```
/time set midnight
/summon stalker:the_stalker ~30 ~ ~
```
- Fique parado olhando para ela: ela vai chegando mais perto e se escondendo.
- Aproxime-se a menos de 12 blocos (ou encare por ~20 s) para ver RUN!!!, glitch e tremor.
- Em pleno dia ela some em ~20 s (por design: só vive no escuro). Para testar de dia, entre numa caverna escura.
- Para zerar: `/kill @e[type=stalker:the_stalker]`.
- Teste também: morrer durante a perseguição (efeitos devem sumir ao aparecer a tela de morte).

## Sons e texturas

Incluí **placeholders gerados por código** para o mod já funcionar. Troque pelos seus, mantendo os nomes.

Sons — `src/main/resources/assets/stalker/sounds/` (**.ogg, mono**, para o som ser posicional):

| Arquivo | Evento | Quando toca | Sugestão |
|---|---|---|---|
| `breath.ogg` | `stalker_breath` | Stalking de perto e durante a corrida | respiração pesada e rouca, ~2–3 s |
| `step.ogg` | `stalker_step` | Cada passo na corrida | passo grave e úmido, curto |
| `whisper.ogg` | `stalker_whisper` | Stalking, piscadas na perseguição | sussurros ininteligíveis, ~2 s |
| `scream.ogg` | `stalker_scream` | Início, saltos, ataque | grito distorcido / bitcrushed, ~1–2 s |
| `heartbeat.ogg` | `stalker_heartbeat` | Ouvido só em cima do jogador, acelera ao se aproximar | duas batidas graves, 1 s |
| `chase_start.ogg` | `stalker_chase_start` | Instante em que o RUN!!! aparece | sting/drone grave crescente |

Textura — `assets/stalker/textures/entity/the_stalker.png` (**64×64**, layout de skin do jogador/zumbi):
corpo quase preto, olhos brancos/vermelhos em `u 8–15, v 8–15` (frente da cabeça). Para um modelo próprio
(Blockbench), troque o `HumanoidModel` do `StalkerRenderer` por uma classe `EntityModel` sua.

## Ajustes rápidos (constantes no topo de `StalkerEntity`)

`CHASE_TRIGGER_DIST` (12), `LOOK_LIMIT_TICKS` (400), `ATTACK_DAMAGE` (10), `CHASE_SPEED` (1.35),
`MAX_CHASE_TICKS` (1200). Spawn: `StalkerSpawner.CHANCE`, `INTERVAL`, `MIN_DIST/MAX_DIST`.
Intensidade dos efeitos: `amp` em `cameraShake()` / `hudShake()` (`StalkerClientState`).

## Pontos de atenção (código escrito sem compilar)

O ambiente em que escrevi bloqueia o Maven do Fabric, então **não consegui compilar**. A API do 1.21.11 mudou
bastante (renderização, HUD, `ResourceLocation` → `Identifier`). Se o build reclamar, os suspeitos mais prováveis são:

1. `StalkerRenderer`: assinaturas de `scale(...)` e `getBlockLightLevel(...)`; construtor de `HumanoidMobRenderer`.
2. `StalkerHud`: nomes em `VanillaHudElements` (`MISC_OVERLAYS`, `CHAT`) e métodos de `graphics.pose()` (`pushMatrix/translate/popMatrix`).
4. `StalkerEntity`: `hurtServer`, `shouldBeSaved` (se não existir `@Override`, remova o método) e os pacotes `Clientbound*Packet`.
5. Se preferir mappings Yarn, troque `loom.officialMojangMappings()` por `"net.fabricmc:yarn:..."` e renomeie as classes.

Cada ponto é isolado em um arquivo pequeno — mande o erro do compilador que eu corrijo.

## Upgrade opcional: distorção real de pixels

O glitch atual é um overlay 2D (barras, separação RGB, ruído). Para deslocar de fato os pixels do mundo
(ondulação, aberração cromática real), adicione um post-process shader (`PostChain`/`post_effect` em
`assets/stalker/post_effect/`) ligado enquanto `StalkerClientState.active()` for verdadeiro.
