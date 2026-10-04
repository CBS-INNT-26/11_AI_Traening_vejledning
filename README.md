# 11 AI Træning

Denne øvelse bygger videre på 10_AI_code: samme chatbot, men nu skal vi "træne" AI'en til at opføre sig mere forudsigeligt, og sætte nogle guardrails op, så den ikke bare svarer på hvad som helst. Få din gruppes API key af en af os. Husk ikke at spamme den med spørgsmål, da vi kun har et begrænset antal credits.

Du kan starte fra dit eget 10_AI_code-projekt, eller oprette et nyt fra bunden som beskrevet herunder.

## Opsætning

Start med at oprette et nyt react-native projekt med expo-cli.

```npx create-expo-app GenAI_Chat --template blank```

Naviger til projektmappen med cd og kør projektet.

Når projektet er startet op, kan du åbne det i en simulator eller på din telefon.
Sluk herefter serveren og erstat jeres package.json dependencies med følgende:
```javascript
"dependencies": {
    "@expo/vector-icons": "^15.0.2",
    "@react-native-async-storage/async-storage": "^2.2.0",
    "@react-navigation/stack": "^6.4.1",
    "expo": "^57.0.0",
    "expo-splash-screen": "~57.0.9",
    "expo-status-bar": "~57.0.1",
    "openai": "^4.65.0",
    "react": "19.2.3",
    "react-native": "0.86.3",
    "react-native-dotenv": "^3.4.11",
    "react-native-gesture-handler": "~2.32.0",
    "react-native-gifted-chat": "^3.4.0",
    "react-native-keyboard-controller": "1.21.9",
    "react-native-reanimated": "4.5.1",
    "react-native-safe-area-context": "~5.7.0",
    "react-native-screens": "~4.26.0",
    "react-native-worklets": "0.10.1"
  },
  "private": true,
  "devDependencies": {
    "babel-preset-expo": "~57.0.0"
  }
```

Kør herefter `npm install` i jeres projekt, så I downloader de nødvendige pakker.

**Bemærk:** `react-native-gifted-chat` skal være version 3 eller nyere - version 2 understøtter ikke `react-native-reanimated` 4, og I vil opleve en crash (`[Worklets] Cannot copy value of type 'FlatList'`) når chatten åbnes.

## Stack navigation
Vi skal bruge en stack navigation til at navigere mellem vores sider.

1. Opret mappen `components` og i den mappe opret en fil ved navn `StackNavigator.js`

Stack navigationen skal have 2 sider, en til vores chat og en til vores home screen.
2. Opret mappen `screens` med følgende filer i:
- ChatScreen.js
- HomeScreen.js

I StackNavigator.js skal vi importere de nødvendige komponenter og oprette en stack navigation med `const Stack=createStackNavigator();`

3. Importer dine screens, createStackNavigator og NavigationContainer
4. Opret stack navigationen `const Stack = createStackNavigator();`
5. Lav selve navigationen
```javascript
export default function HomeNavigation() {
  return (
    <???>
      <???.??? screenOptions={{}}>
        <???.??? name="home" component={HomeScreen}
          options={{ headerShown: false }}
        />
        <???.??? name="chat" component={ChatScreen} />
      </???.???>
    </???>
  );
}
```

### App.js
Vi skal nu tilbage til vores App.js og importere vores StackNavigator. <br>

```javascript
export default function App() {
  return <HomeNavigation />;
}
```

I skal også slette jeres styles i App.js.

**Tip**
Ja din App.js er meget kort i denne øvelse

## GlobalStyles.js
1. Opret mappen `styles` og i den filen `GlobalStyles.js`
2. Indsæt følgende
```javascript
import { StyleSheet } from "react-native";

const GlobalStyles = StyleSheet.create({
  safeArea: {
    flex: 1,
    backgroundColor: "#671ddf",
  },
  mainView: {
    flex: 1,
    backgroundColor: "#fff",
  },
  bubbleRight: {
    backgroundColor: "#671ddf",
  },
  bubbleLeftText: {
    color: "#671ddf",
    padding: 2,
  },
  bubbleRightText: {
    padding: 2,
  },
  inputToolbarContainer: {
    padding: 3,
    backgroundColor: "#671ddf",
    borderTopWidth: 1,
    borderTopColor: "#E8E8E8",
    minHeight: 48,
  },
  inputToolbarText: {
    color: "#fff",
  },
  sendButton: {
    marginRight: 10,
    marginBottom: 5,
  },
  // HomeScreen styles
  homeContainer: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center",
  },
  homeCenter: {
    alignItems: "center",
  },
  homeHello: {
    fontSize: 30,
  },
  homeName: {
    fontSize: 30,
    fontWeight: "bold",
  },
  homeImage: {
    height: 150,
    width: 150,
    marginTop: 20,
  },
  homeHelp: {
    marginTop: 30,
    fontSize: 25,
  },
  homeBotList: {
    marginTop: 20,
    backgroundColor: "#F5F5F5",
    alignItems: "center",
    height: 110,
    padding: 10,
    borderRadius: 10,
  },
  homeBotAvatar: {
    width: 40,
    height: 40,
  },
  homeBotListText: {
    marginTop: 5,
    fontSize: 17,
    color: "#B0B0B0",
  },
  homeChatButton: {
    marginTop: 40,
    padding: 17,
    width: "60%",
    borderRadius: 100,
    alignItems: "center",
  },
  homeChatButtonText: {
    fontSize: 16,
    color: "#fff",
  },
});

export default GlobalStyles;
```

## services
1. Lav en mappe kaldet `services` og lav disse filer i mappen: `ChatFaceData.js`, `Request.js`, `SystemPrompt.js` og `Guardrails.js`

### ChatFaceData.js
1. I ChatFaceData.js skal du indsætte følgende kode:
```javascript
// services/ChatFaceData.js
const chatFaceData = [
  { id: 1, name: 'Noyi', image: 'https://res.cloudinary.com/dknvsbuyy/image/upload/v1685678135/chat_1_c7eda483e3.png', primary: '#FFC107', secondary: '' },
  { id: 2, name: 'Pogu', image: 'https://res.cloudinary.com/dknvsbuyy/image/upload/v1685709886/image_21_2e18bb4a61.png', primary: '#E53057', secondary: '' },
  { id: 3, name: 'Nista', image: 'https://res.cloudinary.com/dknvsbuyy/image/upload/v1685709886/image_22_409561b953.png', primary: '#3B96D2', secondary: '' },
  { id: 4, name: 'Estor', image: 'https://res.cloudinary.com/dknvsbuyy/image/upload/v1685709886/image_18_893d24cebc.png', primary: '#37474F', secondary: '' },
  { id: 5, name: 'Pega', image: 'https://res.cloudinary.com/dknvsbuyy/image/upload/v1685709886/image_23_211d7370cb.png', primary: '#2473FE', secondary: '' },
];

export default chatFaceData;
```

### Request.js
```javascript
import OpenAI from "openai/index.mjs";

import { OPENAI_API_KEY } from "@env"; // Vi opretter denne lige om lidt
// Opret en ny instans af OpenAI-klassen
const openai = new OpenAI({ apiKey: OPENAI_API_KEY });

// En lille console.log til at kontrollere at API keyen kan læses
// console.log("OPENAI_API_KEY:", JSON.stringify(OPENAI_API_KEY));

// Funktion der sender en besked til OpenAI API'et
export default async function SendMessage(messageArray) {
  const response = await openai.chat.completions.create({
    model: "gpt-5", // Her vælger du hvilken model du ønsker at bruge
    messages: messageArray,
  });

  // Udtræk AI'ens svar fra svaret
  const result = response.choices[0]?.message?.content || "";
  // Returnér AI'ens svar
  return { role: "assistant", content: result };
}
```

**Tip:** Lad `console.log`-linjen stå udkommenteret, indtil du rent faktisk har brug for at fejlsøge din API-nøgle. Husk at kommentere den ud igen bagefter - ellers ligger din nøgle synligt i din terminal-log.

#### .env
1. Da din API er hemmelig, ønsker vi ikke, at den skal stå hard coded i koden. Derfor skal du oprette filen `.env` og tilføje `.env` til din `.gitignore`
2. I `.env` skriver du følgende: `OPENAI_API_KEY=`hvor du så sætter din API kode ind

**Faldgrube:** Skriv ikke citationstegn eller mellemrum omkring selve nøglen, og dobbelttjek at den starter med småt `sk-` (ikke `Sk-`) - nøgler er versalfølsomme. Hvis du ændrer `.env` efter du allerede har startet appen, skal du genstarte serveren helt (`npx expo start -c`) - en almindelig reload er ikke nok, da værdien bliver "bagt ind" i koden, når appen bygges.

#### babel.config.js
1. Lav en babel.config.js og indsæt dette:
```javascript
module.exports = {
  presets: ["babel-preset-expo"],
  plugins: [["module:react-native-dotenv"], "react-native-worklets/plugin"],
};
```

Forklaring på koden:
Babel er et værktøj, der oversætter moderne JavaScript-kode (ES6+, JSX osv.) til ældre JavaScript, så den kan køre på alle enheder og browsere.
- babel-preset-expo: kommer fra Expo, og indeholder alt det nødvendige for at køre React Native med moderne JavaScript
- module:react-native-dotenv: er det plugin, der lader dig bruge: `import { OPENAI_API_KEY } from "@env";`
- react-native-worklets/plugin: `react-native-gifted-chat` bruger Reanimated 4 til sine animationer, og Reanimated 4's "worklets" (kode der kører på UI-tråden) kræver dette plugin for at kunne oversættes. Uden det crasher appen, når chatten åbnes.

**Tip**
Hvis det ikke fungerer, kan du prøve at installere følgende i terminalen
```javascript
npm install babel-preset-expo --save-dev
npm install react-native-dotenv
```

## HomeScreen.js
Før vi kan arbejde med vores chatbot, skal vi have lavet en home screen, hvor vi kan vælge at starte en chat.

1. Importere de nødvendige komponenter og styling:
```javascript
import { useEffect, useState } from "react";
import { View, Text, Image, FlatList, TouchableOpacity } from "react-native";
import AsyncStorage from "@react-native-async-storage/async-storage";
import chatFaceData from "../services/ChatFaceData"; // bemærk lille c da det er data vi importerer
import GlobalStyles from "../styles/GlobalStyles";
```

2. Lav en `HomeScreen` funktion
`export default function HomeScreen({ navigation }) {`

3. Opret to useStates
```javascript
  const [faces] = useState(chatFaceData); // statisk liste
  const [selectedFace, setSelectedFace] = useState(faces[0]); // default valgt bot
```
Dette er en React Hook ved navn useState, der bruges til at oprette en tilstandsvariabel og en funktion til at opdatere denne tilstand.

4. Herefter laves en useEffect til at hente de valgte chatbot. Her gør vi også brug af en asynkron funktion med try-catch
```javascript
useEffect(() => {
    (async () => {
      try {
        // vi gemmer og læser et 0-baseret index (samme som i ChatScreen)
        const idxStr = await AsyncStorage.getItem("chatFaceId");
        const idx = Number.parseInt(idxStr ?? "0", 10);
        const chosen = faces[idx] ?? faces[0];
        setSelectedFace(chosen);
      } catch (e) {
        setSelectedFace(faces[0]);
      }
    })();
  }, [faces]);
```

5. Til sidst laver vi en funktion, der håndterer, at vi kan skifte imellem dem.
```javascript
  const onChatFacePress = async (id) => {
    const idx = faces.findIndex((f) => f.id === id);
    if (idx >= 0) {
      setSelectedFace(faces[idx]);
      await AsyncStorage.setItem("chatFaceId", String(idx));
    }
  };
```

6. Nu skal vi lave en return statement. Den består af tre views.
```javascript
return (
    <View style={GlobalStyles.homeContainer}>
      <View style={GlobalStyles.homeCenter}>
        <View style={GlobalStyles.homeBotList}>
        </View>
      </View>
    </View>
  );
```

7. Indsæt følgende kode i andet view (homeCenter)
```javascript
        <Text
          style={[GlobalStyles.homeHello, { color: selectedFace?.primary }]}
        >
          Hello,
        </Text>

        <Text style={[GlobalStyles.homeName, { color: selectedFace?.primary }]}>
          I am {selectedFace?.name}
        </Text>

        <Image
          source={selectedFace?.image ? { uri: selectedFace.image } : undefined}
          style={GlobalStyles.homeImage}
        />

        <Text style={GlobalStyles.homeHelp}>How Can I help you?</Text>
```
8. Opret en FlatList i tredje view
```javascript
      <???
            data={faces}
            keyExtractor={(item) => String(item.id)}
            horizontal
            renderItem={({ item }) =>
              item.id !== selectedFace?.id && (
                <TouchableOpacity
                  style={{ margin: 15 }}
                  onPress={() => onChatFacePress(item.id)}
                >
                  <Image
                    source={{ uri: item.image }}
                    style={GlobalStyles.homeBotAvatar}
                  />
                </TouchableOpacity>
              )
            }
          />
```

9. Tilføj dette efter flatlisten (i samme view)
```javascript
          <Text style={GlobalStyles.homeBotListText}>
            Choose Your Fav ChatBuddy
          </Text>
```

10. Til slut skal du oprette en knap, som skal være i `homeCenter` viewet
```javascript
    <TouchableOpacity
          style={[
            GlobalStyles.homeChatButton,
            { backgroundColor: selectedFace?.primary },
          ]}
          onPress={() => navigation.navigate("chat")}
        >
          <Text style={GlobalStyles.homeChatButtonText}>Let's Chat</Text>
    </TouchableOpacity>
```

## ChatScreen.js
1. I `ChatScreen.js` skal du importere følgende:
```javascript
import { useRef } from "react";
import { useState, useEffect, useCallback } from "react";
import { View } from "react-native";
import { SafeAreaView } from "react-native-safe-area-context";
import AsyncStorage from "@react-native-async-storage/async-storage";
import { Bubble, GiftedChat, InputToolbar, Send} from "react-native-gifted-chat";

import { FontAwesome } from "@expo/vector-icons";
import GlobalStyles from "../styles/GlobalStyles";

import chatFaceData from "../services/ChatFaceData";
import SendMessage from "../services/Request";
import SYSTEM_PROMPT from "../services/SystemPrompt"; // vores "trænede" prompt, se nedenfor
import { runGuardrails } from "../services/Guardrails"; // vores guardrails, se nedenfor

let CHAT_BOT_FACE =
  "https://res.cloudinary.com/dknvsbuyy/image/upload/v1685678135/chat_1_c7eda483e3.png";
```

2. Lav funktionen `export default function ChatScreen() {}`
3. Lav tre useState statements som håndterer vores chat
```javascript
  const [messages, setMessages] = useState([]);
  const [loading, setLoading] = useState(false);
  const [chatFaceColor, setChatFaceColor] = useState();
```

4. Lav følgende const med useRef. I stedet for at skrive hele prompten direkte her, henter vi den fra `services/SystemPrompt.js` - det er den fil, hvor selve "træningen" af AI'en foregår (se afsnittet "Træn din AI" nedenfor).
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

5. Lav en useEffect til at hente vores valgte chatbot.
```javascript
  //Håndtere valgt ChatBot
  useEffect(() => {
    checkFaceId();
  }, []);
```

6. Lav en const som sætter den valgte chatbot til og sender den første besked. Der skal bruges async.
```javascript
  const checkFaceId = async () => {
    const idStr = await AsyncStorage.getItem("chatFaceId");
    const idx = Number.parseInt(idStr ?? "0", 10);
    const face = chatFaceData[idx] ?? chatFaceData[0];
    CHAT_BOT_FACE = face.image;
    setChatFaceColor(face.primary);
    setMessages([
      {
        _id: 1,
        text: "Hello, I am " + face.name + ", How Can I help you?",
        createdAt: Date.now(),
        user: {
          _id: 2,
          name: "React Native",
          avatar: CHAT_BOT_FACE,
        },
      },
    ]);
  };
```
Kort fortalt: checkFaceId funktionen tjekker, hvilken chatbot der tidligere er blevet valgt (hvis nogen), sætter chatbotens visuelle repræsentation, og initialiserer en introduktionsbesked fra den valgte chatbot.

**Vigtigt:** Brug `Date.now()` (et tal), ikke `new Date()` (et Date-objekt), til `createdAt`. `react-native-gifted-chat` sender beskeder gennem Reanimated/Worklets, og Worklets kan ikke kopiere et `Date`-objekt over til UI-tråden - gør du det alligevel, crasher appen med `[Worklets] Cannot copy value of type 'Date'`.

7. Vi skal nu lave en funktion, der viser en bot-besked i chatten. Den bliver brugt både til rigtige AI-svar og til guardrail-afvisninger, så vi undgår at skrive den samme kode to gange.
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

8. Lav en const som håndterer, når vi sender en besked
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

9. Før beskeden sendes til selve AI-modellen, skal den igennem vores guardrails (se afsnittet nedenfor). Lav funktionen `handleIncomingMessage`, der kører guardrails, og kun kalder `getBardResp` hvis beskeden bliver godkendt.
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

10. Vi skal nu lave en funktion, der håndterer, når vi modtager en besked fra vores bot
```javascript
//Håndtere API Kald og BOT Svar
  const getBardResp = (msg) => {
    //Sætter brugerens besked i vores chat "hukommelse"
    const userMessage = { role: "user", content: msg };
    conversationHistory.current.push(userMessage);

    //SendMessage er vores funktion fra RequestPage.js
    SendMessage(conversationHistory.current)
      .then((response) => {
        setLoading(false);

        const text = response.content || "Sorry, I cannot help with it";
        if (response.content) {
          //Sætter chat AI's svar i vores chat "hukommelse"
          conversationHistory.current.push({ role: "assistant", content: text });
        }

        respondWithBotMessage(text);
      })
      .catch((error) => {
        setLoading(false);
        console.error(error);
        // Handle error further if needed
      });
  };
```
Funktionen getBardResp tager en parameter msg. Den kalder SendMessage funktionen fra Request.js med hele vores samtale-historik som parameter. Denne funktion bliver kaldt, når en besked er kommet igennem guardrails. Når API'en svarer, sætter funktionen loading til false og viser svaret i chatten via respondWithBotMessage.

11. Vi laver nu en funktion, der laver en bobble til vores chat
```javascript
  const renderBubble = (props) => {
    return (
      <Bubble
        {...props}
        wrapperStyle={{
          right: GlobalStyles.bubbleRight,
        }}
        textStyle={{
          right: GlobalStyles.bubbleRightText,
          left: GlobalStyles.bubbleLeftText,
        }}
      />
    );
  };
```

12. Vi laver nu en funktion, der laver en toolbar til vores chat
```javascript
 const renderInputToolbar = (props) => {
    return (
      <InputToolbar
        {...props}
        containerStyle={GlobalStyles.inputToolbarContainer}
        textInputStyle={GlobalStyles.inputToolbarText}
        textInputProps={{
          ...props.textInputProps,
          editable: true,
          placeholder: "Type a message...",
          placeholderTextColor: "#eee",
        }}
      />
    );
  };
```

**Vigtigt:** Læg mærke til `...props.textInputProps` først i objektet. `GiftedChat` sender sin egen `textInputProps` ned til jeres `renderInputToolbar`, og den indeholder `onChangeText` og en `ref`, som er det, der rent faktisk får teksten til at opdatere sig, når man skriver. Overskriver du hele objektet uden at sprede det oprindelige ind først, kan du ikke skrive i inputfeltet.

13. Vi laver nu en funktion, der laver en send knap til vores chat
```javascript
  const renderSend = (props) => {
    return (
      <Send {...props}>
        <View style={GlobalStyles.sendButton}>
          <FontAwesome
            name="send"
            size={24}
            color="white"
            resizeMode={"center"}
          />
        </View>
      </Send>
    );
  };
```

14. Til sidst skal vi nu lave en return funktion, der indeholder vores chat
```javascript
 return (
    <SafeAreaView style={GlobalStyles.safeArea}>
      <View style={GlobalStyles.mainView}>
        <GiftedChat
          messages={messages}
          isTyping={loading}
          multiline={true}
          onSend={(messages) => onSend(messages)}
          user={{
            _id: 1,
          }}
          renderBubble={renderBubble}
          renderInputToolbar={renderInputToolbar}
          renderSend={renderSend}
        />
      </View>
    </SafeAreaView>
  );
```
ChatScreen.js skulle nu gerne være done - bortset fra de to filer, vi importerede foroven og ikke har lavet endnu. Det er dem, vi skal nu.

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

2. Gå tilbage til `ChatScreen.js` og tjek at I importerer og bruger `SYSTEM_PROMPT` i `conversationHistory` (det gjorde I allerede i trin 4 ovenfor).

3. Test: åbn chatten og prøv at stille et spørgsmål uden for dit emne. Reagerer AI'en som du forventede, ud fra reglerne i din prompt?

## Guardrails

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

7. Gå tilbage til `ChatScreen.js` og bekræft at `handleIncomingMessage` (trin 9 ovenfor) importerer og bruger `runGuardrails`.

8. Test alle fire guardrails:
   - Send en meget lang besked (over 500 tegn)
   - Send to beskeder hurtigt efter hinanden
   - Send noget stødende
   - Send noget, der tydeligvis ikke har noget med dit emne at gøre

   Svarer appen med en passende afvisning hver gang, uden at ramme selve tutor-modellen?

# Fil Checkliste
- [ ] `StackNavigator.js`
- [ ] `App.js`
- [ ] `HomeScreen.js`
- [ ] `ChatScreen.js`
- [ ] `ChatFaceData.js`
- [ ] `Request.js`
- [ ] `SystemPrompt.js`
- [ ] `Guardrails.js`
- [ ] `.env`
- [ ] `babel.config.js`
- [ ] `GlobalStyles.js`
- [ ] `.gitignore`

Fedt, nu skulle det gerne virke, OG din AI er trænet og har guardrails på! Du kan lege med prompten, tilføje flere guardrails, eller prøve at ændre grænserne (maks. beskedlængde, tid mellem beskeder).
