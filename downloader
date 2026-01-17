import os
import json
import requests
import re

# --- Filename cleaning function ---
def clean_filename(filename):
    name, _ = os.path.splitext(filename)
    # Remove resolutions, codecs, dots, etc.
    name = re.sub(r'\d{3,4}p', '', name, flags=re.IGNORECASE)
    name = re.sub(r'\.', ' ', name)
    return name.strip()

# --- Load configuration ---
with open("config.json") as f:
    config = json.load(f)

OS_USER = config["opensubtitles"]["username"]
OS_PASS = config["opensubtitles"]["password"]
OS_API_KEY = config["opensubtitles"]["api_key"]

PATH_MOVIES = config["paths"]["movies"]
PATH_SERIES = config["paths"]["series"]

# --- Language settings ---
LANG = config.get("settings", {}).get("language", "en")

# --- Telegram Notification Setup ---
TELEGRAM_TOKEN = config.get("telegram", {}).get("token")
TELEGRAM_CHAT_ID = config.get("telegram", {}).get("chat_id")

def send_telegram(msg):
    if not TELEGRAM_TOKEN or not TELEGRAM_CHAT_ID:
        return
    url = f"https://api.telegram.org/bot{TELEGRAM_TOKEN}/sendMessage"
    data = {"chat_id": TELEGRAM_CHAT_ID, "text": msg}
    try:
        requests.post(url, data=data)
    except Exception as e:
        print(f"⚠️ Error sending Telegram message: {e}")

# --- Base headers for all requests ---
HEADERS_BASE = {
    "Api-Key": OS_API_KEY,
    "Content-Type": "application/json",
    "User-Agent": "SubDownloader v1.0"
}

# --- OpenSubtitles Authentication ---
def login():
    url = "https://api.opensubtitles.com/api/v1/login"
    data = {"username": OS_USER, "password": OS_PASS}
    r = requests.post(url, headers=HEADERS_BASE, json=data)
    if r.status_code == 200:
        token = r.json()["token"]
        print("✅ Login successful on OpenSubtitles")
        return token
    else:
        print("❌ Login error:", r.text)
        return None

# --- Search subtitle ---
def search_subtitle(token, file_path):
    url = "https://api.opensubtitles.com/api/v1/subtitles"
    headers = HEADERS_BASE.copy()
    headers["Authorization"] = f"Bearer {token}"
    filename = os.path.basename(file_path)
    name_without_ext, _ = os.path.splitext(filename)

    # 1️⃣ Try original filename first
    print(f"🔎 Searching ({LANG}) subtitle with original name: {name_without_ext}")
    params = {"query": name_without_ext, "languages": LANG}
    r = requests.get(url, headers=headers, params=params)

    if r.status_code == 200:
        data = r.json()
        if data["data"]:
            file_id = data["data"][0]["attributes"]["files"][0]["file_id"]
            return file_id

    # 2️⃣ If not found, try cleaned name
    clean_name = clean_filename(filename)
    print(f"🧹 Retrying with cleaned name: {clean_name}")
    params = {"query": clean_name, "languages": LANG}
    r = requests.get(url, headers=headers, params=params)

    if r.status_code == 200:
        data = r.json()
        if data["data"]:
            file_id = data["data"][0]["attributes"]["files"][0]["file_id"]
            return file_id
    else:
        print(f"❌ HTTP Error {r.status_code}: {r.text}")

    return None

# --- Download subtitle ---
def download_subtitle(token, file_id, dest_folder, original_filename):
    url = "https://api.opensubtitles.com/api/v1/download"
    headers = HEADERS_BASE.copy()
    headers["Authorization"] = f"Bearer {token}"
    data = {"file_id": file_id}
    r = requests.post(url, headers=headers, json=data)
    if r.status_code == 200:
        link = r.json()["link"]
        sub = requests.get(link, headers=HEADERS_BASE)
        base, _ = os.path.splitext(original_filename)
        dest_path = os.path.join(dest_folder, f"{base}.{LANG}.srt")
        with open(dest_path, "wb") as f:
            f.write(sub.content)
        print(f"✅ Subtitle downloaded as {dest_path}")
        send_telegram(f"🎬 Subtitle ({LANG}) downloaded for {original_filename} ✅")
        return True
    else:
        print(f"❌ Error downloading subtitle. HTTP {r.status_code}: {r.text}")
        send_telegram(f"❌ Error downloading subtitle for {original_filename}. HTTP {r.status_code}")
        return False

# --- Check if subtitle already exists ---
def has_lang_sub(folder, basename):
    for f in os.listdir(folder):
        if f.endswith(".srt") and (basename in f) and (f".{LANG}." in f.lower()):
            return True
    return False

# --- Process a video file ---
def process_file(token, file_path):
    folder = os.path.dirname(file_path)
    filename = os.path.basename(file_path)
    basename, _ = os.path.splitext(filename)

    if has_lang_sub(folder, basename):
        print(f"🎬 {filename} already has {LANG} subtitles")
        return

    print(f"🔎 Searching subtitles for: {filename}")
    file_id = search_subtitle(token, file_path)
    if file_id:
        success = download_subtitle(token, file_id, folder, filename)
        if not success:
            print(f"⚠️ Failed to download subtitle for {filename}")
    else:
        print(f"⚠️ No ({LANG}) subtitles found for: {filename}")
        send_telegram(f"⚠️ No ({LANG}) subtitles found for: {filename}")

# --- Main Function ---
def main():
    token = login()
    if not token:
        return

    print(f"🚀 Starting subtitle search in language: {LANG}...\n")

    # --- Process Movies ---
    if os.path.exists(PATH_MOVIES):
        print("🎞️  Checking movies...")
        for root, dirs, files in os.walk(PATH_MOVIES):
            for file in files:
                if file.endswith((".mp4", ".mkv", ".avi")):
                    process_file(token, os.path.join(root, file))
    else:
        print(f"⚠️ Movies path not found: {PATH_MOVIES}")

    # --- Process Series ---
    if os.path.exists(PATH_SERIES):
        print("\n📺 Checking series...")
        for series_folder in os.listdir(PATH_SERIES):
            series_path = os.path.join(PATH_SERIES, series_folder)
            if not os.path.isdir(series_path):
                continue

            print(f"\n➡️ Show: {series_folder}")
            for root, dirs, files in os.walk(series_path):
                for file in files:
                    if file.endswith((".mp4", ".mkv", ".avi")):
                        process_file(token, os.path.join(root, file))
    else:
        print(f"⚠️ Series path not found: {PATH_SERIES}")

    print("\n✅ Finished: Subtitle search completed.")

if __name__ == "__main__":
    main()
