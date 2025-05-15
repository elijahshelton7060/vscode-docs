<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Chat with Emily - Smart AI Assistant</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background-color: #f0f4ff;
      margin: 0;
      padding: 0;
      display: flex;
      flex-direction: column;
      align-items: center;
      height: 100vh;
      justify-content: flex-start;
    }
    h1 {
      background: linear-gradient(to right, #6c63ff, #3f47d6);
      color: white;
      padding: 24px;
      text-align: center;
      width: 100%;
      margin: 0;
    }
    #chat-box {
      width: 90%;
      max-width: 600px;
      height: 60vh;
      overflow-y: auto;
      margin: 20px 0;
      background: white;
      border-radius: 12px;
      padding: 15px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.1);
    }
    .user-message, .ai-message {
      margin: 10px 0;
      padding: 10px 16px;
      border-radius: 18px;
      max-width: 80%;
    }
    .user-message {
      align-self: flex-end;
      background-color: #4caf50;
      color: white;
    }
    .ai-message {
      align-self: flex-start;
      background-color: #3f51b5;
      color: white;
    }
    #input-form {
      display: flex;
      width: 90%;
      max-width: 600px;
      margin-bottom: 20px;
    }
    #text-input {
      flex: 1;
      padding: 12px;
      border-radius: 8px 0 0 8px;
      border: 1px solid #ccc;
      font-size: 16px;
    }
    #submit-btn {
      padding: 12px 16px;
      border: none;
      background-color: #6c63ff;
      color: white;
      font-size: 16px;
      border-radius: 0 8px 8px 0;
      cursor: pointer;
    }
    .listening-indicator {
      color: green;
      font-size: 14px;
    }
  </style>
</head>
<body>
  <h1>Talk to Emily, Your Smart AI Assistant</h1>
  <div id="chat-box"></div>
  <form id="input-form">
    <input type="text" id="text-input" placeholder="Type your question here..." />
    <button type="submit" id="submit-btn">Send</button>
  </form>
  <div id="status" class="listening-indicator">🎤 Listening for "Hey Emily"...</div>

  <script>
    const chatBox = document.getElementById("chat-box");
    const inputForm = document.getElementById("input-form");
    const textInput = document.getElementById("text-input");
    const status = document.getElementById("status");

    const OPENAI_API_KEY = 'YOUR_OPENAI_API_KEY'; // Replace with your actual key

    let recognition;
    let isListening = false;

    // Initialize speech recognition
    function initRecognition() {
      const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
      if (!SpeechRecognition) {
        alert("Your browser doesn't support Speech Recognition. Try Chrome.");
        return;
      }

      recognition = new SpeechRecognition();
      recognition.lang = 'en-US';
      recognition.interimResults = false;
      recognition.continuous = true;

      recognition.onresult = async (event) => {
        const transcript = event.results[event.results.length - 1][0].transcript.trim();
        console.log("Heard:", transcript);
        if (/^hey emily/i.test(transcript)) {
          status.textContent = "👂 Listening...";
          speak("Yes, how can I help you?");
          isListening = true;
        } else if (isListening) {
          appendMessage(transcript, 'user-message');
          const response = await askEmily(transcript);
          appendMessage(response, 'ai-message');
          speak(response);
          isListening = false;
          status.textContent = "🎤 Listening for 'Hey Emily'...";
        }
      };

      recognition.onerror = (event) => {
        console.error("Speech error:", event.error);
        speak("Oops, something went wrong.");
      };

      recognition.onend = () => {
        recognition.start(); // Auto-restart
      };

      recognition.start();
    }

    // Append messages to chat box
    function appendMessage(message, className) {
      const div = document.createElement("div");
      div.className = className;
      div.textContent = message;
      chatBox.appendChild(div);
      chatBox.scrollTop = chatBox.scrollHeight;
    }

    // Text-to-speech
    function speak(text) {
      const utterance = new SpeechSynthesisUtterance(text);
      utterance.lang = "en-US";
      utterance.rate = 1.05;
      utterance.pitch = 1.1;
      const voice = speechSynthesis.getVoices().find(v => /female|samantha|zira|karen/i.test(v.name)) || speechSynthesis.getVoices()[0];
      utterance.voice = voice;
      speechSynthesis.speak(utterance);
    }

    // OpenAI API call
    async function askEmily(prompt) {
      try {
        const res = await fetch("https://api.openai.com/v1/chat/completions", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
            "Authorization": `Bearer ${OPENAI_API_KEY}`,
          },
          body: JSON.stringify({
            model: "gpt-3.5-turbo",
            messages: [{ role: "user", content: prompt }],
            temperature: 0.7,
            max_tokens: 300,
          }),
        });

        const data = await res.json();
        return data.choices?.[0]?.message?.content || "I'm not sure how to respond.";
      } catch (err) {
        console.error("API error:", err);
        return "Sorry, I couldn't reach my brain right now.";
      }
    }

    // Handle text input
    inputForm.addEventListener("submit", async (e) => {
      e.preventDefault();
      const msg = textInput.value.trim();
      if (!msg) return;
      appendMessage(msg, 'user-message');
      const response = await askEmily(msg);
      appendMessage(response, 'ai-message');
      speak(response);
      textInput.value = "";
    });

    window.onload = () => {
      window.speechSynthesis.onvoiceschanged = () => {}; // ensure voices load
      initRecognition();
    };
  </script>
</body>
</html>
