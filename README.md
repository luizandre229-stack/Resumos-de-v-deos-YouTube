youtube-audio-summarizer/
│
├── main.py
├── requirements.txt
├── README.md
└── .gitignore
yt-dlp
openai
.env
*.mp3
*.wav
__pycache__/
import yt_dlp
import openai
import os
import uuid
import sys

# ==========================
# CONFIGURAÇÃO DA API KEY
# ==========================
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")

if not OPENAI_API_KEY:
    print("ERRO: variável de ambiente OPENAI_API_KEY não definida.")
    sys.exit(1)

openai.api_key = OPENAI_API_KEY

AUDIO_FILE = f"audio_{uuid.uuid4()}.mp3"

# ==========================
# DOWNLOAD DO ÁUDIO
# ==========================
def download_audio(youtube_url):
    ydl_opts = {
        "format": "bestaudio/best",
        "outtmpl": AUDIO_FILE,
        "postprocessors": [{
            "key": "FFmpegExtractAudio",
            "preferredcodec": "mp3",
            "preferredquality": "192",
        }],
        "quiet": True,
        "js_runtimes": ["node"]
    }

    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        ydl.download([youtube_url])

    return AUDIO_FILE

# ==========================
# TRANSCRIÇÃO (WHISPER)
# ==========================
def transcribe_audio(audio_path):
    with open(audio_path, "rb") as audio_file:
        transcript = openai.Audio.transcribe(
            model="whisper-1",
            file=audio_file,
            response_format="text"
        )
    return transcript

# ==========================
# RESUMO
# ==========================
def summarize_text(text):
    response = openai.ChatCompletion.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": "Crie um resumo claro, organizado e objetivo."},
            {"role": "user", "content": f"Resuma o texto abaixo:\n\n{text}"}
        ],
        temperature=0.3
    )
    return response.choices[0].message.content

# ==========================
# MAIN
# ==========================
if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Uso: python main.py <URL_DO_YOUTUBE>")
        sys.exit(1)

    youtube_url = sys.argv[1]

    print("🎧 Baixando áudio...")
    audio_path = download_audio(youtube_url)

    print("📝 Transcrevendo áudio...")
    transcription = transcribe_audio(audio_path)

    print("📌 Gerando resumo...")
    summary = summarize_text(transcription)

    print("\n===== RESUMO DO VÍDEO =====\n")
    print(summary)

    os.remove(audio_path)
