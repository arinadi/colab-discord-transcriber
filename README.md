# Colab Discord Transcriber

> A robust Discord bot that leverages OpenAI's Whisper to transcribe audio and video files, optimized for easy deployment on Google Colab.

This project provides a powerful and easy-to-deploy Discord bot that transcribes audio and video files directly within your Discord channel. It's designed to run seamlessly in a Google Colab environment, utilizing its free GPU resources for fast and accurate transcriptions.

## ✨ Key Features

*   **High-Quality Transcription**: Utilizes OpenAI's `large-v2` Whisper model (or other sizes) for state-of-the-art transcription accuracy.
*   **Audio & Video Support**: Transcribes audio from both audio files (`.mp3`, `.wav`, `.m4a`, etc.) and video files (`.mp4`, `.mov`, `.mkv`, etc.) by automatically extracting the audio stream.
*   **Robust Job Queueing System**: Built with an asynchronous queue (`asyncio.Queue`) to handle multiple file uploads concurrently without crashing. Each file is processed one by one in the order it was received.
*   **Advanced File Handling**:
    *   Processes single file uploads.
    *   Automatically extracts and processes files from `.zip` archives.
    *   Intelligently merges and processes split `.zip` archives (e.g., `.zip.001`, `.zip.002`).
*   **Pre-flight Validation**: Before a job is even queued, the bot validates files to:
    *   Reject corrupted or unsupported media files.
    *   Reject files that exceed a configurable maximum duration, preventing resource monopolization.
*   **Clear User Feedback**: Provides a step-by-step feedback loop in Discord:
    1.  `✅ Queued`: Confirms that your file has been received and is waiting in line.
    2.  `▶️ Processing`: Notifies you when the bot starts working on your file, including the file's duration.
    3.  `🎉 Complete`: Replies directly to your original message with the final transcript as a `.txt` file.
*   **Easy Deployment**: Designed specifically for one-click execution in a Google Colab notebook.
*   **Secure Credential Management**: Uses Colab's built-in `Secrets` manager to keep your bot token and other sensitive information safe.

## ⚙️ How It Works

The bot follows a clear and robust workflow for every file it receives:

1.  **Upload**: A user uploads one or more files (or a ZIP archive) to the designated Discord channel.
2.  **Validation**: The `FilesHandler` immediately checks each file. It uses `ffmpeg` to probe the file to ensure it's a valid media file and that its duration is within the configured limits. Invalid files are rejected with an error message.
3.  **Queueing**: Valid files are encapsulated into a `TranscriptionJob` object and placed into an asynchronous queue. The user receives a "Queued" confirmation message.
4.  **Processing**: A background worker (`queue_processor`) picks up the next job from the queue. It sends a "Processing" message to the channel to inform the user.
5.  **Transcription**: The core transcription is performed by the Whisper model running on the Colab GPU (or CPU).
6.  **Delivery**: Once complete, the bot formats the transcript, saves it to a `.txt` file, and replies to the user's original message with an embed containing the results and the attached file.
7.  **Cleanup**: All temporary files (uploads and transcripts) are deleted from the Colab runtime to conserve space.

## 🔧 Setup and Installation

Follow these steps to get your own instance of the bot running.

### Prerequisites

*   A Google Account (for Google Colab).
*   A Discord Account with a server where you have administrative privileges.

### Step-by-Step Guide

1.  **Create a Discord Bot Application**:
    *   Go to the [Discord Developer Portal](https://discord.com/developers/applications).
    *   Click "New Application" and give it a name (e.g., "Colab Transcriber").
    *   Go to the "Bot" tab.
    *   Under **Privileged Gateway Intents**, enable the **MESSAGE CONTENT INTENT**. This is crucial for the bot to read messages.
    *   Click "Reset Token" and **copy the bot token**. Keep it safe.

2.  **Invite the Bot to Your Server**:
    *   Go to the "OAuth2" -> "URL Generator" tab.
    *   Select the `bot` and `applications.commands` scopes.
    *   Under "Bot Permissions", grant the following permissions:
        *   `Read Messages/View Channels`
        *   `Send Messages`
        *   `Read Message History`
        *   `Attach Files`
    *   Copy the generated URL, paste it into your browser, and invite the bot to your server.

3.  **Get Channel and Webhook IDs**:
    *   **Channel ID**: In Discord, enable Developer Mode (Settings -> Advanced -> Developer Mode). Right-click the channel where you want the bot to operate and select "Copy Channel ID".
    *   **Webhook URL (for notifications)**: In the same channel, go to Channel Settings -> Integrations -> Webhooks. Create a "New Webhook", give it a name, and "Copy Webhook URL".

4.  **Configure Google Colab**:
    *   Open the `colab-discord-transcriber.ipynb` notebook in Google Colab.
    *   On the left sidebar, click the **Key icon (Secrets)**.
    *   Add the following three secrets:
        *   `DISCORD_BOT_TOKEN` : Paste the bot token from Step 1.
        *   `DISCORD_CHANNEL_ID`: Paste the channel ID from Step 3.
        *   `DISCORD_WEBHOOK_URL`: Paste the webhook URL from Step 3.
    *   Ensure the toggles are on for "Notebook access" for all three secrets.

5.  **Run the Bot**:
    *   At the top of the Colab notebook, you can review the configuration options.
    *   Click the "Run" button on the main code cell (or press `Ctrl+Enter`).
    *   The cell will first install dependencies, then load the model, and finally, the bot will come online. You will see a "Bot has logged in as..." message in the output and receive a startup notification on Discord via your webhook.

## 🚀 Usage

Once the bot is running, using it is simple:

*   **Transcribe a File**: Drag and drop any audio or video file into the designated channel.
*   **Transcribe Multiple Files**: Upload a `.zip` archive containing your media files.
*   **Check Bot Status**: Type `!ping` in the channel. The bot will reply with its latency and the number of jobs in the queue.
*   **Shut Down**: Type `!shutdown` in the channel. The bot will log off and terminate the Colab runtime gracefully, cleaning up all files.

## ⚙️ Configuration

You can easily tweak the bot's behavior by editing these variables at the top of the script:

*   `model_size`: The Whisper model to use. Options include `large-v2`, `medium`, `small`, and `base`. Larger models are more accurate but slower.
*   `pause_threshold_input`: The duration in seconds to be considered a significant pause, which results in a new paragraph in the transcript.
*   `MAX_AUDIO_DURATION_SECONDS`: The maximum duration of a media file in seconds. Set to `0` to disable this check. The default is `5400` (90 minutes).

## 📄 License

This project is licensed under the MIT License. See the `LICENSE` file for details.
