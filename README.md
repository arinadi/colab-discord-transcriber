# Colab Discord Transcriber

> A robust Discord bot that leverages OpenAI's Whisper to transcribe audio and video files, with an intelligent idle-shutdown system, optimized for Google Colab.

This project provides a powerful, easy-to-deploy Discord bot that transcribes audio and video files directly within your Discord channel. It's designed to run seamlessly in a Google Colab environment, utilizing its free GPU resources for fast transcriptions while intelligently managing runtime to conserve compute units.

## ✨ Key Features

*   **High-Quality & Configurable Transcription**:
    *   Utilizes OpenAI's `large-v2` Whisper model (or other sizes) for state-of-the-art accuracy.
    *   Advanced settings like **FP16 precision** for doubled speed on GPUs and **beam size** adjustment for a fine-tuned accuracy/speed trade-off.
*   **Intelligent Idle Auto-Shutdown**:
    *   Automatically monitors for inactivity to conserve Google Colab compute units.
    *   Sends configurable webhook notifications for idle warnings before shutting down.
    *   Gracefully terminates the Colab runtime, preventing resource waste.
*   **Advanced File Handling**:
    *   Processes single file uploads.
    *   Automatically extracts and processes files from `.zip` archives.
    *   Intelligently merges and processes **split `.zip` archives** (e.g., `.zip.001`, `.zip.002`).
*   **Robust Job Queueing System**:
    *   Built with an asynchronous queue (`asyncio.Queue`) to handle multiple concurrent file uploads without crashing.
    *   Provides users with a unique **Job ID** and their position in the queue.
*   **Pre-flight Validation**:
    *   Before queueing, the bot uses `ffmpeg` to probe files, rejecting corrupted or unsupported media.
    *   Rejects files that exceed a configurable maximum duration, preventing resource monopolization.
*   **Detailed User Feedback & Notifications**:
    *   Provides a step-by-step feedback loop in Discord for each job.
    *   Uses **webhooks** for critical status notifications (startup, idle warnings, auto-shutdown, critical errors).
    *   Replies directly to the user's message with a rich `Embed` containing the transcript, file details, and **detected language**.
*   **Easy & Secure Deployment**:
    *   Designed for one-click execution in a Google Colab notebook.
    *   Uses Colab's built-in `Secrets` manager to keep your bot token and webhook URL safe.

## ⚙️ How It Works

1.  **Upload**: A user uploads one or more files (or a ZIP/split ZIP archive) to the designated Discord channel.
2.  **Validation**: The `FilesHandler` immediately probes each file. It verifies that the file is a valid media format and that its duration is within the configured limits. Invalid files are rejected with an error message.
3.  **Queueing**: Valid files are encapsulated into a `TranscriptionJob` object, assigned a unique Job ID, and placed into an asynchronous queue. The user receives a "Queued" confirmation with their job ID and queue position.
4.  **Processing**: A background worker (`queue_processor`) picks up the next job. It sends a "Processing" message, including the file's duration.
5.  **Transcription**: The core transcription is performed by the Whisper model. It generates the text and detects the source language.
6.  **Delivery**: The bot formats the transcript, saves it to a `.txt` file, and replies to the user's original message with an embed containing the results.
7.  **Cleanup**: All temporary files (uploads, extracted files, and transcripts) are deleted from the Colab runtime to conserve space.
8.  **Idle Monitoring**: In the background, the `IdleMonitor` tracks activity. If the bot is idle for a configurable period, it sends webhook notifications and eventually shuts down the Colab runtime to save resources.

## 🔧 Setup and Installation

Follow these steps to get your own instance of the bot running.

### Prerequisites

*   A Google Account (for Google Colab).
*   A Discord Account with a server where you have administrative privileges.

### Step-by-Step Guide

1.  **Create a Discord Bot Application**:
    *   Go to the [Discord Developer Portal](https://discord.com/developers/applications).
    *   Click "New Application" and give it a name.
    *   Go to the **Bot** tab.
    *   Under **Privileged Gateway Intents**, enable the **MESSAGE CONTENT INTENT**. This is crucial.
    *   Click "Reset Token" and **copy the bot token**. Keep it safe.

2.  **Invite the Bot to Your Server**:
    *   Go to the "OAuth2" -> "URL Generator" tab.
    *   Select the `bot` scope.
    *   Under "Bot Permissions", grant: `Read Messages/View Channels`, `Send Messages`, `Read Message History`, and `Attach Files`.
    *   Copy the generated URL, paste it into your browser, and invite the bot to your server.

3.  **Get Channel ID and Webhook URL**:
    *   **Enable Developer Mode** in Discord (User Settings -> Advanced -> Developer Mode).
    *   **Channel ID**: Right-click the channel where the bot will operate and select "Copy Channel ID".
    *   **Webhook URL**: In the same channel, go to Channel Settings -> Integrations -> Webhooks. Create a "New Webhook", give it a name, and "Copy Webhook URL". This is used for status notifications.

4.  **Configure Google Colab**:
    *   Open the `.ipynb` notebook file in Google Colab.
    *   On the left sidebar, click the **Key icon (Secrets)**.
    *   Add the following **three** secrets:
        *   `DISCORD_BOT_TOKEN`: The bot token from Step 1.
        *   `DISCORD_CHANNEL_ID`: The channel ID from Step 3.
        *   `DISCORD_WEBHOOK_URL`: The webhook URL from Step 3.
    *   Ensure the toggles for "Notebook access" are enabled for all three secrets.

5.  **Run the Bot**:
    *   At the top of the Colab notebook, review the configuration options in the first cell.
    *   Click the "Run" button on that cell (or press `Ctrl+Enter`).
    *   The cell will install dependencies, load the model, and bring the bot online. You will see a "Bot has logged in as..." message in the output and receive a startup notification on Discord via your webhook.

## 🚀 Usage

*   **Transcribe a File**: Drag and drop any audio or video file into the designated channel.
*   **Transcribe from ZIP**: Upload a `.zip` file. The bot will extract and queue all valid media files inside.
*   **Transcribe from Split ZIP**: Upload all parts of a split archive (e.g., `archive.zip.001`, `archive.zip.002`). The bot will automatically merge and process them.
*   **Check Bot Status**: Type `!ping`. The bot will reply with a detailed embed showing its latency, queue size, and current idle status with countdowns to the next action (notification, warning, or shutdown).
*   **Shut Down Manually**: Type `!shutdown`. The bot will log off and terminate the Colab runtime gracefully.

## ⚙️ Configuration

You can easily tweak the bot's behavior by editing the parameters in the first code cell of the Colab notebook.

#### Model & Transcription Settings
*   `model_size`: Whisper model to use (`large-v2`, `medium`, `small`, etc.). A trade-off between speed and accuracy.
*   `use_fp16`: Use half-precision floating-point. `auto` is recommended to double speed on GPUs.
*   `beam_size`: Higher values can increase accuracy at the cost of speed. `5` is a good default. Set to `0` to disable.
*   `pause_threshold`: The duration of silence (in seconds) that creates a new paragraph in the transcript.
*   `MAX_AUDIO_DURATION_SECONDS`: The maximum allowed duration of a media file. `0` disables the check.

#### Idle Monitor & Auto-Shutdown
*   `IDLE_NOTIFY_MIN`: Minutes of inactivity before the first webhook notification is sent.
*   `IDLE_WARN_MIN`: Minutes of inactivity before a final warning notification is sent.
*   `IDLE_SHUTDOWN_MIN`: Minutes of inactivity before the bot automatically shuts down the Colab runtime.

## 📄 License

This project is licensed under the MIT License. See the `LICENSE` file for details.
