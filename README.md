# 11 AI Træning

Denne øvelse bygger videre på jeres eksisterende `10_AI_code`. Vi går ud fra, at appen allerede virker (I kan vælge en chatbot og chatte med den). Her skal vi "træne" AI'en til at opføre sig mere forudsigeligt, og sætte guardrails op, så den ikke bare svarer på hvad som helst.

Start med jeres eget `10_AI_code`-projekt - der skal ikke oprettes noget nyt fra bunden.

## Træn din AI (SystemPrompt.js)

At "træne" en model i den her sammenhæng betyder ikke, at vi laver rigtig fine-tuning (det kræver et træningsdatasæt og et betalt træningsjob hos OpenAI). I stedet "træner" vi adfærden gennem en mere detaljeret system-prompt: konkrete regler for, hvordan tutoren skal reagere i forskellige situationer, og gerne et eksempel på et godt svar.

1. Opret `services/SystemPrompt.js` og skriv en system-prompt, der som minimum:
   - beskriver AI'ens rolle (hvad er den tutor i?)
   - giver mindst 2-3 konkrete regler for *hvordan* den skal reagere i forskellige situationer (fx: eleven vil have et direkte svar, eleven er kørt fast, eleven svarer rigtigt, eleven spørger om noget andet)
   - indeholder ét eksempel på en god dialog (elev-spørgsmål + et godt, guidende svar)

**Tip:** Herunder er et eksempel på, hvordan jeres fil kan være bygget op - brug det som inspiration, men skriv selv reglerne, så de passer til jeres eget emne:
```javascript
const SYSTEM_PROMPT = `You are an assistant that serves as a tutor for ???.

Your role:
- ???
- ???

How to respond in specific situations:
- If the student asks you to just give them the answer, ???
- If the student seems stuck, ???
- If the student asks something unrelated to the topic, ???

Example of a good response:
Student: "???"
You: "???"`;

export default SYSTEM_PROMPT;
```

2. I `ChatScreen.js` skal I importere den og bruge den i stedet for jeres nuværende, hardcodede prompt-tekst:
```javascript
import SYSTEM_PROMPT from "../services/SystemPrompt";
```
```javascript
  const conversationHistory = useRef([
    {
      role: "system",
      content: SYSTEM_PROMPT,
    },
    {
      role: "assistant",
      content: "Hello, I am your assistant. How can I help you?",
    },
  ]);
```

3. Test: åbn chatten og prøv at stille et spørgsmål uden for dit emne. Reagerer AI'en som du forventede, ud fra reglerne i din prompt?

## Guardrails (Guardrails.js)

En system-prompt er ikke en garanti - en bruger kan forsøge at omgå den med en smart formulering, og en prompt alene beskytter heller ikke mod stødende indhold eller mod at blive spammet med beskeder. Derfor bygger vi nu et lag af guardrails, som kører på hver besked, *før* den når selve chat-modellen.

1. Opret `services/Guardrails.js` og importer det, du skal bruge:
```javascript
import OpenAI from "openai/index.mjs";
import { OPENAI_API_KEY } from "@env";

const openai = new OpenAI({ apiKey: OPENAI_API_KEY });

export const MAX_MESSAGE_LENGTH = 500;
const MIN_MS_BETWEEN_MESSAGES = 3000;

let lastSentAt = 0;
```

2. **Guardrail 1 - beskedlængde.** Afvis beskeder, der er længere end `MAX_MESSAGE_LENGTH`. Hver guardrail-funktion skal returnere `{ allowed: true }` eller `{ allowed: false, reason: "besked til brugeren" }`.
```javascript
export function checkLength(text) {
  if (text.length > ???) {
    return {
      allowed: ???,
      reason: `Den besked er for lang (maks. ${MAX_MESSAGE_LENGTH} tegn). Prøv at forkorte den.`,
    };
  }
  return { allowed: ??? };
}
```

3. **Guardrail 2 - rate limiting.** Forhindrer at man sender beskeder for hurtigt efter hinanden. Dette er kun en klient-side beskyttelse - en rigtig produktionsapp skal rate-limite på en server.
```javascript
export function checkRateLimit() {
  const now = Date.now();
  if (now - lastSentAt < MIN_MS_BETWEEN_MESSAGES) {
    return {
      allowed: false,
      reason: "Vent lidt – du sender beskeder meget hurtigt efter hinanden.",
    };
  }
  ??? = now;
  return { allowed: true };
}
```

4. **Guardrail 3 - indholdsmoderation.** Brug OpenAIs moderation-endpoint til at fange stødende/skadeligt indhold. Moderation er gratis at kalde.
```javascript
export async function checkModeration(text) {
  try {
    const result = await openai.moderations.create({
      model: "omni-moderation-latest",
      input: text,
    });
    const flagged = result.results?.[0]?.flagged ?? false;
    if (???) {
      return {
        allowed: false,
        reason: "Den besked kan jeg ikke hjælpe med. Prøv at formulere dig anderledes.",
      };
    }
    return { allowed: true };
  } catch (error) {
    console.error("Fejl ved moderation:", error);
    // Fail open: en teknisk fejl i selve tjekket skal ikke blokere brugeren.
    return { allowed: true };
  }
}
```

5. **Guardrail 4 - emnebegrænsning.** Lav et selvstændigt, billigt klassificeringskald på en mindre model (fx `gpt-5-mini`), der afgør om beskeden hører til dit emne, uafhængigt af hvad system-prompten selv siger.
```javascript
export async function checkOnTopic(text) {
  try {
    const response = await openai.chat.completions.create({
      model: "gpt-5-mini",
      messages: [
        {
          role: "system",
          content:
            'Classify whether the following message is about ??? . Respond with only one word: "yes" or "no".',
        },
        { role: "user", content: text },
      ],
      max_tokens: 1,
    });
    const answer = response.choices[0]?.message?.content?.trim().toLowerCase();
    if (answer !== "yes") {
      return {
        allowed: false,
        reason: "???",
      };
    }
    return { allowed: true };
  } catch (error) {
    console.error("Fejl ved emne-tjek:", error);
    return { allowed: true };
  }
}
```

**Tænk over:** Denne klassifikation ser kun på den enkelte besked, ikke resten af samtalen. Kan du komme i tanke om en besked, der giver mening som opfølgning midt i en samtale, men som ville blive forkert vurderet som "uden for emnet", hvis den stod helt alene?

6. **Kør guardrails i rækkefølge.** Sæt dem billigst/hurtigst først, og stop ved den første, der blokerer - så betaler vi ikke for API-kald, vi ikke behøver.
```javascript
export async function runGuardrails(text) {
  const checks = [???, ???, ???, ???];
  for (const check of checks) {
    const result = await check(text);
    if (!result.allowed) {
      return result;
    }
  }
  return { allowed: true };
}
```

## Kobl guardrails ind i ChatScreen.js

Lige nu kalder jeres `onSend` direkte `getBardResp`, som sender beskeden videre til AI-modellen uden om noget filter. Det skal vi ændre, så beskeden først går igennem `runGuardrails`.

1. Importer `runGuardrails`:
```javascript
import { runGuardrails } from "../services/Guardrails";
```

2. Lav en hjælpefunktion, der viser en bot-besked i chatten. Den skal bruges både til rigtige AI-svar og til guardrail-afvisninger, så I ikke skal skrive den samme kode to gange:
```javascript
  const respondWithBotMessage = (text) => {
    const chatAIResp = {
      _id: Math.random() * (9999999 - 1),
      text,
      createdAt: Date.now(),
      user: {
        _id: 2,
        name: "React Native",
        avatar: CHAT_BOT_FACE,
      },
    };
    setMessages((previousMessages) =>
      GiftedChat.append(previousMessages, chatAIResp)
    );
  };
```

3. Lav funktionen `handleIncomingMessage`, der kører guardrails, og kun kalder `getBardResp` hvis beskeden bliver godkendt:
```javascript
  const handleIncomingMessage = async (msg) => {
    setLoading(true);
    const guardrailResult = await ???(msg);
    if (!guardrailResult.???) {
      setLoading(false);
      respondWithBotMessage(guardrailResult.???);
      return;
    }
    getBardResp(msg);
  };
```

4. Ret `onSend`, så den kalder `handleIncomingMessage` i stedet for `getBardResp` direkte:
```javascript
  const onSend = useCallback((messages = []) => {
    setMessages((previousMessages) =>
      GiftedChat.append(previousMessages, messages)
    );
    if (messages[0].text) {
      handleIncomingMessage(messages[0].text);
    }
  }, []);
```

5. Til slut kan I bruge jeres nye `respondWithBotMessage` i `getBardResp` i stedet for at bygge bot-beskeden manuelt igen - det er den samme kode, I allerede skrev i trin 2.

## Test guardrails

Test alle fire guardrails hver for sig:
- Send en meget lang besked (over 500 tegn)
- Send to beskeder hurtigt efter hinanden
- Send noget stødende
- Send noget, der tydeligvis ikke har noget med dit emne at gøre

Svarer appen med en passende afvisning hver gang, uden at ramme selve tutor-modellen (og dermed betale for et kald, I ikke behøvede)?

# Fil Checkliste
- [ ] `SystemPrompt.js`
- [ ] `Guardrails.js`
- [ ] `ChatScreen.js` opdateret (import, respondWithBotMessage, handleIncomingMessage, onSend)

Fedt, nu er din AI trænet og har guardrails på! Du kan lege med prompten, tilføje flere guardrails, eller prøve at ændre grænserne (maks. beskedlængde, tid mellem beskeder).
