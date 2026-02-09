# Architettura Personal Assistant 24/7

> Setup personalizzato per gestione di due business:
> 1. Azienda di ispezioni termografiche su impianti fotovoltaici con drone (fase di lancio)
> 2. SaaS piattaforma gestione appuntamenti AI per professionisti del beauty

---

## Profilo e vincoli

| Parametro | Valore |
|---|---|
| Email | Gmail |
| Calendario | Google Calendar |
| Task Manager | TickTick |
| Messaggistica | Telegram (testo, poi voce) |
| Piano Claude | Max 5x (€100/mese) |
| Hardware | NUC server H24 + PC Windows |
| Memoria | Qdrant + Mem0 |
| Budget extra | Da definire (ElevenLabs/Twilio per fase 2) |

---

## ATTENZIONE: Vincolo critico - Claude Max 5x

Il piano Max 5x ha limiti importanti per un setup 24/7:

| Metrica | Max 5x (€100) | Max 20x (€200, Goda) |
|---|---|---|
| Messaggi per finestra 5h | ~225 | ~900+ |
| Ore Sonnet/settimana | 140-280h | 240-480h |
| Ore Opus/settimana | 15-35h | 24-40h |
| Switch auto Opus→Sonnet | Al 20% uso | Al 50% uso |

**Impatto:** Goda usa il piano 20x e fa check-in ogni 30 minuti. Con il 5x, ogni check-in consuma quota (legge email, calendario, task = molte tool call). 48 check-in/giorno esaurirebbero il budget rapidamente.

**Soluzione: Smart Pre-filter** (dettagli sotto)

---

## Architettura complessiva

```
                    ┌──────────────┐
                    │  Tu (Phone)  │
                    │   Telegram   │
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │  Telegram    │
                    │  Bot API     │
                    └──────┬───────┘
                           │
              ┌────────────▼────────────┐
              │   NUC SERVER (Linux)    │
              │                         │
              │  ┌───────────────────┐  │
              │  │  Bun Relay        │  │
              │  │  (Grammy bot)     │  │
              │  └────────┬──────────┘  │
              │            │             │
              │  ┌────────▼──────────┐  │
              │  │  Pre-filter       │  │
              │  │  (lightweight)    │  │
              │  │  Controlla se     │  │
              │  │  serve Claude     │  │
              │  └────────┬──────────┘  │
              │            │             │
              │  ┌────────▼──────────┐  │
              │  │  Claude Code      │  │
              │  │  (headless -p)    │  │
              │  └────────┬──────────┘  │
              │            │             │
              │       ┌────▼────┐        │
              │       │  MCP    │        │
              │       │ Servers │        │
              │       └────┬────┘        │
              │            │             │
              │   ┌────────┼─────────┐   │
              │   │        │         │   │
              │   ▼        ▼         ▼   │
              │ Gmail  Calendar  TickTick│
              │                         │
              │  ┌───────────────────┐  │
              │  │  Docker           │  │
              │  │  ├─ Qdrant        │  │
              │  │  └─ Mem0/         │  │
              │  │    OpenMemory MCP │  │
              │  └───────────────────┘  │
              │                         │
              │  ┌───────────────────┐  │
              │  │  Smart Check-in   │  │
              │  │  (systemd timer)  │  │
              │  │  ogni 30 min      │  │
              │  └───────────────────┘  │
              │                         │
              └─────────────────────────┘
                           │
                    (futuro, SSH/RDP)
                           │
                    ┌──────▼───────┐
                    │  PC Windows  │
                    └──────────────┘
```

---

## Fase 1: Testo su Telegram (MVP)

### 1.1 NUC Server Setup

**OS consigliato: Ubuntu Server 24.04 LTS** (o altra distro Linux leggera)

Perche Linux e non Windows sul NUC:
- Docker nativo (Qdrant, Mem0 girano in container)
- systemd per gestire i servizi come daemon
- Meno risorse, piu stabile per un server H24
- Bun gira nativamente
- Il PC Windows resta accessibile via rete

**Software base:**
```
- Bun (runtime TypeScript)
- Docker + Docker Compose (Qdrant, OpenMemory)
- Claude Code CLI (autenticato con Max plan)
- Node.js 18+ (per alcuni MCP server)
- Python 3.10+ + uv (per altri MCP server)
```

### 1.2 Telegram Bot Relay

Il cuore del sistema. Basato sul pattern di Goda (claude-telegram-relay).

**Componenti:**
- **Grammy** - framework bot Telegram per Bun/Deno/Node
- **Relay logic** - riceve messaggio → chiama Claude Code headless → ritorna risposta
- **User ID check** - solo il tuo Telegram user ID puo interagire

**Comandi Claude Code headless:**
```bash
claude -p "[prompt con contesto]" \
  --output-format json \
  --allowedTools "mcp__gmail__*,mcp__calendar__*,mcp__ticktick__*,mcp__openmemory__*"
```

**Flusso messaggi:**
```
1. Tu mandi messaggio su Telegram
2. Grammy lo riceve
3. Il relay costruisce il prompt (messaggio + contesto da Mem0)
4. Spawna claude -p con il prompt
5. Claude usa MCP tools se necessario (email, calendario, task, memoria)
6. La risposta torna al relay
7. Il relay la manda su Telegram
```

### 1.3 Memoria: Qdrant + Mem0

**Docker Compose:**
```yaml
services:
  qdrant:
    image: qdrant/qdrant
    ports:
      - "6333:6333"
    volumes:
      - qdrant_data:/qdrant/storage
    restart: always

  # OpenMemory MCP (opzionale, se vuoi il server MCP di Mem0)
  openmemory:
    image: mem0/openmemory
    depends_on:
      - qdrant
    environment:
      - QDRANT_HOST=qdrant
      - QDRANT_PORT=6333
    restart: always

volumes:
  qdrant_data:
```

**Integrazione nel relay (TypeScript):**
```typescript
import { Memory } from 'mem0ai/oss';

const memory = new Memory({
  vectorStore: {
    provider: 'qdrant',
    config: { host: 'localhost', port: 6333, collection_name: 'assistant_memory' }
  },
  // Usa OpenAI per embeddings (migliori) oppure Ollama (gratis)
  embedder: { provider: 'openai', config: { model: 'text-embedding-3-small' } },
  llm: { provider: 'openai', config: { model: 'gpt-4.1-nano' } }
});

// Dopo ogni conversazione
await memory.add(messages, { user_id: 'owner', metadata: { source: 'telegram' } });

// Prima di ogni risposta - recupera contesto rilevante
const context = await memory.search(userMessage, { user_id: 'owner' });
```

**Tipi di memoria che il sistema traccia automaticamente:**
- **Fatti personali** - preferenze, abitudini
- **Obiettivi** - "Lanciare il sito entro marzo", "Chiudere il deal con cliente X"
- **Contesto business** - drone: clienti, preventivi, ispezioni | SaaS: feature, bug, clienti beta
- **Conversazioni** - cosa avete discusso, quando, su che argomento
- **Decisioni** - "Ho deciso di usare Stripe per i pagamenti"

### 1.4 MCP Servers

#### Gmail + Calendar (combo)

**Consigliato: [MarkusPfundstein/mcp-gsuite](https://github.com/MarkusPfundstein/mcp-gsuite)**
- Gmail + Calendar in un unico server
- Leggero, ben mantenuto (472 stars)
- Gmail: cerca email, leggi, crea bozze, rispondi, allegati
- Calendar: lista eventi, crea eventi, elimina, gestisci

```bash
claude mcp add-json gsuite '{"command":"uvx","args":["mcp-gsuite"]}'
```

**Alternativa piu completa: [taylorwilsdon/google_workspace_mcp](https://github.com/taylorwilsdon/google_workspace_mcp)**
- 1.3K stars, copre Gmail + Calendar + Drive + Docs + Sheets
- Utile se vuoi anche accesso a Google Drive per documenti

#### TickTick

**Consigliato: [krisrowe/ticktick-mcp](https://github.com/krisrowe/ticktick-mcp)**
- Setup piu semplice, nativo Claude Code
- 5 tool essenziali: list_projects, list_tasks, create_task, update_task, complete_task

```bash
claude mcp add --scope user ticktick -- ticktick-mcp --stdio
```

**Alternativa power-user: [jacepark12/ticktick-mcp](https://github.com/jacepark12/ticktick-mcp)**
- 20+ tool, filtri per priorita/scadenza, workflow GTD

#### Memoria (OpenMemory MCP)

```bash
# Se usi OpenMemory MCP di Mem0
claude mcp add-json openmemory '{"command":"docker","args":["exec","openmemory","..."]}'
```

Oppure integrazione diretta nel relay via SDK Mem0 (consigliato per piu controllo).

### 1.5 Smart Check-in con Pre-filter

Questo e il pezzo chiave per non bruciare la quota del Max 5x.

**Problema:** Ogni check-in con Claude consuma ~5-15 messaggi equivalenti (legge email, calendario, task, memoria, decide, compone messaggio). 48 check-in/giorno = potenzialmente 240-720 messaggi, quasi tutta la quota giornaliera.

**Soluzione: Pre-filter a 2 livelli**

```
┌──────────────────────────────────────────────┐
│           LIVELLO 1: Pre-filter              │
│           (NO Claude, NO AI, costo: €0)      │
│                                              │
│  Ogni 30 minuti, script leggero che:         │
│  1. Controlla Gmail API → nuove email?       │
│  2. Controlla Calendar API → eventi prossimi?│
│  3. Controlla TickTick API → task scaduti?   │
│  4. Applica regole semplici:                 │
│     - Nessuna novita → SKIP (nessun costo)   │
│     - Qualcosa di nuovo → passa a Livello 2  │
│                                              │
│  Regole di gating (no AI necessaria):        │
│  - Orari sacri: 23:00-07:00 → SKIP           │
│  - Cooldown: ultimo msg < 2h → SKIP          │
│  - Weekend: solo urgenze                     │
└──────────────────┬───────────────────────────┘
                   │ Solo se c'e qualcosa
                   ▼
┌──────────────────────────────────────────────┐
│           LIVELLO 2: Claude Valuta           │
│           (consuma quota Max 5x)             │
│                                              │
│  Claude Code riceve:                         │
│  - Lista nuove email (subject + mittente)    │
│  - Eventi prossime 4h                        │
│  - Task scaduti/urgenti                      │
│  - Memoria: ultimi check-in, obiettivi       │
│                                              │
│  Claude decide:                              │
│  - SKIP → niente di rilevante               │
│  - TEXT → manda messaggio su Telegram        │
│  - (futuro) CALL → chiama                    │
└──────────────────────────────────────────────┘
```

**Stima consumo ottimizzato:**
- 48 check di Livello 1/giorno → 0 messaggi Claude (API dirette, gratis)
- ~5-10 passaggi a Livello 2/giorno → ~50-150 messaggi Claude
- Resta ampia quota per le tue interazioni dirette via Telegram

**systemd timer (ogni 30 min):**
```ini
# /etc/systemd/system/checkin.timer
[Unit]
Description=Smart Check-in Timer

[Timer]
OnCalendar=*:00,30
Persistent=true

[Install]
WantedBy=timers.target
```

### 1.6 Accesso al PC Windows

**Opzioni dal NUC Linux al PC Windows:**

| Metodo | Pro | Contro |
|---|---|---|
| **SSH (OpenSSH su Windows)** | Nativo da Win10+, esegui comandi | Serve config iniziale |
| **Samba/SMB** | Accesso file condivisi | Solo file, non comandi |
| **RDP (xfreerdp)** | Controllo completo desktop | Pesante, non serve per automazione |
| **WinRM/PowerShell Remoting** | Comandi remoti nativi Windows | Setup piu complesso |

**Consiglio:** Abilita OpenSSH Server su Windows e usa SSH dal NUC. Cosi Claude Code puo eseguire comandi sul PC Windows quando serve.

```bash
# Dal NUC, il relay puo fare:
ssh user@windows-pc "powershell -Command 'Get-Process'"
```

---

## Fase 2: Voce e Chiamate (futuro)

Quando vorrai aggiungere voce:

| Componente | Servizio | Costo stimato |
|---|---|---|
| Text-to-Speech | ElevenLabs | ~€5-20/mese |
| Speech-to-Text | Gemini API / Whisper | Gratis-€5/mese |
| Telefonate | Twilio + ElevenLabs | ~€5-15/mese |

**Flusso voce su Telegram:**
```
Tu mandi vocale → Gemini/Whisper trascrive → Claude elabora → ElevenLabs genera audio → risposta vocale
```

**Flusso chiamate:**
```
Tu dici "chiamami" su Telegram → Twilio chiama il tuo numero → ElevenLabs voice agent → conversazione con contesto da Mem0
```

---

## Fase 3: Multi-agente (futuro)

Visione a lungo termine, come suggerisce Goda:

| Agente | Ruolo | Chat Telegram |
|---|---|---|
| **Assistente Principale** | Coordinatore, memoria, check-in | Chat principale |
| **Agente Drone Business** | Clienti, preventivi, ispezioni, reportistica | Chat dedicata |
| **Agente SaaS** | Sviluppo, bug, clienti beta, metriche | Chat dedicata |
| **Critico/Reviewer** | Revisiona decisioni, fa da avvocato del diavolo | Chat dedicata |

Ogni agente con il proprio contesto e memoria separata in Mem0 (`agent_id` diverso).

---

## Costi stimati

### Fase 1 (MVP - solo testo)

| Voce | Costo/mese |
|---|---|
| Claude Max 5x | €100 |
| Qdrant (self-hosted Docker) | €0 |
| Mem0 OSS (self-hosted) | €0 |
| OpenAI embeddings (text-embedding-3-small) | ~€1-3 |
| NUC (elettricita) | ~€5-10 |
| **Totale** | **~€106-113/mese** |

Oppure €100/mese esatti se usi Ollama per embeddings (qualita leggermente inferiore).

### Fase 2 (voce)

| Voce | Costo/mese |
|---|---|
| Fase 1 | ~€110 |
| ElevenLabs | ~€5-20 |
| Twilio | ~€5-15 |
| **Totale** | **~€120-145/mese** |

---

## Confronto con Goda Go

| Aspetto | Setup Goda | Il tuo setup |
|---|---|---|
| Piano Claude | Max 20x ($200) | Max 5x (€100) |
| Check-in | Ogni 30 min, tutti via Claude | Smart pre-filter + Claude solo se serve |
| Memoria | Supabase + pgvector (manuale) | Mem0 + Qdrant (automatica) |
| Embeddings | OpenAI text-embedding-3-large | OpenAI small o Ollama |
| Voce | Si (ElevenLabs + Twilio) | Fase 2 |
| Hosting | MacBook sempre acceso | NUC dedicato |
| MCP Memory | Nessuno | OpenMemory MCP |
| Costo | ~$250/mese | ~€110/mese (fase 1) |

---

## Prossimi passi per iniziare

1. **Setup NUC** - Installa Ubuntu Server, Docker, Bun, Claude Code CLI
2. **Docker Compose** - Avvia Qdrant
3. **MCP Servers** - Configura Gmail+Calendar (mcp-gsuite) e TickTick
4. **Telegram Bot** - Crea bot via @BotFather, implementa relay con Grammy
5. **Mem0 integration** - Integra nel relay per memoria persistente
6. **Pre-filter** - Script leggero per check-in intelligenti
7. **systemd services** - Relay come daemon + timer per check-in
8. **Test e fine-tuning** - Regola gating rules, personalizza prompt di sistema
