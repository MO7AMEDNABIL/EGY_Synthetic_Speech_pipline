# EGY_Synthetic_Speech_pipline
A complete end-to-end pipeline for generating, reviewing, and exporting a synthetic Egyptian Arabic speech dataset for Speech-to-Text (STT) model training.

Project Overview

This project builds a practical data generation pipeline focused on Egyptian Arabic a dialect that presents major challenges for off-the-shelf ASR/STT systems due to:

-Heavy dialect variation
-Informal pronunciation
-Code switching with English
-Non standard spelling
-difficult to find open source llm write egyptian arabic correctly

## The pipeline automatically:

Generates Egyptian Arabic prompts using an LLM
Synthesizes speech using TTS
Provides a human review interface for filtering bad samples
Exports approved data into a training-ready dataset format

# LLM Prompt Generation → Prompt list → TTS → Audio files → Review UI → Export JSONL

## The system is designed to be:

Runnable end-to-end
Modular and configurable
Observable through manifests/status endpoints
Resumable and scalable for long-running synthesis jobs

===================================================================================
#Technologies Used
## Backend

| Tool         | Purpose                            |

| Flask        | API server                         |
| Coqui TTS    | Speech synthesis                   |
| XTTS-v2      | Multilingual voice cloning support arabic|
| Transformers | LLM prompt generation              |
| Nile-Chat-4B | Egyptian Arabic text generation    |
| pyngrok      | Public tunneling for Colab backend |
| threading    | Background synthesis jobs          |

## Frontend

| Tool                | Purpose                  |

| HTML/CSS/JavaScript | Interactive review UI    |
| Custom audio player | Review synthesized audio |
| Queue management    | Approve/reject workflow  |

=======================================================================================
# Why Egyptian Arabic is Challenging

Egyptian Arabic differs significantly from Modern Standard Arabic

Challenges include:

1. Non-standard Writing


عايز اروح -> حرف "ا" بدل "أ"
عايز أروح -> حرف "أ" بدل "ا"
عاوز أروح-> حرف "ا" بدل "ي"
### بس كلمهم في لاخر نفس المعني بس لهاجت محافظات مختلفه
2. Code Switching
mix English words:

ال wifi بطيء
هطلب uber
رقم موبيلي 10 ZERO 
3. Informal Pronunciation

لست اعرف → مش عارف
### التنوين هيفرق مع كل كلمه و محتاج دقه عاليه جدا

4. Sparse Public Datasets

Compared to English ASR datasets, Egyptian Arabic has limited high-quality labeled speech corpora.

======================================================================================
# Prompt Generation
Model Used From Hugging face

## MBZUAI-Paris/Nile-Chat-4B

Chosen because it performs significantly better on Egyptian Arabic dialect generation than many general-purpose LLMs.

Prompting Strategy

# The prompt strongly constrains the model:

Egyptian Arabic only
Cairo dialect
No MSA
No explanations
Spoken language only
EX:
- Use ONLY spoken Egyptian Arabic
- Do NOT use Modern Standard Arabic
- Output ONLY 10 sentences

# Prompt Cleaning

LLM outputs are cleaned using regex rules to remove:
Numbering
Formatting artifacts
Empty lines
EX:

line = re.sub(r"^\s*[0-9٠-٩]+\s*[.\-]?\s*", "", line)


# TTS Synthesis
Model Used
"tts_models/multilingual/multi-dataset/xtts_v2"

Because :
Strong multilingual support
Arabic language support
Voice cloning capability
High audio quality
Easy deployment in Python
Free & open source 

# Long-running Job Handling

The pipeline handles this using:
Background threads
Status polling
Incremental file tracking
Manifest generation

EX:
t = threading.Thread(target=run_generation, daemon=True)

====================================================================================
# The most important part why Google Colab + ngrok

The project was developed using:
Google Colab
NVIDIA T4 GPU
ngrok tunnel

Reasoning:
GPU Acceleration
XTTS-v2 is computationally expensive on CPU.
Using a T4 GPU significantly improved:

Synthesis speed
Batch throughput
End-to-end iteration time
Public Backend Access

Since Colab runs remotely, ngrok was used to expose the Flask backend publicly so the frontend could communicate with it.

Example:
### public_url = ngrok.connect(port)

=========================================================================================
# Review System

A major focus of this project is human quality control.

Synthetic data can easily introduce:
Mispronunciations
Wrong dialect
Audio glitches
Hallucinated speech
Alignment issues

The frontend review interface allows users to:
Listen to audio
Read transcript text
Approve samples
Reject bad samples
Add rejection comments

## Frontend Features
Backend Connection
Connect to the ngrok backend dynamically.
Audio Generation
Trigger synthesis jobs asynchronously.
Review Queue

Displays:
Pending samples
Approved samples
Rejected samples
Audio Player

Custom audio player with:

Play/pause
Seek bar
Timeline
Quality Filtering
Human reviewers can reject problematic samples before export

==========================================================================================

# And the last part Training ready Output Format

Approved samples are exported as:
STT_HF.jsonl

EX:
{"audio":"audio/0.wav","text":"أنا رايح الجامعة دلوقتي"}
{"audio":"audio/1.wav","text":"عايز أطلب كشري"}

## Why JSONL Format

Streaming friendly
Easy to parse
Compatible with Hugging Face datasets
Widely used in ML pipelines
Simple to extend with metadata later




##### Dataset Structure
dataset/
│
├── audio/
│   ├── 0.wav
│   ├── 1.wav
│   └── ...
│
└── metadata.jsonl

=======================================================================================

# Intermediate Artifacts

| Artifact         | Purpose                   |
| ---------------- | ------------------------- |
| prompts.json     | Generated text prompts    |
| manifest.json    | Generated audio mapping   |
| status endpoint  | Real-time synthesis state |
| review decisions | Human QA filtering        |

# Reliability Features

Thread-safe TTS Access
tts_lock = threading.Lock()

Prevents concurrent synthesis corruption.

# "Background Processing"

Long synthesis jobs do not block the API.

# "Status Tracking"
Generation_status

Tracks:
Running state
Errors
Completed files

# "Error Handling"
Exceptions are captured and surfaced to the frontend.

====================================================================================

# Synthetic Data Risks

Synthetic speech datasets can negatively affect downstreammodels if quality is poor.

| Risk                         | Impact                       |
| ---------------------------- | ---------------------------- |
| Overly clean audio           | Poor real-world robustness   |
| Single speaker bias          | Weak speaker generalization  |
| Repeated sentence structures | Reduced linguistic diversity |
| TTS pronunciation mistakes   | Incorrect ASR learning       |

======================================================================================

# Mitigations Used

Human Review
Manual filtering removes obvious failures.
Dialect-focused Prompting
Prompts intentionally avoid MSA.
Diverse Sentence Topics

Prompts include:
Transportation
Food
Internet
Daily conversation
Navigation
Technology


# Observed Quality Issues

During testing, several issues were observed:

1. Occasional MSA Leakage: Some generated prompts became partially formal.
2. English Word Pronunciation:Words like-> wifi & uber

sometimes sounded unnatural.

3. TTS Mispronunciation

Rare pronunciation errors occurred with slang expressions.

4. Repetitive Structures

LLM generations occasionally repeated sentence patterns.

========================================================================================

# Trade offs

| Decision     | Benefit         | Trade-off          |

| XTTS-v2      | High quality    | Slow generation    |
| Nile-Chat-4B | Better dialect  | Larger GPU memory  |
| Human review | Better quality  | More manual effort |
| Google Colab | Free GPU access | Temporary runtime  |

--------------------------------------------------------------------------------------

## Future Improvements
Multi-speaker synthesis
Automatic quality scoring
Forced alignment validation
Phoneme diversity analysis
Speaker embeddings

========================================================================================

# Project Goals Achieved

✅ Egyptian Arabic prompt generation
✅ TTS audio synthesis
✅ Human review workflow
✅ Training-ready dataset export
✅ Async/background processing
✅ Observable intermediate artifacts
✅ Quality-aware pipeline design



#                                         Conclusion

This project demonstrates a practical pipeline for generating synthetic Egyptian Arabic speech datasets with a strong focus on:

Data quality
Dialect awareness
Human validation
Reliability
Real-world usability

Rather than focusing only on generation volume, the pipeline emphasizes careful filtering and structured dataset preparation — both critical for building useful downstream STT systems.
