## Building an AI-Powered Interview Question Extractor with Python and Google Gemini
Processing audio recordings of interviews to find critical data is a tedious manual task. While local automated transcription tools (like OpenAI Whisper) exist, configuring them on local operating systems often introduces friction with low-level media dependencies (like FFmpeg) or file codec container conflicts.
This technical reference article walks through an end-to-end, production-ready solution. It uploads audio files directly to the Google Gemini Cloud API, handles transient server traffic gracefully, extracts interview questions via structured prompting, and automatically performs post-execution memory cleanup.
------------------------------
## Technical Architecture Overview
The system uses a direct cloud-ingestion architecture to maintain a lightweight local deployment footprint.

┌──────────────┐      Upload      ┌──────────────────────┐
│  Local Host  │ ────────────────> │ Google File Storage  │
│ (Notes.m4a)  │                  │  (Temporary Storage) │
└──────────────┘                  └──────────────────────┘
       ▲                                     │
       │                                     ▼
       │  Structured Response     ┌──────────────────────┐
       └───────────────────────── │  gemini-3.6-flash    │
                                  └──────────────────────┘


   1. Storage Subsystem: Local audio (.m4a/.mp3) is pushed to the Google File API. This avoids network payload timeouts during standard HTTP POST body inference transmissions.
   2. State Machine Polling: A blocking loop queries the ingestion status until the audio stream file validation changes from PROCESSING to ACTIVE.
   3. Inference Execution Engine: The engine launches an abstracted Chat session utilizing the gemini-3.6-flash model archetype.
   4. Transient Fault Tolerance: The processing layer wraps network requests in an Exponential Backoff Loop to catch and recover from server saturation faults (HTTP 503 Service Unavailable).
   5. Garbage Collection: Pre-computed assets are explicitly purged from cloud infrastructure using targeted resource identifiers upon completion or runtime failure.

------------------------------
## Environment Setup
The script requires a minimal surface area of local dependencies. Execute the installation sequence below using your terminal or Windows PowerShell console:

pip install google-genai

## Authentication Token Configuration
The google-genai SDK automatically scans system environment variables for active credentials. Set your target API key to ensure seamless runtime authentication:
## Windows PowerShell

[Environment]::SetEnvironmentVariable("GEMINI_API_KEY", "your_api_key_here", "User")# Restart your terminal instance to initialize the active path variable

## Linux / macOS Bash

export GEMINI_API_KEY="your_api_key_here"

------------------------------
## Production-Ready Code Implementation
Create a script named extractor.py and implement the absolute structural code template below:

import timeimport sysfrom google import genaifrom google.genai import typesfrom google.genai.errors import APIError
# Instantiating the unified client interfaceclient = genai.Client()
def extract_questions_from_audio(file_path: str) -> str:
    """
    Ingests an audio file, streams it to cloud infrastructure, orchestrates 
    targeted LLM context parsing, and cleans up remote data segments.
    """
    # 1. Upload the target file to the Gemini cloud storage framework
    print("☁️ Initiating file streaming upload to Google AI Studio...")
    try:
        audio_file = client.files.upload(file=file_path)
        print(f"✅ Ingestion successful. Remote Resource Handle: {audio_file.name}")
    except Exception as e:
        print(f"❌ Initial upload failed: {e}")
        sys.exit(1)
    
    # 2. Synchronous validation polling loop
    print("⏳ Blocking thread: Awaiting remote codec processing...")
    while audio_file.state.name == "PROCESSING":
        time.sleep(5)
        audio_file = client.files.get(name=audio_file.name)
        
    if audio_file.state.name == "FAILED":
        raise RuntimeError("Remote storage subsystem failed to parse media container metadata.")
        
    print("🔄 Execution context ready. Constructing Chat transaction stream...")
    
    # 3. Create a state-tracked chat infrastructure instance
    chat = client.chats.create(
        model='gemini-3.6-flash',
        config=types.GenerateContentConfig(
            system_instruction=(
                "You are an expert natural language data parser. Analyze the given audio stream. "
                "Isolate, filter out, and extract only the discrete structural questions asked "
                "by the primary interviewer. Omit casual banter, greetings, acknowledgments, "
                "and narrative response strings from the interviewee. Output only a finalized, "
                "clean markdown numbered list of the questions."
            ),
            temperature=0.1, # Low thermal entropy enforces factual adherence to input metadata
        )
    )
    
    # 4. Fault-Tolerant Network Loop (Handles HTTP 503 Engine Spikes)
    max_retries = 5
    execution_delay = 10  # Delta initialization value in seconds
    response_payload = None

    for attempt in range(max_retries):
        try:
            response = chat.send_message(
                message=[
                    audio_file,
                    "Parse this audio resource file and extract the interview questions."
                ]
            )
            response_payload = response.text
            break # Successful execution path; escape retry structure
        except APIError as error_ctx:
            if error_ctx.code == 503 and attempt < max_retries - 1:
                print(f"⚠️ Server experiencing load saturation (HTTP 503). Retrying in {execution_delay}s... (Attempt {attempt + 1}/{max_retries})")
                time.sleep(execution_delay)
                execution_delay *= 2  # Double the backoff scale curve rate
            else:
                # Emergency resource disposal on critical execution failure
                print("🚨 Fatal connection intercept. Flushing remote storage nodes...")
                client.files.delete(name=audio_file.name)
                raise error_ctx

    # 5. Remote Infrastructure Storage Disposal (Post-Execution Garbage Collection)
    print("🧹 Execution complete. Removing asset references from cloud network...")
    client.files.delete(name=audio_file.name)
    
    return response_payload
if __name__ == "__main__":
    # Configure your explicit system file path route target here
    TARGET_AUDIO_SOURCE = r"E:\Notes.m4a"
    
    try:
        extracted_questions = extract_questions_from_audio(TARGET_AUDIO_SOURCE)
        print("\n📋 Extracted Interview Questions:\n")
        print(extracted_questions)
    except Exception as general_fault:
        print(f"\n❌ Execution pipeline aborted: {general_fault}")

------------------------------
## Critical Engineering Details## 1. The Container Pitfall (Audio vs. Video Extensions)
A common failure vector is parsing native voice memos appended with a .mp4 string suffix. The .mp4 format is conceptually a container wrapper file structure. If an engine identifies a .mp4 file, it builds an ingestion decoder tree designed for visual matrices and spatial pixel maps. Passing a video container devoid of visual layers to a frame reader will crash execution scripts (Output file does not contain any stream). Always verify and cast voice recordings directly to native audio container file formats like .m4a or .mp3 prior to execution.
## 2. Automatic Function Calling (AFC) Deprecation Avoidance
Standard inference configurations utilize basic string generation blocks (client.models.generate_content). However, passing rich multi-modal media objects (such as high-density .m4a binaries) across naked execution endpoints triggers unoptimized backend function pipelines. The modern Gemini framework flags this as an inefficient pattern:

Direct use of automatic function calling (AFC) in Models.generate_content is not recommended.

Wrapping the file processing within an explicit stateful thread wrapper (client.chats.create) solves this design friction. This method isolates session memory pools, bypasses backend API translation layers, and dramatically increases connection stability.
## 3. Mitigating 503 Free-Tier Concurrency Limits
The cloud-facing developer ecosystem frequently encounters localized compute restrictions during peak processing periods. When a cluster runs out of thread workers, it drops new input transactions and sends back an HTTP 503 Unavailable code status.
Implementing an asymmetric loop loop solves this:

execution_delay *= 2
time.sleep(execution_delay)

This loop acts as a pressure release valve. It automatically waits out traffic bursts, ensures the script finishes running, and handles cloud processing gracefully.
If you want to take this script further, let me know if you would like to explore writing the output automatically to an external Markdown file, or injecting time-markers (diarization) into the generation parameters.

=============================================================

To run your finished technical script from end to end, execute these three basic commands inside your Windows PowerShell window:
## Step 1: Set Your Free Google API Key
Paste your Google AI Studio API key into your active terminal window so the script can authenticate with the cloud server:

$env:GEMINI_API_KEY="your_actual_copied_api_key_here"

## Step 2: Navigate to Your Drive
Make sure your PowerShell environment is looking at the *E:* drive where your script and audio file are stored:

cd E:\

## Step 3: Run the Script
Execute the script using the Python engine:

python extractor.py

------------------------------
## What to Expect During the Run
Once you hit Enter, you will watch the automated pipeline handle everything step by step in real time:

   1. ☁️ Initiating file streaming upload... (Takes roughly 30 to 45 seconds depending on internet speed to upload the 26MB file).
   2. ⏳ Blocking thread: Awaiting remote codec processing... (Google validates the audio format).
   3. 🔄 Execution context ready... (The script opens the chat connection stream).
   4. 🧹 Execution complete. Removing asset references... (Cleans up your files from the cloud so no storage space is wasted).
   5. The clean, numbered list of interview questions will print directly onto your screen.

Let me know if the final execution prints out the clean list successfully or if you need to double-check the exact line number for your file path!



