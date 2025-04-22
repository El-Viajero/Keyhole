import requests
from bs4 import BeautifulSoup
from collections import Counter
import re
import logging
import os
import json
from datetime import datetime
from typing import List, Tuple, Optional, Set
from urllib.parse import urljoin, urlparse
from transformers import pipeline
from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer
import textstat

# Configure logging
logging.basicConfig(level=logging.INFO, format='%(levelname)s: %(message)s')

CONFIG_DIR = "config"
EXCLUDE_WORDS_FILE = os.path.join(CONFIG_DIR, "exclude_words.json")
PRESETS_DIR = os.path.join(CONFIG_DIR, "presets")
DOCUMENTS_FOLDER = os.path.join(os.path.expanduser("~"), "OneDrive", "Documents", "Section Report")

summarizer = pipeline("summarization", model="facebook/bart-large-cnn")
sentiment_analyzer = SentimentIntensityAnalyzer()

def init_config():
    os.makedirs(CONFIG_DIR, exist_ok=True)
    os.makedirs(PRESETS_DIR, exist_ok=True)
    if not os.path.exists(EXCLUDE_WORDS_FILE):
        with open(EXCLUDE_WORDS_FILE, 'w') as f:
            json.dump([], f)

def load_excluded_words() -> Set[str]:
    with open(EXCLUDE_WORDS_FILE, 'r') as f:
        return set(json.load(f))

def save_excluded_words(words: Set[str]):
    with open(EXCLUDE_WORDS_FILE, 'w') as f:
        json.dump(list(words), f)

def save_preset(name: str, words: Set[str]):
    preset_path = os.path.join(PRESETS_DIR, f"{name}.json")
    with open(preset_path, 'w') as f:
        json.dump(list(words), f)

def load_preset(name: str) -> Optional[Set[str]]:
    preset_path = os.path.join(PRESETS_DIR, f"{name}.json")
    if os.path.exists(preset_path):
        with open(preset_path, 'r') as f:
            return set(json.load(f))
    return None

def fetch_page(url: str, timeout: int = 8) -> Optional[str]:
    headers = {
        "User-Agent": (
            "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 "
            "(KHTML, like Gecko) Chrome/120.0 Safari/537.36"
        )
    }
    try:
        response = requests.get(url, headers=headers, timeout=timeout)
        response.raise_for_status()
        return response.text
    except requests.RequestException as e:
        logging.error(f"Failed to fetch {url}: {e}")
        return None

def extract_section_text(html_content: str, section_title: str) -> Optional[str]:
    soup = BeautifulSoup(html_content, 'html.parser')

    def normalize(text: str) -> str:
        return re.sub(r'\W+', '', text).lower()

    normalized_input = normalize(section_title)
    headers = soup.find_all(re.compile(r'h[1-6]'))
    target_header = None

    for header in headers:
        if normalize(header.get_text()) == normalized_input:
            target_header = header
            break

    if not target_header:
        logging.warning(f'Section "{section_title}" not found.')
        return None

    content = []
    for sibling in target_header.find_next_siblings():
        if sibling.name and re.match(r'h[1-6]', sibling.name):
            break
        if sibling.name in ['p', 'div', 'span', 'li']:
            content.append(sibling.get_text(separator=' ', strip=True))

    return ' '.join(content)

def extract_full_page_text(html_content: str) -> str:
    soup = BeautifulSoup(html_content, 'html.parser')
    return soup.get_text(separator=' ', strip=True)

def summarize_text(text: str, max_len=130, min_len=30) -> str:
    text = text.strip().replace("\n", " ")
    if len(text.split()) > 500:
        text = ' '.join(text.split()[:500])
    summary = summarizer(text, max_length=max_len, min_length=min_len, do_sample=False)
    return summary[0]['summary_text']

def analyze_sentiment(text: str) -> str:
    scores = sentiment_analyzer.polarity_scores(text)
    compound = scores['compound']
    sentiment = "Positive" if compound >= 0.05 else "Negative" if compound <= -0.05 else "Neutral"
    return f"{sentiment} (compound score: {compound})"

def readability_score(text: str) -> str:
    grade = textstat.text_standard(text, float_output=False)
    flesch = textstat.flesch_reading_ease(text)
    return f"Grade Level: {grade} (Flesch Score: {flesch})"

def get_common_words(text: str, num_words: int = 10, exclude_words: Optional[Set[str]] = None) -> List[Tuple[str, int]]:
    words = re.findall(r'\b[a-zA-Z]\w*\b', text.lower())
    if exclude_words:
        words = [word for word in words if word not in exclude_words]
    return Counter(words).most_common(num_words)

def export_domain_report(domain_label: str, reports: List[str]):
    os.makedirs(DOCUMENTS_FOLDER, exist_ok=True)
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    filename = f"Domain_Report_{timestamp}.txt"
    file_path = os.path.join(DOCUMENTS_FOLDER, filename)

    with open(file_path, 'w', encoding='utf-8') as f:
        f.write(f"Domain Summary Report for {domain_label}\n")
        f.write("=" * 60 + "\n\n")
        for report in reports:
            f.write(report)
            f.write("\n" + "-" * 60 + "\n\n")

    logging.info(f"\nFull domain report saved to: {file_path}")

def crawl_domain(start_url: str, exclude_words: Set[str], max_pages: int = 5):
    visited = set()
    to_visit = [start_url]
    parsed_start = urlparse(start_url).netloc
    domain_reports = []

    while to_visit and len(visited) < max_pages:
        url = to_visit.pop(0)
        if url in visited:
            continue
        visited.add(url)

        html = fetch_page(url)
        if not html:
            continue

        content = extract_full_page_text(html)
        common_words = get_common_words(content, exclude_words=exclude_words)
        summary = summarize_text(content)
        sentiment = analyze_sentiment(content)
        readability = readability_score(content)

        report = f"URL: {url}\n\n"
        report += "Top 10 Common Words:\n"
        for i, (word, count) in enumerate(common_words, 1):
            report += f"{i}. {word.capitalize():<15} {count} occurrences\n"
        report += f"\nSummary:\n{summary}\n"
        report += f"\nSentiment:\n{sentiment}\n"
        report += f"\nReadability:\n{readability}\n"
        domain_reports.append(report)

        soup = BeautifulSoup(html, 'html.parser')
        for link_tag in soup.find_all('a', href=True):
            href = link_tag['href']
            absolute_url = urljoin(url, href)
            parsed_href = urlparse(absolute_url)
            if parsed_href.netloc == parsed_start and absolute_url not in visited and absolute_url.startswith(start_url):
                to_visit.append(absolute_url)

    export_domain_report(parsed_start, domain_reports)

def main():
    init_config()
    site = input("Enter website address (with or without 'https://'): ").strip()
    url = site if site.startswith("http") else f"https://{site}"
    mode = input("Extract a section, full page, or entire domain? (section/full/domain): ").strip().lower()
    exclude_words = load_excluded_words()

    action = input("Modify excluded words? (add/remove/save preset/load preset/none): ").strip().lower()
    if action == "add":
        new_word = input("Enter word to add: ").strip().lower()
        exclude_words.add(new_word)
        save_excluded_words(exclude_words)
    elif action == "remove":
        rem_word = input("Enter word to remove: ").strip().lower()
        exclude_words.discard(rem_word)
        save_excluded_words(exclude_words)
    elif action == "save preset":
        preset_name = input("Enter preset name: ").strip()
        save_preset(preset_name, exclude_words)
    elif action == "load preset":
        preset_name = input("Enter preset name: ").strip()
        preset = load_preset(preset_name)
        if preset:
            exclude_words = preset
            save_excluded_words(exclude_words)

    if mode == "domain":
        crawl_domain(url, exclude_words)
    else:
        html = fetch_page(url)
        if not html:
            logging.error("Unable to retrieve webpage content.")
            return

        if mode == "section":
            section = input("Enter the section title to extract: ").strip()
            content = extract_section_text(html, section)
            if not content:
                logging.error("Section not found or empty.")
                return
        else:
            content = extract_full_page_text(html)

        common_words = get_common_words(content, exclude_words=exclude_words)
        summary = summarize_text(content)
        sentiment = analyze_sentiment(content)
        readability = readability_score(content)

        output = f"Top 10 Common Words:\n"
        for i, (word, count) in enumerate(common_words, 1):
            output += f"{i}. {word.capitalize():<15} {count} occurrences\n"
        output += f"\nSummary:\n{summary}\n"
        output += f"\nSentiment:\n{sentiment}\n"
        output += f"\nReadability:\n{readability}\n"

        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        filename = f"{'FullPage' if mode == 'full' else 'Section'}_Report_{timestamp}.txt"
        os.makedirs(DOCUMENTS_FOLDER, exist_ok=True)
        filepath = os.path.join(DOCUMENTS_FOLDER, filename)

        with open(filepath, 'w', encoding='utf-8') as f:
            f.write(f"{url}\n\n{output}")

        logging.info(f"Report saved to: {filepath}")

if __name__ == "__main__":
    main()
