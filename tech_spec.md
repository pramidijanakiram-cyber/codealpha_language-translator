# Technical Specification: Smart Transliteration & Validation Layer

## 1. Overview
This specification details the implementation of a "Smart Transliteration" engine designed to handle romanized input (e.g., typing "konnichiwa" for Japanese) for any target language, alongside a validation layer that provides word-level breakdowns and spelling corrections.

## 2. Core Features

### 2.1 Lazy API Validation (Transliteration Detection)
- **Trigger**: Activates when user input is ASCII/Latin script but the target language uses a non-Latin script (e.g., Japanese, Chinese, Arabic, Hindi).
- **Mechanism**: 
  1. Detect input script type vs. target script type.
  2. Perform a "test translation" via the primary API.
  3. **Back-Translation Check**: Translate the result back to the source language.
  4. **Confidence Score**: If the back-translation semantically matches the original intent (or if the API returns a direct character conversion), treat it as valid transliteration.
  5. **Fallback**: If confidence is low, treat as standard English text.

### 2.2 Shadow Text Display (Ruby/Furigana Style)
- **Visual**: Display the translated non-Latin text with the original romanized input shown as "shadow" text underneath.
- **Implementation**: Use HTML `<ruby>` tags for native browser support where possible, falling back to CSS-stacked spans for broader compatibility.
- **Format**: 
  ```html
  <ruby>
    こんにちは
    <rt>konnichiwa</rt>
  </ruby>
  ```

### 2.3 "Did You Mean?" Correction
- **Trigger**: When the translation API returns a generic error, an empty string, or a low-confidence score.
- **Mechanism**: 
  - Utilize the API's built-in suggestion field (if available, e.g., Google/MyMemory often return `did-you-mean` or similar metadata).
  - If unavailable, perform a secondary lightweight request to a dictionary API for fuzzy matching.
- **UI**: A subtle, non-intrusive block appearing above the translation result offering a clickable correction.

### 2.4 Word-Level Validation & Breakdown
- **Location**: Panel immediately below the main translation block.
- **Content**: 
  - Tokenizes source and target sentences.
  - Maps source words to target words.
  - Fetches dictionary definitions and synonyms for each target word.
- **Purpose**: Builds user trust by showing the "math" behind the translation.

## 3. Data Flow

1. **Input**: User types text (e.g., "arigatou").
2. **Detection**: System sees Target = Japanese, Input = ASCII.
3. **Primary Request**: Send "arigatou" to Translation API (Source: Auto, Target: ja).
4. **Validation**: 
   - API returns "ありがとう".
   - System flags this as a successful transliteration.
5. **Rendering**: 
   - Main Box: Shows "ありがとう" with "arigatou" as shadow text.
   - Breakdown Box: Splits "ありがとう" → "ari", "gatou" (conceptually) or whole word, fetches meaning ("Thank you"), shows synonyms.
6. **Error Handling**: If API returns garbage or empty:
   - Check for `alternative_translations` or `did_you_mean`.
   - Render suggestion chip: "Did you mean: [suggestion]?"

## 4. Technology Stack
- **Frontend**: HTML5, CSS3 (Flexbox/Grid), Vanilla JavaScript (ES6+).
- **APIs**: 
  - Primary: MyMemory / Google Translate (Unofficial)
  - Dictionary: Free Dictionary API / Jisho (for Japanese specific fallback if needed)
- **Storage**: `localStorage` for history (existing).
- **No External Libraries**: Pure vanilla JS to keep load times minimal; no heavy NLP libraries downloaded.

## 5. Edge Cases & Mitigation
- **False Positives**: English words that look like Romaji (e.g., "no"). 
  - *Mitigation*: Context analysis via API back-translation.
- **Ambiguous Romaji**: "Kani" (Crab vs. God).
  - *Mitigation*: Default to most common usage; allow user to click "Did you mean?" for alternatives.
- **API Rate Limits**: 
  - *Mitigation*: Debounce input; cache recent transliterations in memory.

## 6. Future Scalability
- Support for Pinyin (Chinese) and Konglish (Korean).
- User-contributed corrections to improve local heuristics over time.
