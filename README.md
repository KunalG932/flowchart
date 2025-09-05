# Auto-Manga System – Simple Overview  

## Parts
- **Bot** – Handles user commands, DB  
- **UserBot** – Creates channels, uploads  
- **MongoDB** – Stores manga + users  
- **Scrapers** – Get chapters + images  
- **Queue** – Holds upload tasks  
- **Auto Checker** – Finds new chapters  
- **Worker** – Downloads → Converts → Uploads  

---

## Data Example (`manga_channels`)
```json
{
  "manga_id": "abc123",
  "title": "One Piece",
  "channel_id": -10012345678,
  "invite_link": "t.me/+something",
  "last_chapter": 1120,
  "status": "ongoing",
  "url": "https://...",
  "sf": "ck",
  "owner_id": 12345678
}
````

---

## System Overview

```mermaid
graph LR
  U[User] --> B(Bot)
  B --> M[(MongoDB)]
  B --> UB(UserBot)
  subgraph Auto
    AC[Auto Checker]
    Q[Queue]
    W[Worker]
  end
  AC --> S[Scrapers]
  AC --> Q
  Q --> W
  W --> UB
  UB --> CH[[Private Channels]]
  B --> UPD[[Update Channel]]
```

---

## Flow 1: Enable Auto-Upload

```mermaid
sequenceDiagram
  participant U as User
  participant B as Bot
  participant DB as MongoDB
  participant UB as UserBot

  U->>B: Click "Auto-upload"
  B->>DB: Check manga
  alt No channel
    B->>UB: Create channel
    UB-->>B: Channel + invite link
    B->>DB: Save channel
  else Already exists
    B->>DB: Add subscriber
  end
  B-->>U: Send invite link
```

---

## Flow 2: Auto Checker

```mermaid
flowchart TD
  A[Every few minutes] --> B[Get ongoing manga]
  B --> C[Fetch chapters from scraper]
  C --> D[Find new chapters]
  D --> E[Add tasks to Queue]
```

---

## Flow 3: Upload Worker

```mermaid
flowchart TD
  Q[Get task] --> A[Load settings]
  A --> B[Download images]
  B --> C[Add banners + thumbnail]
  C --> D[Make PDF]
  D --> E[Upload via UserBot]
  E --> F[Update DB + release task]
```

---

## Notes

* Each manga has a stable `manga_id` (hash of URL).
* Queue is locked per channel (no duplicate uploads).
* Admin commands:

  * `/force_check <id>` → scan now
  * `/mark_complete <id>` → mark finished
---
