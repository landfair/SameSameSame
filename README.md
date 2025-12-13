# SameSameSame

**Homogenometry Dashboard** - A client-side text similarity analysis tool for detecting plagiarism and AI-generated content convergence.

## Overview

SameSameSame analyzes pairwise text similarity across a corpus to detect suspicious homogeneity patterns that may indicate plagiarism or AI-generated content. The tool runs entirely in your browser - no data is sent to any server, ensuring FERPA compliance.

## Key Features

- **21 Similarity Metrics** including:
  - Lexical similarity (Cosine, Jaccard, N-grams)
  - Statistical features (Sentence variance, word count, burstiness)
  - **AI-Powered Semantic Similarity** (Transformers.js)
  - **Syntactic Pattern Analysis** (POS tagging with Compromise.js)
  - **Function Word Distribution** (Stylometric analysis)
  - Named Entity Recognition, Fuzzy matching, and more

- **100% Client-Side Processing**
  - All analysis happens in your browser
  - No server uploads, no data transmission
  - FERPA compliant for educational use

- **Corpus-Internal Comparison**
  - Compares texts only within your dataset
  - No external benchmarks or reference corpora
  - Detects convergence patterns (the "beigeness" phenomenon)

- **Advanced NLP Libraries**
  - Transformers.js for semantic similarity
  - Compromise.js for POS tagging and syntactic analysis
  - All processing uses Web Workers for performance

## Usage

1. Open `index.html` in a modern web browser
2. Upload a CSV file with text responses (must have "Respondent" and response columns)
3. Configure similarity threshold and select metrics
4. Click "Analyze" to detect high-similarity pairs
5. Review results and investigate flagged pairs

## File Structure

```
SameSameSame/
├── index.html                    # Main application (single-file)
├── libs/
│   └── compromise.min.js        # NLP library for POS tagging
├── STYLE_METRICS_PLAN.md        # Implementation plan for stylistic metrics
├── .gitignore                    # Git ignore file
└── README.md                     # This file
```

## Technical Details

- **Dependencies**:
  - Compromise.js (local, 322 KB) - POS tagging
  - Transformers.js (CDN) - Semantic similarity
- **Browser Requirements**: Modern browser with ES6 module support
- **Performance**: Uses Web Workers for parallel processing

## Privacy & Compliance

- **FERPA Compliant**: All processing is client-side
- **No External APIs**: No data sent to any server
- **No Tracking**: No analytics or telemetry
- **Open Source**: Inspect the code yourself

## Research Background

This tool is based on research showing that LLMs produce homogeneous outputs when given identical prompts (the "beigeness" phenomenon). By measuring pairwise similarity across multiple dimensions, the tool can detect:

- Plagiarism between students
- AI-generated content convergence
- Suspicious stylistic homogeneity
- Copy-paste patterns

## License

[Add your license here]

## Citation

If you use this tool in research, please cite [citation info]

## Contact

[Your contact information]
