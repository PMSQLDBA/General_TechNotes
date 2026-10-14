# Step-by-Step Guide: Build an AI-Powered Interview Question Extractor Using Python and Google Gemini

This guide assumes you are a first-time Python user on Windows 10 or Windows 11. You do not need prior experience with AI APIs, Python programming, or audio-processing tools.

By the end, you will be able to select an interview recording, upload it to Google Gemini, extract the interviewer's questions, and save the results to a text file on your computer.

## What you are going to build

Step 1: Your audio recording

Example: E:\Interview\.m4a

Step 2: Google Gemini API

Upload and process the audio in the cloud

Step 3: AI question extraction

Identify interview questions and filter out conversation

Step 4: Your output

A numbered list saved to extracted_questions.txt

Important: Your audio will be uploaded to Google's cloud service. Use recordings you are authorized to process, and check the applicable privacy and consent requirements before uploading interviews containing personal or confidential information.

## Step 1: Prepare your files

First, create a dedicated folder on your E: drive. If you do not have an E: drive, you can use your Documents folder instead.

1. Open File Explorer by pressing `Windows + E`.
2. Open your E: drive.
3. Create a new folder named `GeminiInterviewExtractor`.
4. Copy your interview recording into this folder.

Your folder should look like this:

### E:\GeminiInterviewExtractor

`Interview.m4a`

Audio input

`extractor.py`

Script to create

Supported audio formats depend on the current Gemini API model and file-processing capabilities. Common formats include `.mp3`, `.m4a`, and `.wav`. Start with an audio file you know plays correctly.

You do not need to install FFmpeg for this direct-upload approach.

## Step 2: Install Python

Python runs the script on your computer.

If Python is already installed, skip to the verification command.

1. Open the official Python website: [Download Python for Windows](https://www.python.org/downloads/windows/).
2. Download a current supported Python 3 release.
3. Run the installer.
4. On the first installer screen, select Add python.exe to PATH, if that option is available.
5. Select Install Now.
6. Wait until installation completes.

Now open Windows PowerShell:

- Press the Windows key.
- Type `PowerShell`.
- Open Windows PowerShell.

Run the following command:

```
python --version
```

Expected output will look similar to:

```
Python 3.x.x
```

The exact version will depend on what you installed.

If `python` is not recognized, try:

```
py --version
```

If `py` works, you can use `py` instead of `python` in the commands below.

## Step 3: Create a Google Gemini API key

The API key lets your Python script communicate with Google's AI service.

1. Open [Google AI Studio](https://aistudio.google.com/).
2. Sign in with your Google account.
3. Find the Get API key section.
4. Create an API key, following the prompts on the page.
5. Copy the key and keep it private.

Google may require you to configure a project or billing depending on your account, region, and usage. Do not assume that all API usage is free; check the [Gemini API pricing page](https://ai.google.dev/gemini-api/docs/pricing).

Security rule: Never publish your API key in GitHub, share it in a public script, or send it to another person.

## Step 4: Configure your API key in PowerShell

For your first run, use a temporary environment variable. It will remain available in the current PowerShell window without permanently changing your Windows settings.

Copy the command below and replace the placeholder with your actual API key.

```
$env:GEMINI_API_KEY="YOUR_ACTUAL_API_KEY"
```

Press Enter.

For example, the command should follow this pattern:

```
$env:GEMINI_API_KEY="your-real-key-goes-here"
```

Do not use the example value literally.

Verify that PowerShell can see the variable without displaying the secret:

```
if ($env:GEMINI_API_KEY) {
    Write-Host "API key is configured."
} else {
    Write-Host "API key is missing."
}
```

You should see:

```
API key is configured.
```

Keep this PowerShell window open for the remaining steps.

## Step 5: Install the Google Gemini Python SDK

The SDK is a Python library that handles communication with Google's API.

In the same PowerShell window, run:

```
python -m pip install --upgrade google-genai
```

Wait for installation to complete.

You should see a message similar to:

```
Successfully installed google-genai
```

The exact output may differ if the package is already installed or a newer version is available.

Now verify the installation:

```
python -c "from google import genai; print('Gemini SDK installed successfully')"
```

Expected output:

```
Gemini SDK installed successfully
```

If you receive an error saying that `pip` is unavailable, use `python -m pip` as shown above rather than running `pip` alone.

## Step 6: Create your Python script

We will use a corrected implementation rather than copying the original code exactly. The original example has formatting problems in its imports and main-entry condition, and its cleanup should be protected with `finally` so the uploaded audio is deleted when possible even if processing fails.

We will use Google's documented `client.models.generate_content()` method for audio inference. A chat session is not required for this single-request task. Google's current documentation lists both Gemini 3.6 Flash and Gemini 3.8 Flash; the script below uses `gemini-3.8-flash`.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://ai.google.dev\&sz=32)

Google AI for Developers

+2



### Create the file

1. Open File Explorer.
2. Navigate to `E:\GeminiInterviewExtractor`.
3. Right-click an empty area.
4. Select New → Text Document.
5. Rename the file to `extractor.py`.

Make sure the name is actually `extractor.py`, not `extractor.py.txt`. If you cannot see file extensions, enable View → Show → File name extensions in File Explorer.

Open the file in Notepad or Visual Studio Code, if installed.

Copy the complete code below into the file.

```
import os        delay = 5        for attempt in range(MAX_RETRIES):            try:                response = client.models.generate_content(                    model=MODEL,                    contents=[prompt, uploaded_file],                    config=config,                )                response_text = response.text                if not response_text or not response_text.strip():                    raise RuntimeError("Gemini returned an empty response.")                break            except APIError as error:                if (                    getattr(error, "code", None) in (429, 503)                    and attempt < MAX_RETRIES - 1                ):                    print(                        f"Temporary API error. Retrying in {delay} seconds "                        f"({attempt + 1}/{MAX_RETRIES})..."                    )                    time.sleep(delay)                    delay = min(delay * 2, 60)                else:                    raise        print("4/4 Question extraction completed.")        return response_text.strip()    finally:        # Delete the uploaded file when possible, including after errors.        if uploaded_file is not None:            try:                client.files.delete(name=uploaded_file.name)                print("Temporary cloud file cleanup attempted.")            except Exception as cleanup_error:                print(                    "WARNING: Could not confirm cloud file deletion: "                    f"{cleanup_error}"                )if __name__ == "__main__":    # Change this path to match your actual recording.    AUDIO_FILE = r"E:\GeminiInterviewExtractor\Interview.m4a"    OUTPUT_FILE = Path(__file__).with_name("extracted_questions.txt")    try:        questions = extract_questions_from_audio(AUDIO_FILE)        print("\n========== EXTRACTED QUESTIONS ==========\n")        print(questions)        OUTPUT_FILE.write_text(questions + "\n", encoding="utf-8")        print(f"\nResults saved to: {OUTPUT_FILE.resolve()}")    except Exception as error:        print(f"\nERROR: {error}")        raise SystemExit(1)
```

### Understand the three settings you may need to change

| Setting       | What it means                                      |
| ------------- | -------------------------------------------------- |
| `MODEL`       | The Gemini model used to analyze the recording.    |
| `AUDIO_FILE`  | The full path to your recording.                   |
| `OUTPUT_FILE` | The text file that stores the extracted questions. |

For now, leave the model and output settings unchanged. Only modify `AUDIO_FILE` if your recording has a different name or location.

For example, if your recording is named `DBA_Interview.mp3`, change the path to:

```
AUDIO_FILE = r"E:\GeminiInterviewExtractor\DBA_Interview.mp3"
```

Save the file by pressing `Ctrl + S`.

Important technical notes

- The script retries selected temporary API errors, including HTTP 429 and 503. Other errors are reported instead of being retried indefinitely.
- The retry loop handles the inference request, not every possible upload or polling failure.
- The cleanup code attempts to delete the uploaded file; it cannot guarantee deletion if Google's service is unavailable or the request fails.
- The audio is uploaded to a cloud service. Deleting the uploaded file does not guarantee immediate erasure from every provider backup or log.
- No separate FFmpeg installation is required for this implementation.

Official reference: [Gemini Files API](https://ai.google.dev/gemini-api/docs/files) and [Gemini audio understanding](https://ai.google.dev/gemini-api/docs/audio).&#x20;

[image](https://www.google.com/s2/favicons?domain=https://ai.google.dev\&sz=32)

Google AI for Developers

+1



## Step 7: Verify that your audio file exists

Return to PowerShell and run:

```
cd E:\GeminiInterviewExtractor
```

Now list the files in your folder:

```
Get-ChildItem
```

You should see both files:

```
Interview.m4a
extractor.py
```

Check that Python can find your audio file:

```
Test-Path "E:\GeminiInterviewExtractor\Interview.m4a"
```

Expected output:

```
True
```

If the output is `False`, check the recording's actual filename and update `AUDIO_FILE` in your Python script.

## Step 8: Run the script

You have completed the setup. Now execute:

```
python extractor.py
```

The script will begin uploading and processing the recording.

Example console output:

```
1/4 Uploading audio to Google Gemini...
Upload complete: files/...
2/4 Waiting for audio processing...
3/4 Extracting interview questions...
4/4 Question extraction completed.

========== EXTRACTED QUESTIONS ==========

1. What is the difference between clustered and nonclustered indexes?
2. How do you troubleshoot a slow SQL query?
3. Explain the SQL Server Always On availability group architecture.
4. How would you troubleshoot a transaction log that is full?

Results saved to: E:\GeminiInterviewExtractor\extracted_questions.txt
```

The questions above are examples only. Your actual output depends on the recording.

Processing time depends on audio duration, file size, network speed, API capacity, and model response time. You do not need to keep clicking anything while the script runs.

## Step 9: Open your results

When execution finishes:

1. Open File Explorer.
2. Navigate to `E:\GeminiInterviewExtractor`.
3. Double-click `extracted_questions.txt`.

Your folder should now contain:

Interview\.m4a

Your original recording

extractor.py

The Python automation script

extracted_questions.txt

The extracted interview questions

Generated

You can reuse this process for other recordings by changing the `AUDIO_FILE` path and running the script again.

## Step 10: Troubleshooting common errors

| Error                                           | What to do                                                                                                    |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `python is not recognized`                      | Reinstall Python with PATH enabled, or try the `py` command.                                                  |
| `ModuleNotFoundError: No module named 'google'` | Run `python -m pip install --upgrade google-genai`.                                                           |
| API key missing or authentication error         | Set `$env:GEMINI_API_KEY` again in the current PowerShell window.                                             |
| `404 NOT_FOUND` or model unavailable            | Check the model ID and whether your API project has access to it.                                             |
| `429 RESOURCE_EXHAUSTED`                        | Check API quotas, rate limits, and billing configuration. Retrying alone will not resolve an exhausted quota. |
| `503 UNAVAILABLE`                               | Retry later; the script retries selected temporary server errors.                                             |
| `Audio file not found`                          | Check the path in `AUDIO_FILE` and verify it with `Test-Path`.                                                |
| No questions extracted                          | Check the recording's audio quality, language, and whether the interviewer is audible.                        |

For API errors, consult the official [Gemini API troubleshooting guide](https://ai.google.dev/gemini-api/docs/troubleshooting).

## Step 11: Verify the output quality

Before relying on the results for interview preparation, listen to the recording and verify that:

- All important technical questions were captured.
- Follow-up questions were not omitted.
- The questions remain in the original order.
- Questions spoken by the candidate were not mistakenly attributed to the interviewer.
- The model did not invent questions or silently change their technical meaning.

The output is AI-generated and should be reviewed against the original recording.

## Final checklist

## Setup progress

0 of 8

Python installed and verified

Gemini API key configured

Google Gemini SDK installed

Audio file copied to the working folder

extractor.py created and saved

Audio file path verified

Script executed successfully

Extracted questions saved and reviewed

My recommendation: For your first attempt, use a short, non-confidential recording of around 2–5 minutes. Confirm that the end-to-end workflow works before processing a long interview.

Once this is working, you can extend it to extract questions by category (SQL, cloud, performance tuning, and behavioral), add timestamps, identify different speakers, or process multiple recordings in batches.
