# 🎥 AI YouTube Video Summarizer

An AI-powered tool that extracts the transcript from any YouTube video and generates a structured, visual summary dashboard — including key points, insights, topics, and auto-generated Q&A — built and run entirely in Google Colab using the **Gemini API**.

Just provide a video ID, and the notebook fetches the transcript, sends it to Gemini for analysis, and renders a clean visual dashboard covering everything from a quick summary to a deep-dive explanation.

---

## 🚀 Features

- 📝 **Transcript Extraction** — Pulls the full transcript of any YouTube video using `youtube-transcript-api`
- 🤖 **AI-Powered Analysis** — Uses Google's Gemini API (`gemini-3.5-flash`) to break down the video content
- 📊 **Visual Dashboard** — Renders results as a styled HTML dashboard with:
  - A short **Summary**
  - **Topic tags** for quick scanning
  - A **Detailed Explanation** of the full video content
  - **Key Points** (bullet list)
  - **Important Insights** (bullet list)
  - **5 auto-generated Q&A pairs** based on the video content
- ❓ **Custom Q&A** — Ask your own question about the video and get an answer grounded in the transcript

---

## 🖼️ Example Output

The dashboard renders as a multi-card grid layout:

| Section | Description |
|---------|-------------|
| 📌 Summary | 2–3 sentence overview of the video |
| 🏷️ Topics Covered | Tag chips for quick context (e.g. JavaScript, DOM Manipulation) |
| 📖 Detailed Explanation | Longer paragraph covering the full video content |
| 🔑 Key Points | Bullet list of core takeaways |
| 💡 Important Insights | Bullet list of deeper/less obvious takeaways |
| ❓ Questions & Answers | 5 auto-generated Q&A pairs based on the transcript |

---

## 🛠️ Tech Stack

| Component           | Tool/Library                              |
|-----------------------|--------------------------------------------|
| Environment           | Google Colab                              |
| Transcript Extraction | `youtube-transcript-api`                  |
| AI Model              | Google Gemini API (`google-genai` SDK) — `gemini-3.5-flash` |
| Dashboard Rendering   | `IPython.display.HTML`                    |
| Language              | Python 3                                  |

---

## 📋 How It Works

1. **Install dependencies**
```python
   !pip install google-genai youtube-transcript-api
   !pip install --upgrade youtube-transcript-api
```

2. **Set up your Gemini API key**
```python
   from google.colab import userdata
   GOOGLE_API_KEY = userdata.get('GOOGLE_API_KEY')

   from google import genai
   client = genai.Client(api_key=GOOGLE_API_KEY)
```

3. **Fetch the video transcript**
```python
   from youtube_transcript_api import YouTubeTranscriptApi

   def get_transcript(video_id, languages=['en']):
       transcript = YouTubeTranscriptApi().fetch(video_id, languages=languages)
       text = " ".join([item.text for item in transcript])
       return text
```

4. **Analyze the transcript with Gemini**
```python
   import json, re

   def analyze_video(transcript):
       prompt = f"""
   Analyze this YouTube video transcript and return a structured breakdown.

   Transcript:
   {transcript}

   Return in JSON format with these exact keys:
   - summary (2-3 sentence short summary)
   - detailed_explanation (a longer paragraph explaining the video's content)
   - key_points (list of short bullet points)
   - insights (list of short bullet points, important takeaways)
   - qna (list of objects, each with "question" and "answer" keys — exactly 5 items)
   - topics (list of up to 6 short topic/keyword tags for this video)

   Rules:
   - Output ONLY JSON
   - No extra text, no markdown code fences
   """
       response = client.models.generate_content(
           model="gemini-3.5-flash",
           contents=prompt
       )
       return response.text
```

5. **Render the results as a dashboard**
   The final cell parses the JSON response and displays a styled HTML dashboard with all sections above using `IPython.display.HTML`.

6. **(Optional) Ask a custom question**
```python
   def ask_question(transcript, question):
       prompt = f"""
   Answer the question based on this video transcript:

   Transcript:
   {transcript}

   Question:
   {question}
   """
       response = client.models.generate_content(
           model="gemini-3.5-flash",
           contents=prompt
       )
       return response.text

   # Example
   print(ask_question(transcript, "What is the main topic?"))
```

---

## ⚙️ Setup & Requirements

- A **Google Colab** account
- A **Gemini API key** ([Google AI Studio](https://aistudio.google.com/app/apikey))
- Store your key in Colab's **Secrets** manager as `GOOGLE_API_KEY` (🔑 icon in the left sidebar), so it can be accessed via:
```python
  from google.colab import userdata
  userdata.get('GOOGLE_API_KEY')
```

### Run it yourself
1. Open `AI_YouTube_Video_Summarizer.ipynb` in Google Colab
2. Add your `GOOGLE_API_KEY` to Colab Secrets
3. Run all cells in order
4. Replace `video_id` with the ID of any YouTube video (the part after `v=` in the URL)
5. View your video summary dashboard at the bottom

---

## 📌 Notes & Limitations

- The video must have captions/subtitles available (auto-generated or manual) — `youtube-transcript-api` cannot extract text from videos with no captions.
- Gemini's free-tier API can occasionally return `503 UNAVAILABLE` errors during high demand. If this happens, wait a few moments and re-run the analysis cell, or implement a retry loop with a fallback model.
- Longer videos produce longer transcripts, which may take more time to process and could hit token limits on very long content (e.g. multi-hour videos).
- Results depend on the AI model's interpretation — the Q&A and insights are AI-generated, not a substitute for watching the video for critical use cases.

---

## 🗺️ Roadmap / Ideas for Improvement

- [ ] Add retry logic with exponential backoff for API overload errors
- [ ] Auto-detect video language instead of hardcoding `languages=['en']`
- [ ] Add timestamp links so key points jump to the relevant part of the video
- [ ] Turn this into a standalone web app (e.g. with Streamlit or Gradio) for use outside Colab
- [ ] Support batch summarization of an entire playlist
- [ ] Add an interactive chat interface for the custom Q&A feature

---

## 📄 License

This project is open source and available for personal or educational use. Add a license of your choice (e.g. MIT) if publishing publicly.
