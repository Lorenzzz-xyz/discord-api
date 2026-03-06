# 📦 Discord Webhook API

Libreria per l'invio di messaggi Discord via Webhook, compatibile con **Paper**, **Spigot**, **BungeeCord** e **Velocity**.

---

## 📋 Requisiti

- Java **21+**
- Gradle **8.7+**

---

## 🔧 Installazione

### 1. Aggiungi le credenziali fornite da lorenzzz nel tuo `gradle.properties`


```properties
# ~/.gradle/gradle.properties  (globale, consigliato)
# oppure: tuoprogetto/gradle.properties

lorenzRepoUser=IL_TUO_USERNAME
lorenzRepoPass=TOKEN
```

---

### 2. Aggiungi il repository nel tuo `build.gradle.kts`

```kotlin
repositories {
    maven {
        name = "LorenzRepo"
        url = uri("https://repo.lorenzzzz.xyz/private")
        credentials {
            username = project.findProperty("lorenzRepoUser") as String? ?: System.getenv("REPO_USER")
            password = project.findProperty("lorenzRepoPass") as String? ?: System.getenv("REPO_PASS")
        }
    }
}
```

---

### 3. Aggiungi la dipendenza

Scegli **solo** il modulo per la tua piattaforma:

```kotlin
dependencies {
    // Paper
    implementation("dev.lorenzz.libs:discord-api-paper:1.0.0-SNAPSHOT")

    // Spigot
    implementation("dev.lorenzz.libs:discord-api-spigot:1.0.0-SNAPSHOT")

    // BungeeCord
    implementation("dev.lorenzz.libs:discord-api-bungeecord:1.0.0-SNAPSHOT")

    // Velocity
    implementation("dev.lorenzz.libs:discord-api-velocity:1.0.0-SNAPSHOT")
}
```

### 4. Shadowing (obbligatorio)

Aggiungi la rilocazione nel tuo `shadowJar` per evitare conflitti con altri plugin:

```kotlin
tasks.shadowJar {
    relocate("dev.lorenzz.libs.discord", "tuoplugin.shaded.discord")
}
```

---

## 🚀 Utilizzo

### Impostare il webhook di default (consigliato in `onEnable`)

```java
Discord.setWebhook("https://discord.com/api/webhooks/ID/TOKEN");
```

---

### Messaggi semplici

```java
Discord.webhook("https://discord.com/api/webhooks/ID/TOKEN")
    .sendSimpleMessage("Server avviato!");
```

---

### Embed colorati pronti

```java
// ✅ Verde – successo
Discord.webhook(WEBHOOK).sendSuccess("Titolo", "Descrizione");

// ❌ Rosso – errore
Discord.webhook(WEBHOOK).sendError("Titolo", "Descrizione");

// ⚠️ Giallo – avviso
Discord.webhook(WEBHOOK).sendWarning("Titolo", "Descrizione");

// ℹ️ Blu – info
Discord.webhook(WEBHOOK).sendInfo("Titolo", "Descrizione");
```

---

### Embed personalizzato

```java
import dev.lorenzz.libs.discord.Discord;
import dev.lorenzz.libs.discord.builders.Embed;
import dev.lorenzz.libs.discord.builders.embeds.Author;
import dev.lorenzz.libs.discord.builders.embeds.Footer;
import dev.lorenzz.libs.discord.builders.embeds.Thumbnail;

Embed embed = Embed.builder()
    .title("🔨 Ban Log")
    .description("Un giocatore è stato bannato dal server")
    .color(0xFF0000)
    .field("Giocatore", "Steve", true)
    .field("Motivo", "Hacking", true)
    .field("Bannato da", "Admin", true)
    .thumbnail(Thumbnail.of("https://crafatar.com/avatars/UUID"))
    .author(Author.of("BanSystem", "https://url-icona.png"))
    .footer(Footer.of("Server Network", "https://url-logo.png"))
    .timestampNow()
    .build();

Discord.webhook(WEBHOOK).sendEmbed(embed);
```

---

### Payload completo (username e avatar personalizzati)

```java
import dev.lorenzz.libs.discord.builders.WebhookPayload;

WebhookPayload payload = WebhookPayload.builder()
    .username("BanBot")
    .avatarUrl("https://url-avatar.png")
    .embed(embed)
    .noMentions()
    .build();

Discord.webhook(WEBHOOK).sendMessage(payload);
```

---

### Invio asincrono (non blocca il thread principale)

```java
Discord.webhook(WEBHOOK)
    .sendSuccess("Titolo", "Testo")
    .thenAccept(response -> {
        if (response.statusCode() == 204) {
            // inviato con successo
        }
    })
    .exceptionally(ex -> {
        ex.printStackTrace();
        return null;
    });
```

---

### Invio sincrono

```java
try {
    var response = Discord.webhook(WEBHOOK)
        .sendMessageSync(payload);
    System.out.println("Status: " + response.statusCode());
} catch (Exception e) {
    e.printStackTrace();
}
```

---

## 💡 Esempio completo — Plugin Paper

```java
public class MyPlugin extends JavaPlugin {

    private static final String WEBHOOK = "https://discord.com/api/webhooks/ID/TOKEN";

    @Override
    public void onEnable() {
        Discord.setWebhook(WEBHOOK);
        Discord.webhook(WEBHOOK)
            .sendSuccess("🟢 Server Online", "Il server è avviato!");
    }

    @Override
    public void onDisable() {
        Discord.webhook(WEBHOOK)
            .sendError("🔴 Server Offline", "Il server si è spento.");
    }
}
```

```java
@EventHandler
public void onPlayerJoin(PlayerJoinEvent event) {
    Discord.webhook(WEBHOOK).sendEmbed(
        Embed.builder()
            .title("👋 Giocatore connesso")
            .description(event.getPlayer().getName() + " è entrato nel server")
            .info()
            .timestampNow()
            .build()
    );
}

@EventHandler
public void onPlayerQuit(PlayerQuitEvent event) {
    Discord.webhook(WEBHOOK).sendEmbed(
        Embed.builder()
            .title("👋 Giocatore disconnesso")
            .description(event.getPlayer().getName() + " ha lasciato il server")
            .warning()
            .timestampNow()
            .build()
    );
}
```

---

## 📦 Pubblicare la libreria

```sh
# Imposta le credenziali in gradle.properties (vedi sopra)
# Poi pubblica:
./gradlew publish
```

Pubblica automaticamente su:
- `https://repo.lorenzzzz.xyz/private` → versioni SNAPSHOT
- `https://repo.lorenzzzz.xyz/snapshots` → versioni release

---

## 🛠 Build locale

```sh
./gradlew build
# I jar si trovano in:
# discord-api-paper/build/libs/
# discord-api-spigot/build/libs/
# discord-api-bungeecord/build/libs/
# discord-api-velocity/build/libs/
```

---

## 📁 Struttura del progetto

```
discord-api/
├── discord-api-common/        ← Logica core (Discord.java, builders, embeds)
├── discord-api-paper/         ← Modulo Paper 1.21.4
├── discord-api-spigot/        ← Modulo Spigot 1.21.4
├── discord-api-bungeecord/    ← Modulo BungeeCord 1.21
└── discord-api-velocity/      ← Modulo Velocity 3.4.0
```

---

## ⚙️ Metodi disponibili su `WebhookClient`

| Metodo | Descrizione |
|--------|-------------|
| `sendSimpleMessage(String)` | Invia un messaggio di testo semplice |
| `sendMessage(WebhookPayload)` | Invia un payload completo |
| `sendMessageSync(WebhookPayload)` | Invia in modo sincrono (blocca) |
| `sendEmbed(Embed)` | Invia un singolo embed |
| `sendEmbeds(Embed...)` | Invia più embed |
| `sendSuccess(String, String)` | Embed verde con titolo e descrizione |
| `sendError(String, String)` | Embed rosso con titolo e descrizione |
| `sendWarning(String, String)` | Embed giallo con titolo e descrizione |
| `sendInfo(String, String)` | Embed blu con titolo e descrizione |

---

## ⚙️ Metodi disponibili su `Embed.builder()`

| Metodo | Descrizione |
|--------|-------------|
| `.title(String)` | Titolo dell'embed |
| `.description(String)` | Descrizione dell'embed |
| `.color(int)` | Colore RGB es. `0xFF0000` |
| `.color(Color)` | Colore da `java.awt.Color` |
| `.field(String, String, boolean)` | Aggiunge un campo (inline o no) |
| `.footer(Footer)` | Footer con testo e icona |
| `.author(Author)` | Autore con nome e icona |
| `.thumbnail(Thumbnail)` | Immagine piccola in alto a destra |
| `.image(Image)` | Immagine grande nell'embed |
| `.timestampNow()` | Timestamp corrente |
| `.timestamp(Instant)` | Timestamp personalizzato |
| `.success()` | Colore verde `0x00FF00` |
| `.error()` | Colore rosso `0xFF0000` |
| `.warning()` | Colore giallo `0xFFFF00` |
| `.info()` | Colore blu `0x0099FF` |
