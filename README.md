# 🎓 AI-Workshop

A full-stack AI study assistant built with React (frontend) and Flask (backend), integrating GPT-4, speech-to-text, file upload, and email functionality.

---

## 🔹 Section 1: Frontend

### 🧱 Step 1: Folder Structure

```
ai-buddy/
└── frontend/
    ├── public/
    └── src/
        ├── App.js
        ├── components/
        │   └── ChatBox.js
        ├── index.js
        └── App.css
```

### 🔄 Step 2: Code Flow

```
📄 public/index.html
    ↓
📄 src/index.js
    → ReactDOM renders <App />

📄 src/App.js
    → Renders <ChatBox /> component
    → Imports global styles from App.css

📄 src/components/ChatBox.js
    → Handles:
        - User input
        - Speech-to-text (Web Speech API)
        - File upload (PDF)
        - GPT-4 API call via POST to `/ask`
        - Text-to-speech for responses
        - Email POST to `/send-email`
```

### 💻 Step 3: Add Code to Files

<details>
<summary><code>frontend/public/index.html</code></summary>

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="theme-color" content="#000000" />
    <title>AI Study Buddy</title>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
  </body>
</html>
```
</details>

<details>
<summary><code>frontend/src/index.js</code></summary>

```js
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";
import "./App.css";

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```
</details>

<details>
<summary><code>frontend/src/App.js</code></summary>

```js
import React from "react";
import ChatBox from "./components/ChatBox";
import "./App.css";

function App() {
  return (
    <div className="App">
      {/* <h1>🎓 AI Study Buddy</h1> */}
      <ChatBox />
    </div>
  );
}
export default App;
```
</details>

<details>
<summary><code>frontend/src/components/ChatBox.js</code></summary>

```js
import React, { useState } from "react";

const ChatBox = () => {
  const [prompt, setPrompt] = useState("");
  const [response, setResponse] = useState("");
  const [loading, setLoading] = useState(false);

  const handleAsk = async () => {
    setLoading(true);
    setResponse("");
    try {
      const res = await fetch("http://localhost:5000/ask", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ prompt }),
      });
      const data = await res.json();
      if (data.response) {
        setResponse(data.response);
      } else {
        setResponse("Error: " + data.error);
      }
    } catch (err) {
      setResponse("Error connecting to server.");
    }
    setLoading(false);
  };

  return (
    <div className="chatbox">
      <textarea
        placeholder="Ask a question or paste your notes..."
        value={prompt}
        onChange={(e) => setPrompt(e.target.value)}
      />
      <button onClick={handleAsk} disabled={loading}>
        {loading ? "Asking..." : "Submit"}
      </button>
      <div className="response">
        <strong>Response:</strong>
        <p>{response}</p>
      </div>
    </div>
  );
};

export default ChatBox;
```
</details>

<details>
<summary><code>frontend/src/App.css</code></summary>

```css
.App {
  font-family: Arial, sans-serif;
  padding: 2rem;
  max-width: 700px;
  margin: auto;
  text-align: center;
}

.chatbox textarea {
  width: 100%;
  height: 100px;
  margin-bottom: 1rem;
  padding: 1rem;
  font-size: 1rem;
}

.chatbox button {
  padding: 0.5rem 1rem;
  font-size: 1rem;
  margin-bottom: 1rem;
}

.response {
  background: #f9f9f9;
  padding: 1rem;
  border-radius: 8px;
  text-align: left;
}
```
</details>

<details>
<summary><code>frontend/.gitignore</code></summary>

```
# dependencies
/node_modules
/.pnp
.pnp.js

# testing
/coverage

# production
/build

# misc
.DS_Store
.env
.env.local
.env.development.local
.env.test.local
.env.production.local

# logs
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# IDE
.vscode/
.idea/
*.suo
*.ntvs*
*.njsproj
*.sln
*.sw?

# Mac system files
*.DS_Store
```
</details>

### ▶️ Execution Steps

```bash
cd frontend
npm install
npm start
```

---

## 🔹 Section 2: Backend

### 🧱 Step 1: Folder Structure

```
ai-buddy/
├── backend/
│   ├── app.py
│   ├── requirements.txt
│   └── .env
└── frontend/
    ├── public/
    └── src/
        ├── App.js
        ├── components/
        │   └── ChatBox.js
        ├── index.js
        └── App.css
```

### 🔄 Step 2: Code Flow

```
📄 backend/app.py

Routes:
├── POST /ask
│   → Receives prompt from frontend
│   → Sends to GPT-4 via OpenAI SDK
│   → Returns response to React

├── POST /send-email
│   → Receives { prompt, response, to_email }
│   → Sends email via SMTP (Gmail)

Config:
📄 .env
→ Stores:
   - OPENAI_API_KEY
   - SMTP_SENDER
   - SMTP_PASSWORD
```

### 💻 Step 3: Add Code to Files

<details>
<summary><code>backend/app.py</code></summary>

```python
from flask import Flask, request, jsonify
from flask_cors import CORS
from openai import OpenAI
import os
from dotenv import load_dotenv

load_dotenv()

# Initialize OpenAI client with API key
client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

app = Flask(__name__)
CORS(app)

@app.route("/ask", methods=["POST"])
def ask():
    data = request.get_json()
    prompt = data.get("prompt")

    if not prompt:
        return jsonify({"error": "No prompt provided"}), 400

    try:
        response = client.chat.completions.create(
            model="gpt-4",
            messages=[{"role": "user", "content": prompt}],
            temperature=0.7,
            max_tokens=500
        )
        answer = response.choices[0].message.content.strip()
        return jsonify({"response": answer})
    except Exception as e:
        return jsonify({"error": str(e)}), 500

if __name__ == "__main__":
    app.run(debug=True)
```
</details>

<details>
<summary><code>backend/requirements.txt</code></summary>

```
flask
flask-cors
openai
python-dotenv
```
</details>

<details>
<summary><code>backend/.env</code></summary>

```
OPENAI_API_KEY=your_openai_key
```
</details>

### ▶️ Execution Steps

```bash
cd backend
pip install -r requirements.txt
python app.py
```

---

## 🔁 Final Flow Overview

```
index.html
   ↓
index.js
   ↓
App.js
   ↓
ChatBox.js
   ↓
→ POST /ask → app.py → GPT-4 → Response
→ POST /send-email → app.py → SMTP → Email sent
```

---

## 📌 Notes

- Ensure environment variables are securely stored in `.env`
- You need an OpenAI API key and SMTP credentials for full functionality
- CORS is enabled to allow frontend-backend communication locally

---

Feel free to customize, fork, and improve!

---

### ▶️ For adding Speech to Text and Text to Speech and Upload PDF Features:

<details>
<summary><code>frontend/src/components/ChatBox.js</code></summary>

```js
import React, { useState, useRef } from "react";
import * as pdfjsLib from "pdfjs-dist/legacy/build/pdf";
import "pdfjs-dist/build/pdf.worker.entry";

const ChatBox = () => {
  const [prompt, setPrompt] = useState("");
  const [response, setResponse] = useState("");
  const [loading, setLoading] = useState(false);
  const recognitionRef = useRef(null);

  const handleAsk = async () => {
    setLoading(true);
    setResponse("");
    try {
      const res = await fetch("http://localhost:5000/ask", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ prompt }),
      });
      const data = await res.json();
      if (data.response) {
        setResponse(data.response);
        speakResponse(data.response); // 👂 Speak it
      } else {
        setResponse("Error: " + data.error);
      }
    } catch (err) {
      setResponse("Error connecting to server.");
    }
    setLoading(false);
  };

  const startSpeechRecognition = () => {
    const SpeechRecognition =
      window.SpeechRecognition || window.webkitSpeechRecognition;
    if (!SpeechRecognition) {
      alert("Speech Recognition not supported in this browser.");
      return;
    }

    const recognition = new SpeechRecognition();
    recognition.lang = "en-US";
    recognition.continuous = false;
    recognition.interimResults = false;

    recognition.onresult = (event) => {
      const transcript = event.results[0][0].transcript;
      setPrompt((prev) => prev + " " + transcript);
    };

    recognition.start();
    recognitionRef.current = recognition;
  };

  const speakResponse = (text) => {
    const synth = window.speechSynthesis;
    if (!synth) return;

    const utter = new SpeechSynthesisUtterance(text);
    utter.lang = "en-US";
    synth.speak(utter);
  };

  const handlePdfUpload = async (event) => {
    const file = event.target.files[0];
    if (file && file.type === "application/pdf") {
      const fileReader = new FileReader();
      fileReader.onload = async function () {
        const typedArray = new Uint8Array(this.result);
        const pdf = await pdfjsLib.getDocument(typedArray).promise;
        let text = "";

        for (let i = 1; i <= pdf.numPages; i++) {
          const page = await pdf.getPage(i);
          const content = await page.getTextContent();
          const pageText = content.items.map((item) => item.str).join(" ");
          text += pageText + "\n";
        }
        setPrompt(text);
      };
      fileReader.readAsArrayBuffer(file);
    }
  };

  return (
    <div className="chatbox">
      <textarea
        placeholder="Ask a question or paste your notes..."
        value={prompt}
        onChange={(e) => setPrompt(e.target.value)}
      />
      <br />
      <button onClick={handleAsk} disabled={loading}>
        {loading ? "Processing..." : "Submit"}
      </button>
      <button onClick={startSpeechRecognition}>🎙️ Speak</button>
      <input type="file" accept=".pdf" onChange={handlePdfUpload} />
      <div className="response">
        <strong>Response:</strong>
        <p>{response}</p>
        <button onClick={() => speakResponse(response)}>🔊 Read Response</button>
      </div>
    </div>
  );
};

export default ChatBox;

```
</details>

---

### ▶️ For adding Agent:

<details>
<summary><code>frontend/src/components/ChatBox.js</code></summary>

```js
import React, { useState, useRef } from "react";
import * as pdfjsLib from "pdfjs-dist/legacy/build/pdf";
import "pdfjs-dist/build/pdf.worker.entry";

const ChatBox = () => {
  const [prompt, setPrompt] = useState("");
  const [response, setResponse] = useState("");
  const [loading, setLoading] = useState(false);
  const [email, setEmail] = useState("");
  const [emailStatus, setEmailStatus] = useState("");
  const recognitionRef = useRef(null);

  const handleAsk = async () => {
    setLoading(true);
    setResponse("");
    try {
      const res = await fetch("http://localhost:5000/ask", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ prompt }),
      });
      const data = await res.json();
      if (data.response) {
        setResponse(data.response);
        speakResponse(data.response);
      } else {
        setResponse("Error: " + data.error);
      }
    } catch (err) {
      setResponse("Error connecting to server.");
    }
    setLoading(false);
  };

  const startSpeechRecognition = () => {
    const SpeechRecognition =
      window.SpeechRecognition || window.webkitSpeechRecognition;
    if (!SpeechRecognition) {
      alert("Speech Recognition not supported in this browser.");
      return;
    }
    const recognition = new SpeechRecognition();
    recognition.lang = "en-US";
    recognition.continuous = false;
    recognition.interimResults = false;
    recognition.onresult = (event) => {
      const transcript = event.results[0][0].transcript;
      setPrompt((prev) => prev + " " + transcript);
    };
    recognition.start();
    recognitionRef.current = recognition;
  };

  const speakResponse = (text) => {
    const synth = window.speechSynthesis;
    if (!synth) return;
    const utter = new SpeechSynthesisUtterance(text);
    utter.lang = "en-US";
    synth.speak(utter);
  };

  const handlePdfUpload = async (event) => {
    const file = event.target.files[0];
    if (file && file.type === "application/pdf") {
      const fileReader = new FileReader();
      fileReader.onload = async function () {
        const typedArray = new Uint8Array(this.result);
        const pdf = await pdfjsLib.getDocument(typedArray).promise;
        let text = "";
        for (let i = 1; i <= pdf.numPages; i++) {
          const page = await pdf.getPage(i);
          const content = await page.getTextContent();
          const pageText = content.items.map((item) => item.str).join(" ");
          text += pageText + "\n";
        }
        setPrompt(text);
      };
      fileReader.readAsArrayBuffer(file);
    }
  };

  const sendEmail = async () => {
    setEmailStatus("Sending...");
    try {
      const res = await fetch("http://localhost:5000/send-email", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ to: email, prompt, response }),
      });
      const data = await res.json();
      if (data.success) {
        setEmailStatus("\ud83d\udce9 Email sent!");
      } else {
        setEmailStatus("\u274c Failed: " + data.error);
      }
    } catch (err) {
      setEmailStatus("\u274c Network error.");
    }
  };

  return (
    <div className="chatbox">
      <textarea
        placeholder="Ask a question or paste your notes..."
        value={prompt}
        onChange={(e) => setPrompt(e.target.value)}
      />
      <br />
      <button onClick={handleAsk} disabled={loading}>
        {loading ? "Processing..." : "Submit"}
      </button>
      <button onClick={startSpeechRecognition}>🎙️ Speak</button>
      <input type="file" accept=".pdf" onChange={handlePdfUpload} />
      <div className="response">
        <strong>Response:</strong>
        <p>{response}</p>
        <button onClick={() => speakResponse(response)}>🔊 Read Response</button>
      </div>
      <div style={{ marginTop: "1rem" }}>
        <input
          type="email"
          placeholder="Your email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          style={{ padding: "0.5rem", width: "60%" }}
        />
        <button onClick={sendEmail} disabled={!response || !email}>
          📧 Send to Email
        </button>
        <div>{emailStatus}</div>
      </div>
    </div>
  );
};

export default ChatBox;


```
</details>


<details>
<summary><code>backend/app.py</code></summary>

```python
from flask import Flask, request, jsonify
from flask_cors import CORS
from openai import OpenAI
import os
import smtplib
from email.mime.text import MIMEText
from dotenv import load_dotenv

load_dotenv()

app = Flask(__name__)
CORS(app)

# Initialize OpenAI client
client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

@app.route("/ask", methods=["POST"])
def ask():
    data = request.get_json()
    prompt = data.get("prompt")

    if not prompt:
        return jsonify({"error": "No prompt provided"}), 400

    try:
        # New OpenAI v1.x syntax
        response = client.chat.completions.create(
            model="gpt-4",
            messages=[{"role": "user", "content": prompt}],
            temperature=0.7,
            max_tokens=500
        )
        answer = response.choices[0].message.content.strip()
        return jsonify({"response": answer})
    except Exception as e:
        return jsonify({"error": str(e)}), 500

@app.route("/send-email", methods=["POST"])
def send_email():
    data = request.get_json()
    to_email = data.get("to")
    question = data.get("prompt")
    answer = data.get("response")

    if not (to_email and question and answer):
        return jsonify({"error": "Missing fields"}), 400

    subject = "Your AI Study Buddy Response"
    body = f"Your Question:\n{question}\n\nGPT-4 Response:\n{answer}"

    msg = MIMEText(body)
    msg['Subject'] = subject
    msg['From'] = os.getenv("SMTP_SENDER")
    msg['To'] = to_email

    try:
        with smtplib.SMTP_SSL("smtp.gmail.com", 465) as server:
            server.login(os.getenv("SMTP_SENDER"), os.getenv("SMTP_PASSWORD"))
            server.send_message(msg)
        return jsonify({"success": True})
    except Exception as e:
        return jsonify({"error": str(e)}), 500

if __name__ == "__main__":
    app.run(debug=True)

```
</details>

<details>
<summary><code>backend/.env</code></summary>

```
OPENAI_API_KEY=your_openai_key
SMTP_SENDER=your_email@gmail.com
SMTP_PASSWORD=your_email_app_password

```
</details>

