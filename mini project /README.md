# 🎓 GenAI Mentor Studio

An AI-powered, multimodal learning assistant. It combines six study tools in one web app that runs from a **single Google Colab cell**, with no installation and no local setup.

> Built for the Generative AI course, Department of Information Technology, VESIT (University of Mumbai), AY 2026-27.

---

## ✨ Features

| Tab | What it does |
|---|---|
| 💬 **Mentor Chat** | Multi-turn tutor with 4 personalities (Patient Professor, Friendly Study Buddy, Socratic Coach, Exam Strategist). Answers stream live, word by word. |
| 👁️ **Vision Tutor** | Upload a diagram, handwritten notes or a problem photo. The AI explains or solves it step by step. |
| 🧠 **Quiz Arena** | Enter a topic and difficulty. Get 5 AI-generated MCQs, automatic grading, a score and explanations. |
| 🎧 **Explain & Listen** | Generates a short spoken-style explanation and converts it to an MP3. |
| 📄 **Chat with PDF** | Upload a text-based PDF, ask questions or get a summary. Answers cite pages like `[PAGE 3]`. |
| 🗺️ **Mind Map Maker** | Turns any topic into a colour-coded mind map (SVG) that you can download. |

---

## 🚀 Quick Start (Google Colab)

1. Get a free Gemini API key from [Google AI Studio](https://aistudio.google.com/).
2. Open [Google Colab](https://colab.research.google.com/) and create a new notebook.
3. Copy the **entire** contents of `genai_mentor_studio_colab.py` into one empty cell.
4. Run the cell and wait for the packages to install.
5. Paste your API key when asked (the input is hidden).
6. Click the public `https://xxxx.gradio.live` link printed at the end.

Keep the Colab cell running while you use the app. The public link lasts about 72 hours and stops when the Colab session ends.

---

## 🧱 Tech Stack

- **Python 3** on **Google Colab**
- **Gradio 5.49.1**: web interface and server
- **google-genai** (version below 2): Gemini API client with streaming
- **Gemini Flash models**: chosen automatically from the models your key can use
- **PyMuPDF**: PDF text extraction
- **Pillow**: image handling
- **gTTS**: text-to-speech
- **SVG**: mind map drawing (own layout algorithm, no extra library)

> `google-genai` is pinned below version 2 because the newest release conflicts with the `pydantic` version required by Gradio 5.

---

## 🏗️ How It Works

```
Browser (Gradio UI)
      │
      ▼
Gradio server + tool handlers (Python on Colab)
      │
      ▼
Core: prompt builder + stream_ai()
  • picks an available Gemini model
  • low "thinking" level for speed
  • falls back to another model on error
      │   HTTPS + JSON, streamed back with SSE
      ▼
Gemini API (Google cloud)
```

Key design points:

- **Streaming:** `stream_ai()` is a Python generator, so the first words appear within seconds.
- **Model auto-detection:** the app lists the models available to your key and orders them by preference, so a retired model name cannot break it.
- **Fast reasoning setting:** Gemini 3 models are set to a low thinking level (Gemini 2.5 Flash models use a thinking budget of zero) to reduce delay.
- **Structured output:** quizzes and mind maps use JSON mode and are validated before use. For example, a quiz question is kept only if its correct answer matches one of its options.
- **PDF grounding:** the first 40,000 characters of the PDF are placed in the prompt with page tags, and the model is told to answer only from the document. This is context stuffing, **not** full RAG (no embeddings or vector database).
- **Mind map layout:** branches are placed at angles `θ = 2πi/n − π/2` on an ellipse, children are fanned around the parent, and neighbours are staggered to avoid overlap.

---

## 📁 Project Structure

The whole project is one file:

```
genai_mentor_studio_colab.py   # paste into a single Colab cell
```

Main functions inside it:

| Function | Purpose |
|---|---|
| `pick_models()` | Lists available models and orders them by preference |
| `make_config()` | Builds generation settings (system prompt, JSON mode, thinking level) |
| `stream_ai()` | Streaming generator with model fallback |
| `chat()` | Mentor Chat handler |
| `vision()` | Vision Tutor handler |
| `make_quiz()`, `grade()` | Quiz creation, validation and grading |
| `explain_listen()` | Script generation and text-to-speech |
| `load_pdf()`, `pdf_ask()` | PDF reading (cached) and document Q&A |
| `build_mindmap_svg()`, `make_mindmap()` | Mind map layout and SVG output |

---

## ⚙️ Configuration

- **API key:** the app reads the `GEMINI_API_KEY` environment variable if it is set, otherwise it asks for the key with a hidden prompt.
- **Models:** edit the `PREFERRED` list near the top of the file to change which Gemini models are tried first. Put a Flash-Lite model first for even faster responses.
- **Personalities:** add or edit entries in the `PERSONAS` dictionary.
- **Quiz length:** change `NQ` (number of questions, default 5).

---

## 🛠️ Troubleshooting

| Problem | What to try |
|---|---|
| "All models failed" error | Check that your API key is correct and has quota. Run the cell again. |
| `429` or quota errors | You hit the free-tier rate limit. Wait a minute or use another key. |
| Slow responses | Move a Flash-Lite model to the top of `PREFERRED`. Check your internet connection. |
| Quiz or mind map shows a JSON error | Click the button again; the model occasionally returns an unusable reply. |
| PDF says "No selectable text found" | The PDF is scanned. Use a text-based PDF (OCR is not supported yet). |
| Audio is missing | gTTS needs internet access to Google's speech service. The text is still shown. |
| Link stopped working | The Colab session ended or the 72-hour link expired. Run the cell again for a new link. |
| `pip` dependency error | Restart the Colab runtime, then run the cell again. |

---

## ⚠️ Limitations

- AI can **hallucinate**. Verify important answers against your course material.
- The PDF tool reads only the first 40,000 characters and cannot read scanned documents.
- No login, database, progress tracking or permanent logs. Nothing is saved after the session.
- The public share link can be used by anyone who has it, and that use consumes **your** API quota. Do not share it widely.
- Uploaded files and prompts are sent to Google's servers. Do not upload confidential documents.
- Text-to-speech supports one English voice.
- Quiz questions are AI-generated and not reviewed by a teacher.

---

## 🔭 Future Scope

- Full RAG with embeddings and a vector store (e.g. FAISS) for many or long documents
- OCR for scanned PDFs and photographed notes
- User accounts, saved progress and spaced-repetition revision
- A privacy-friendly audit log
- Flashcards, a study planner and weak-topic detection
- More languages, voice input and multiple voices
- Systematic evaluation with real students

---

## 🌍 Sustainable Development Goal

Supports **SDG 4: Quality Education** by offering affordable, personalised, tutor-style help to any learner with a browser.

---

## 📚 Key References

- Vaswani et al., *Attention Is All You Need*, NeurIPS 2017
- Lewis et al., *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*, NeurIPS 2020
- Gemini Team, Google, *Gemini: A Family of Highly Capable Multimodal Models*, 2023
- Wei et al., *Chain-of-Thought Prompting Elicits Reasoning in Large Language Models*, NeurIPS 2022
- Bloom, *The 2 Sigma Problem*, Educational Researcher, 1984
- Roediger and Karpicke, *Test-Enhanced Learning*, Psychological Science, 2006

---

## 👥 Authors

- Khushi Shukla (D20A, Roll no. 29)

Subject Teacher: Dipti Karani · Department of Information Technology, VESIT

---

## 🙏 Acknowledgements

Google Gemini API, Gradio, PyMuPDF, gTTS, Pillow and Google Colab.
