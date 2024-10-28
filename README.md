# -Conversando-por-Voz-Com-o-ChatGPT-Utilizando-Whisper-OpenAI-e-Python
pip install openai pyaudio
import openai
import sounddevice as sd
import numpy as np
import queue
import wave

# Configura suas chaves da API da OpenAI
openai.api_key = "SUA_OPENAI_API_KEY"

# Configuração do áudio
samplerate = 16000  # Taxa de amostragem recomendada para o Whisper
q = queue.Queue()

# Função para capturar áudio do microfone
def audio_callback(indata, frames, time, status):
    q.put(indata.copy())

def gravar_audio(duracao=5, filename="entrada.wav"):
    with wave.open(filename, 'wb') as f:
        f.setnchannels(1)
        f.setsampwidth(2)
        f.setframerate(samplerate)
        with sd.InputStream(samplerate=samplerate, channels=1, callback=audio_callback):
            print("Gravando...")
            sd.sleep(duracao * 1000)
            print("Finalizado.")
        while not q.empty():
            f.writeframes(q.get())

# Função para transcrever áudio com Whisper
def transcrever_audio(filename):
    with open(filename, "rb") as audio_file:
        result = openai.Audio.transcribe("whisper-1", audio_file)
    return result["text"]

# Função para obter resposta do ChatGPT
def obter_resposta(mensagem):
    resposta = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[
            {"role": "user", "content": mensagem}
        ]
    )
    return resposta['choices'][0]['message']['content']

# Função principal
def main():
    # Passo 1: Gravar áudio
    gravar_audio(duracao=5)
    
    # Passo 2: Transcrever áudio
    mensagem = transcrever_audio("entrada.wav")
    print(f"Você disse: {mensagem}")
    
    # Passo 3: Obter resposta do ChatGPT
    resposta = obter_resposta(mensagem)
    print(f"ChatGPT: {resposta}")

# Executar a função principal
if __name__ == "__main__":
    main()
from gtts import gTTS
import os

def texto_para_voz(texto):
    tts = gTTS(text=texto, lang='pt')
    tts.save("resposta.mp3")
    os.system("start resposta.mp3")  # Em Windows; para MacOS/Linux, use `afplay` ou `mpg123`.
