# opensubtitles-auto-downloader
Automated subtitle downloader for movies and TV shows using OpenSubtitles.com API v1 with Telegram notifications.
Subtitle Auto-Downloader (OpenSubtitles.com)
This Python script automatically scans your movie and series directories to find and download missing subtitles in your preferred language using the OpenSubtitles.com API (v1). It also sends a notification to Telegram once a subtitle is successfully downloaded.

🚀 Features
Recursive Scanning: Searches through movie and series folders.

Smart Search: Tries searching by original filename first, then by a "cleaned" version (removing tags like 1080p, x264, etc.).

Multi-language Support: Configurable language via config.json.

Telegram Integration: Real-time notifications when subtitles are found or if errors occur.

Duplicate Prevention: Checks if a subtitle for the specific language already exists before downloading.

📋 Prerequisites
Before running the script, you will need:

1. OpenSubtitles.com Credentials
Create an account at OpenSubtitles.com.

Go to your Profile Settings > API Management.

Register a new Application to receive your API Key.

2. Telegram Bot (Optional for Notifications)
Message @BotFather on Telegram and create a new bot to get your Token.

Message @userinfobot to find your Chat ID.

⚙️ Configuration
Create a config.json file in the root directory with the following structure:

JSON

{
    "opensubtitles": {
        "username": "YOUR_USERNAME",
        "password": "YOUR_PASSWORD",
        "api_key": "YOUR_API_KEY"
    },
    "paths": {
        "movies": "/path/to/your/movies",
        "series": "/path/to/your/series"
    },
    "settings": {
        "language": "en"
    }
}
language: Use ISO 639-1 codes (e.g., en for English, es for Spanish, fr for French).

🛠️ Installation & Usage
Clone the repository:

Bash

git clone https://github.com/youruser/subtitle-downloader.git
cd subtitle-downloader
Install dependencies: The script requires the requests library.

Bash

pip install requests
Run the script:

Bash

python downloader.py
📖 How it Works
Authentication: The script logs into the OpenSubtitles API using your credentials to retrieve a session token.

File Discovery: It walks through your configured movies and series paths looking for .mp4, .mkv, and .avi files.

Language Check: It checks the folder for an existing .srt file containing your target language code (e.g., movie_name.en.srt).

Search Logic:

Attempt 1: Searches using the full file name.

Attempt 2: If no results, it "cleans" the name (removes dots, resolutions, and codecs) and tries again.

Download: If a match is found, it downloads the file and saves it with the proper language suffix.

Notification: A message is sent to your Telegram bot with the result.
