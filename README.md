# Passo 0: Instale as bibliotecas necessárias
# pip install SpeechRecognition pyaudio pyttsx3 wikipedia-api gTTS playsound

import speech_recognition as sr
import pyttsx3
import webbrowser
import wikipediaapi
import requests # Usado para verificar a conexão com a internet

# --- Módulo 1: Text-to-Speech (TTS) ---
engine = pyttsx3.init()

def speak(text):
    """
    Função para converter texto em áudio e falar.
    """
    print(f"Assistente: {text}")
    engine.say(text)
    engine.runAndWait()

# --- Módulo 2: Speech-to-Text (STT) ---
def listen_for_command():
    """
    Função para ouvir o microfone, reconhecer a fala e retornar o texto.
    """
    recognizer = sr.Recognizer()
    with sr.Microphone() as source:
        print("Ouvindo...")
        # Ajusta o ruído do ambiente para melhorar o reconhecimento
        recognizer.adjust_for_ambient_noise(source)
        audio = recognizer.listen(source)

    try:
        print("Reconhecendo...")
        # Tenta reconhecer a fala usando a API do Google
        command = recognizer.recognize_google(audio, language='pt-BR')
        print(f"Você disse: {command}\n")
        return command.lower()
    except sr.UnknownValueError:
        # Erro se não entender o que foi dito
        speak("Desculpe, não entendi o que você disse.")
        return None
    except sr.RequestError:
        # Erro se houver problema com a API do Google (ex: sem internet)
        speak("Não consegui me conectar ao serviço de reconhecimento. Verifique sua internet.")
        return None

# --- Funções de Ação (Módulo 3) ---

def search_wikipedia(query):
    """
    Pesquisa um termo na Wikipedia e resume o resultado.
    """
    # Define o idioma da Wikipedia para Português
    wiki = wikipediaapi.Wikipedia('pt')
    page = wiki.page(query)
    
    if page.exists():
        # Pega os primeiros 3 parágrafos do resumo
        summary = "\n".join(page.summary.split('\n')[:3])
        speak(f"De acordo com a Wikipedia: {summary}")
    else:
        speak(f"Desculpe, não encontrei resultados para '{query}' na Wikipedia.")

def open_youtube():
    """
    Abre o site do YouTube no navegador padrão.
    """
    speak("Abrindo o YouTube...")
    webbrowser.open("https://www.youtube.com")

def find_nearest_pharmacy():
    """
    Abre o Google Maps com a busca pela farmácia mais próxima.
    """
    speak("Mostrando a farmácia mais próxima no mapa.")
    # A busca "farmácia mais próxima" no Google Maps usa a localização do dispositivo
    webbrowser.open("https://www.google.com/maps/search/?api=1&query=farmacia+mais+proxima")

# --- Loop Principal do Assistente ---
def run_assistant():
    speak("Olá! Como posso ajudar?")
    
    while True:
        command = listen_for_command()
        
        if command:
            if "pesquisar por" in command or "o que é" in command:
                # Remove o comando de ativação para pegar apenas o termo da busca
                search_term = command.replace("pesquisar por", "").replace("o que é", "").strip()
                search_wikipedia(search_term)
            
            elif "abrir o youtube" in command:
                open_youtube()
                
            elif "farmácia mais próxima" in command:
                find_nearest_pharmacy()

            elif "desligar" in command or "parar" in command:
                speak("Até mais!")
                break
            else:
                speak("Não entendi o comando. Tente novamente.")

if __name__ == "__main__":
    run_assistant()
