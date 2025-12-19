import React, { useState, useEffect, useRef } from "react";

function App() {
  const [messages, setMessages] = useState([]);
  const [input, setInput] = useState("");
  const apiKey = "YOUR_OPENAI_API_KEY";
  const messagesEndRef = useRef(null);

  // Voice recognition
  const [recognition, setRecognition] = useState(null);

  useEffect(() => {
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    if (SpeechRecognition) {
      const rec = new SpeechRecognition();
      rec.continuous = false;
      rec.interimResults = false;
      rec.lang = "en-US";
      rec.onresult = (event) => {
        setInput(event.results[0][0].transcript);
      };
      setRecognition(rec);
    }
  }, []);

  const startVoice = () => recognition && recognition.start();

  const speak = (text) => {
    const utterance = new SpeechSynthesisUtterance(text);
    window.speechSynthesis.speak(utterance);
  };

  const sendMessage = async () => {
    if (!input) return;
    const userMessage = { role: "user", content: input };
    setMessages([...messages, userMessage]);
    setInput("");

    try {
      const res = await fetch("https://api.openai.com/v1/chat/completions", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Authorization": `Bearer ${apiKey}`,
        },
        body: JSON.stringify({
          model: "gpt-3.5-turbo",
          messages: [...messages, userMessage],
        }),
      });

      const data = await res.json();
      const botMessage = { role: "bot", content: data.choices[0].message.content };
      setMessages(prev => [...prev, botMessage]);
      speak(botMessage.content);
    } catch (err) {
      const errMessage = { role: "bot", content: "Server error 😓" };
      setMessages(prev => [...prev, errMessage]);
    }
  };

  useEffect(() => {
    messagesEndRef.current?.scrollIntoView({ behavior: "smooth" });
  }, [messages]);

  return (
    <div style={{ padding: 20, fontFamily: "Arial, sans-serif", background: "#111", color: "#fff", minHeight: "100vh" }}>
      <h1 style={{ textAlign: "center", marginBottom: 20 }}>HopWeb ChatGPT</h1>
      <div style={{ height: "60vh", overflowY: "scroll", border: "1px solid #444", padding: 10, borderRadius: 8, marginBottom: 10 }}>
        {messages.map((msg, i) => (
          <div key={i} style={{ textAlign: msg.role === "user" ? "right" : "left", margin: "10px 0" }}>
            <span style={{ background: msg.role === "user" ? "#0af" : "#333", color: "#fff", padding: "8px 12px", borderRadius: 8, display: "inline-block" }}>
              {msg.content}
            </span>
          </div>
        ))}
        <div ref={messagesEndRef} />
      </div>

      <div style={{ display: "flex" }}>
        <input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          placeholder="Type or speak..."
          style={{ flex: 1, padding: 10, borderRadius: 8, border: "1px solid #555", background: "#222", color: "#fff" }}
        />
        <button onClick={sendMessage} style={{ marginLeft: 8, padding: "0 16px", borderRadius: 8, background: "#0af", color: "#fff" }}>Send</button>
        <button onClick={startVoice} style={{ marginLeft: 8, padding: "0 16px", borderRadius: 8, background: "#f90", color: "#fff" }}>🎤 Speak</button>
      </div>
    </div>
  );
}

export default App;
