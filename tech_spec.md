# Translator Technical Specification

## Overview
A lightweight, single-file HTML translator with smart validation, auto-translate, and transliteration support for 23+ languages.

## Architecture

### Core Components

1. **Translation Engine**
   - Primary: MyMemory API (crowdsourced translations)
   - Fallback: Google Translate unofficial API
   - Consensus mechanism: Compares results from both APIs
   - Spam detection: Structural pattern matching (no profanity filters)

2. **Auto-Translate System**
   - Debounced input monitoring (800ms delay)
   - Silent mode for background updates
   - Manual trigger via button or Ctrl/Cmd+Enter

3. **Transliteration Detection**
   - Detects Latin script input for non-Latin target languages
   - Supported: Japanese (romaji), Chinese (pinyin), Arabic (chat alphabet), Hindi, Korean, Russian, Greek, Thai, Hebrew
   - Lazy API validation: Tests translation then verifies output script
   - Ruby tag display: Shows original input as shadow text beneath translated characters

4. **Validation Layer**
   - Multi-source consensus (MyMemory + Google)
   - Similarity scoring using Levenshtein distance
   - Spam pattern detection (API pollution protection)
   - Sanity checks: length ratios, character set validation

5. **Word Breakdown Panel**
   - Tokenizes source and target text
   - Dictionary lookup (local + Free Dictionary API)
   - Shows meanings and synonyms per word
   - Validation badge with loading states

6. **Translation History**
   - localStorage persistence (up to 50 entries)
   - Star/favorite system
   - Click-to-load functionality
   - Relative timestamps

## Key Functions

### `translate(silent = false)`
Main translation function with optional silent mode for auto-translate.
- Queries both MyMemory and Google APIs
- Applies consensus logic
- Runs spam detection
- Handles transliteration display
- Updates history (only if not silent)

### `detectTransliteration(text, targetLang)`
Checks if Latin input should be converted to non-Latin script.
- Validates input is ASCII/Latin
- Confirms target is non-Latin language
- Tests via API and back-translates
- Returns confidence level

### `isLikelySpam(text)`
Detects API pollution without content filtering.
- Pattern-based detection only
- Checks for: phone numbers, parenthetical spam, numeric-only responses
- No profanity blocking (preserves exact meaning)

### `calculateSimilarity(str1, str2)`
Levenshtein-based string comparison for consensus checking.
- Returns similarity score 0.0-1.0
- Used to detect divergent API results

### `renderShadowText(original, translated, targetLang)`
Creates ruby/furigana display for transliterated text.
- Uses HTML `<ruby>` tags for native browser support
- Falls back to flexbox layout if needed

## Data Flow

```
User Input → Debounce (800ms) → translate()
    ↓
[Transliteration Check] → detectTransliteration()
    ↓
[Parallel API Calls]
    ├─ MyMemory API
    └─ Google API
    ↓
[Consensus Engine]
    ├─ Compare results (similarity score)
    ├─ Run spam detection
    └─ Select best result
    ↓
[Display Logic]
    ├─ Transliteration? → renderShadowText()
    └─ Normal → plain text
    ↓
[Post-Processing]
    ├─ Add to history (if manual)
    ├─ Render word breakdown
    └─ Update UI badges
```

## API Endpoints

1. **MyMemory**: `https://api.mymemory.translated.net/get?q={text}&langpair={from}|{to}`
2. **Google Fallback**: `https://translate.googleapis.com/translate_a/single?client=gtx&sl={from}&tl={to}&dt=t&q={text}`
3. **Dictionary**: `https://api.dictionaryapi.dev/api/v2/entries/en/{word}`

## Supported Languages (23)

English, Spanish, French, German, Italian, Portuguese, Dutch, Russian, Japanese, Korean, Chinese (Simplified), Arabic, Hindi, Tamil, Telugu, Bengali, Turkish, Vietnamese, Polish, Swedish, Greek, Hebrew, Thai, Indonesian

## Non-Latin Script Languages (for transliteration)

Japanese, Korean, Chinese, Arabic, Hindi, Tamil, Telugu, Bengali, Thai, Hebrew, Russian, Greek

## Error Handling

- Network errors: User-friendly messages about connection/ad-blockers
- API failures: Automatic fallback to secondary service
- Spam detection: Silently switches to cleaner API result
- No voice installed: Falls back to Google's online TTS audio

## Performance Optimizations

- Debounced auto-translate prevents API flooding
- Local dictionary cache reduces external calls
- localStorage for history (no database needed)
- Single HTML file architecture (no build step)

## Security Considerations

- No user data sent to servers except translation text
- localStorage used only for history (user-controlled)
- No third-party tracking or analytics
- CSP-friendly (no inline eval)

## Browser Compatibility

- Modern browsers (ES6+ support required)
- Speech synthesis: Graceful fallback to audio playback
- Ruby tags: Native support in Chrome/Firefox/Safari/Edge
