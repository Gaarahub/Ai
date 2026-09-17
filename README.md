// ============================================================
// CHAT — IA PERSONAL
// Vercel + Vercel AI Gateway + OpenAI
//
// ARCHIVO: api/index.js
//
// VARIABLE DE ENTORNO NECESARIA EN VERCEL:
// AI_GATEWAY_API_KEY = TU_CLAVE_DE_VERCEL
//
// NO pongas tu API key dentro de este archivo.
// ============================================================

const MODEL = "openai/gpt-5.6-luna";

const SYSTEM_PROMPT = `
Tu nombre es Chat.

Eres un asistente de inteligencia artificial creado para ayudar
al usuario de forma clara, natural, útil y amigable.

Reglas:
- Responde en el idioma que utilice el usuario.
- Sé natural y conversacional.
- Explica las cosas de manera sencilla cuando sea necesario.
- No inventes información cuando no estés seguro.
- Puedes ayudar con programación, matemáticas, escritura,
  tecnología, ideas, aprendizaje y preguntas generales.
- Si el usuario pide una explicación paso a paso, dásela.
- No digas que tienes acceso a información que realmente no tienes.
- Tu identidad dentro de esta aplicación es "Chat".
`;

function escapeHTML(text) {
  return String(text)
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&#039;");
}

function getHTML() {
  return `<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta
  name="viewport"
  content="width=device-width,
  initial-scale=1.0,
  maximum-scale=1.0,
  user-scalable=no"
>
<title>Chat</title>

<style>
* {
  box-sizing: border-box;
  -webkit-tap-highlight-color: transparent;
}

html,
body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  background: #000;
  color: #fff;
  font-family:
    -apple-system,
    BlinkMacSystemFont,
    "Segoe UI",
    Arial,
    sans-serif;
}

body {
  overflow: hidden;
}

.app {
  width: 100%;
  height: 100dvh;
  display: flex;
  flex-direction: column;
  background: #000;
}

/* =========================
   BARRA SUPERIOR
========================= */

.header {
  height: 64px;
  min-height: 64px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 16px;
  border-bottom: 1px solid #202020;
  background: #050505;
}

.logo {
  display: flex;
  align-items: center;
  gap: 10px;
}

.logo-circle {
  width: 34px;
  height: 34px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #fff;
  color: #000;
  font-weight: 800;
  font-size: 17px;
}

.logo-text {
  font-size: 19px;
  font-weight: 700;
}

.clear-button {
  border: 1px solid #2b2b2b;
  background: #111;
  color: #ddd;
  padding: 9px 12px;
  border-radius: 10px;
  font-size: 13px;
  cursor: pointer;
}

.clear-button:active {
  transform: scale(.97);
  background: #191919;
}

/* =========================
   MENSAJES
========================= */

.messages {
  flex: 1;
  min-height: 0;
  overflow-y: auto;
  padding: 22px 14px 130px;
  scroll-behavior: smooth;
}

.welcome {
  max-width: 700px;
  margin: 12vh auto 0;
  text-align: center;
  padding: 20px;
}

.welcome-icon {
  width: 66px;
  height: 66px;
  margin: 0 auto 18px;
  border-radius: 22px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #fff;
  color: #000;
  font-size: 30px;
  font-weight: 800;
}

.welcome h1 {
  margin: 0 0 10px;
  font-size: 30px;
}

.welcome p {
  margin: 0;
  color: #858585;
  font-size: 15px;
  line-height: 1.5;
}

.message-row {
  width: 100%;
  max-width: 850px;
  margin: 0 auto 18px;
  display: flex;
}

.message-row.user {
  justify-content: flex-end;
}

.message-row.assistant {
  justify-content: flex-start;
}

.message {
  max-width: 82%;
  padding: 13px 15px;
  border-radius: 17px;
  font-size: 15px;
  line-height: 1.55;
  white-space: pre-wrap;
  word-break: break-word;
}

.user .message {
  background: #fff;
  color: #000;
  border-bottom-right-radius: 5px;
}

.assistant .message {
  background: #171717;
  color: #f2f2f2;
  border: 1px solid #242424;
  border-bottom-left-radius: 5px;
}

.error .message {
  background: #260d0d;
  border-color: #5a2020;
  color: #ffb3b3;
}

/* =========================
   ESCRIBIENDO
========================= */

.typing {
  display: inline-flex;
  gap: 5px;
  align-items: center;
  height: 20px;
}

.typing span {
  width: 6px;
  height: 6px;
  background: #aaa;
  border-radius: 50%;
  animation: bounce 1.1s infinite;
}

.typing span:nth-child(2) {
  animation-delay: .15s;
}

.typing span:nth-child(3) {
  animation-delay: .3s;
}

@keyframes bounce {
  0%, 60%, 100% {
    transform: translateY(0);
    opacity: .4;
  }

  30% {
    transform: translateY(-5px);
    opacity: 1;
  }
}

/* =========================
   ZONA DE ESCRITURA
========================= */

.composer-wrapper {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  padding:
    10px 12px
    calc(10px + env(safe-area-inset-bottom))
    12px;
  background:
    linear-gradient(
      to top,
      #000 65%,
      transparent
    );
}

.composer {
  width: 100%;
  max-width: 850px;
  margin: 0 auto;
  display: flex;
  align-items: flex-end;
  gap: 8px;
  padding: 8px;
  border-radius: 18px;
  border: 1px solid #2b2b2b;
  background: #111;
  box-shadow: 0 8px 35px rgba(0,0,0,.55);
}

textarea {
  flex: 1;
  min-width: 0;
  max-height: 130px;
  resize: none;
  border: 0;
  outline: 0;
  padding: 11px 10px;
  background: transparent;
  color: white;
  font-family: inherit;
  font-size: 16px;
  line-height: 1.4;
}

textarea::placeholder {
  color: #666;
}

.send {
  width: 43px;
  height: 43px;
  flex: 0 0 43px;
  border: 0;
  border-radius: 13px;
  background: white;
  color: black;
  font-size: 18px;
  font-weight: 800;
  cursor: pointer;
}

.send:disabled {
  opacity: .35;
  cursor: default;
}

.send:active:not(:disabled) {
  transform: scale(.94);
}

/* =========================
   DESKTOP
========================= */

@media (min-width: 700px) {
  .messages {
    padding-left: 24px;
    padding-right: 24px;
  }

  .message {
    font-size: 16px;
  }

  .welcome h1 {
    font-size: 36px;
  }
}
</style>
</head>

<body>

<div class="app">

  <header class="header">
    <div class="logo">
      <div class="logo-circle">C</div>
      <div class="logo-text">Chat</div>
    </div>

    <button
      class="clear-button"
      onclick="clearChat()"
    >
      Limpiar
    </button>
  </header>

  <main
    id="messages"
    class="messages"
  >
    <div
      id="welcome"
      class="welcome"
    >
      <div class="welcome-icon">C</div>

      <h1>¿En qué puedo ayudarte?</h1>

      <p>
        Soy Chat, tu asistente de inteligencia artificial.
        Escribe un mensaje para comenzar.
      </p>
    </div>
  </main>

  <div class="composer-wrapper">
    <div class="composer">

      <textarea
        id="input"
        rows="1"
        placeholder="Escribe un mensaje..."
        autocomplete="off"
      ></textarea>

      <button
        id="send"
        class="send"
        onclick="sendMessage()"
        aria-label="Enviar"
      >
        ↑
      </button>

    </div>
  </div>

</div>

<script>

let conversation = [];

const input = document.getElementById("input");
const send = document.getElementById("send");
const messages = document.getElementById("messages");
const welcome = document.getElementById("welcome");

input.addEventListener("input", () => {
  input.style.height = "auto";
  input.style.height =
    Math.min(input.scrollHeight, 130) + "px";
});

input.addEventListener("keydown", (event) => {

  if (event.key === "Enter" && !event.shiftKey) {
    event.preventDefault();
    sendMessage();
  }

});

function scrollToBottom() {
  setTimeout(() => {
    messages.scrollTop = messages.scrollHeight;
  }, 50);
}

function addMessage(role, text, isError = false) {

  if (welcome) {
    welcome.style.display = "none";
  }

  const row = document.createElement("div");

  row.className =
    "message-row " +
    (role === "user" ? "user" : "assistant") +
    (isError ? " error" : "");

  const bubble = document.createElement("div");

  bubble.className = "message";

  bubble.textContent = text;

  row.appendChild(bubble);
  messages.appendChild(row);

  scrollToBottom();

  return row;
}

function addTyping() {

  if (welcome) {
    welcome.style.display = "none";
  }

  const row = document.createElement("div");

  row.className =
    "message-row assistant";

  row.id = "typing-message";

  row.innerHTML = \`
    <div class="message">
      <div class="typing">
        <span></span>
        <span></span>
        <span></span>
      </div>
    </div>
  \`;

  messages.appendChild(row);

  scrollToBottom();
}

function removeTyping() {

  const typing =
    document.getElementById("typing-message");

  if (typing) {
    typing.remove();
  }
}

async function sendMessage() {

  const text = input.value.trim();

  if (!text || send.disabled) {
    return;
  }

  addMessage("user", text);

  conversation.push({
    role: "user",
    content: text
  });

  input.value = "";
  input.style.height = "auto";

  send.disabled = true;

  addTyping();

  try {

    const response = await fetch("/api/chat", {

      method: "POST",

      headers: {
        "Content-Type": "application/json"
      },

      body: JSON.stringify({
        messages: conversation
      })

    });

    const data = await response.json();

    removeTyping();

    if (!response.ok) {

      addMessage(
        "assistant",
        data.error ||
        "Ocurrió un error.",
        true
      );

      return;
    }

    addMessage(
      "assistant",
      data.response
    );

    conversation.push({
      role: "assistant",
      content: data.response
    });

  } catch (error) {

    removeTyping();

    addMessage(
      "assistant",
      "No pude conectarme con la IA. Comprueba la configuración de Vercel.",
      true
    );

  } finally {

    send.disabled = false;
    input.focus();

  }
}

function clearChat() {

  conversation = [];

  messages.innerHTML = \`
    <div
      id="welcome"
      class="welcome"
    >
      <div class="welcome-icon">C</div>

      <h1>¿En qué puedo ayudarte?</h1>

      <p>
        Soy Chat, tu asistente de inteligencia artificial.
        Escribe un mensaje para comenzar.
      </p>
    </div>
  \`;

  input.focus();
}

</script>

</body>
</html>`;
}

// ============================================================
// VERCEL FUNCTION
// ============================================================

module.exports = async function handler(req, res) {

  // -------------------------
  // PÁGINA
  // -------------------------

  if (req.method === "GET") {

    res.setHeader(
      "Content-Type",
      "text/html; charset=utf-8"
    );

    return res.status(200).send(getHTML());
  }

  // -------------------------
  // CHAT
  // -------------------------

  if (req.method === "POST") {

    try {

      const apiKey =
        process.env.AI_GATEWAY_API_KEY;

      if (!apiKey) {

        return res.status(500).json({
          error:
            "Falta configurar AI_GATEWAY_API_KEY en Vercel."
        });

      }

      const body =
        typeof req.body === "string"
          ? JSON.parse(req.body)
          : req.body;

      const incomingMessages =
        Array.isArray(body?.messages)
          ? body.messages
          : [];

      const messages =
        incomingMessages
          .filter(
            item =>
              item &&
              (item.role === "user" ||
               item.role === "assistant") &&
              typeof item.content === "string"
          )
          .slice(-30);

      if (messages.length === 0) {

        return res.status(400).json({
          error: "No recibí ningún mensaje."
        });

      }

      const gatewayResponse =
        await fetch(
          "https://ai-gateway.vercel.sh/v1/chat/completions",
          {
            method: "POST",

            headers: {
              "Authorization":
                "Bearer " + apiKey,

              "Content-Type":
                "application/json"
            },

            body: JSON.stringify({

              model: MODEL,

              messages: [
                {
                  role: "system",
                  content: SYSTEM_PROMPT
                },
                ...messages
              ]

            })
          }
        );

      const data =
        await gatewayResponse.json();

      if (!gatewayResponse.ok) {

        console.error(
          "AI Gateway error:",
          data
        );

        return res.status(500).json({
          error:
            "El servicio de IA devolvió un error."
        });

      }

      const answer =
        data?.choices?.[0]?.message?.content;

      if (!answer) {

        return res.status(500).json({
          error:
            "La IA no devolvió una respuesta."
        });

      }

      return res.status(200).json({
        response: answer
      });

    } catch (error) {

      console.error(
        "Server error:",
        error
      );

      return res.status(500).json({
        error:
          "Error interno del servidor."
      });
    }
  }

  // -------------------------
  // MÉTODO NO PERMITIDO
  // -------------------------

  res.setHeader(
    "Allow",
    "GET, POST"
  );

  return res.status(405).json({
    error: "Método no permitido."
  });
};
