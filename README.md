# 🔍 Keyhole

**Keyhole** is an intelligent web content analyzer and crawler designed for investigators, researchers, and security analysts. It enables users to extract, summarize, and assess content from individual web sections, full pages, or entire domains using powerful AI-driven techniques.

---

## 🚀 Features

- **🧠 AI Summarization** — Powered by `facebook/bart-large-cnn` for concise and intelligent summaries.
- **🔬 Section, Page, or Domain Analysis** — Analyze a single section, a full page, or crawl multiple pages from the same domain.
- **📊 Common Word Frequency** — Identify key terms and frequently discussed topics.
- **💬 Sentiment Analysis** — Determine the tone of content using VADER Sentiment.
- **📚 Readability Score** — Measure text complexity (Flesch score and educational level).
- **📁 Configurable Word Exclusion** — Customize excluded terms with live editing and preset management.
- **📝 Report Exporting** — Automatically generates timestamped `.txt` reports.
- **🌐 Smart Domain Crawling** — Crawl and summarize up to 5 internal pages from a domain.

---

## 👥 Ideal For

- **Digital Investigators** analyzing suspicious domains or leaked content.
- **Cybersecurity Researchers** examining tone, threats, or patterns.
- **Academic & Industry Research Teams** clustering themes across pages.
- **SEO/OSINT Specialists** profiling web content for structure and sentiment.

---

## 🛠 Requirements

- Python 3.8 or higher

### Installation

```bash
pip install requests beautifulsoup4 transformers vaderSentiment textstat torch
```

---

## 🧪 Usage

Launch the app from your terminal or command prompt:

```bash
python keyhole.py
```

---

### 🕹 Interactive Flow

1. **Website Input**
   - Enter a URL (e.g., `en.wikipedia.org/wiki/OpenAI` or `https://cybercrime.gov/report`)

2. **Choose Analysis Mode**
   - `section` → Extract and summarize a specific section
   - `full` → Analyze the full content of the page
   - `domain` → Crawl and summarize up to 5 internal pages

3. **Word Exclusion Management (Optional)**
   - Add or remove terms
   - Save or load custom presets

4. **AI-Driven Output**
   - Top 10 word frequency breakdown
   - Summary generated with NLP
   - Sentiment analysis via VADER
   - Readability grade level + Flesch score

---

## 📂 Output

Reports are saved automatically to:

```
C:\Users\<YourUsername>\OneDrive\Documents\Section Report
```

### 📄 File Naming Convention

Each file is named with a timestamp and analysis type:

- `Section_Report_YYYYMMDD_HHMMSS.txt`
- `FullPage_Report_YYYYMMDD_HHMMSS.txt`
- `Domain_Report_YYYYMMDD_HHMMSS.txt`

---

## 🧠 Tips for Investigators & Researchers

- **Targeted Surveillance**: Use `domain` mode on suspected sources to uncover tone and content shifts.
- **Evidence Archiving**: Timestamped files make it easy to document and cite findings.
- **Keyword Focus**: Exclude generic words to focus on relevant technical or thematic terms.
- **Cross-Site Comparison**: Analyze similar sites to compare sentiment, clarity, or lexical patterns.

---

### 💬 Example Session

```text
Enter website address (with or without 'https://'): en.wikipedia.org/wiki/OpenAI
Extract a section, full page, or entire domain? (section/full/domain): section
Enter the section title to extract: History
Modify excluded words? (add/remove/save preset/load preset/none): load preset
Enter preset name: research
```

The report will be saved automatically in your documents folder.

---

## 🔮 Coming Soon

Keyhole is evolving to offer more investigative power:

### 🚧 In Development

- **Batch URL Input** — Analyze multiple URLs from a `.txt` or `.csv` file
- **Named Entity Recognition (NER)** — Extract people, places, and organizations
- **Keyword Watchlists** — Highlight sensitive or topic-specific keywords
- **Markdown / PDF Export** — Create publication-ready reports
- **Topic Modeling & Clustering** — Auto-detect and group thematic content
- **Interactive Report Viewer** — Navigate reports via CLI or Streamlit

### 🧪 Experimental Ideas

- **Fake vs Legitimate Tone Detection**
- **Regex-based Search Across Pages**
- **Offline/Isolated Mode for Secure Labs**
- **Snapshot Caching & Archival Tools**

---

Stay curious. Stay analytical. Use **Keyhole** to see what others miss. 🔍

