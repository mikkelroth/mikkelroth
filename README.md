import webbrowser
import random
import threading
import requests
from bs4 import BeautifulSoup
import sympy
import os
import time
import enchant
from textblob import TextBlob
import openai
import wikipediaapi
import yfinance as yf
from transformers import pipeline
import pyttsx3
import speech_recognition as sr

class Mimer:
    def __init__(self):
        self.data_sources = {
            "bbc": "https://www.bbc.com/",
            "faktalink": "https://faktalink.dk/?check_logged_in=1",
            "gyldendal": "https://www.gyldendal.dk/",
            "alinea": "https://www.alinea.dk/",
            "indidansk": "https://indidansk.dk/",
            "office": "https://www.office.com/?auth=2",
            "historienet": "https://historienet.dk/",
            "ordbogen": "https://www.ordbogen.com/da/#/",
            "google_translate": "https://translate.google.com/?hl=da",
            "true_size": "https://www.thetruesize.com/#?borders=1~!MTU0Njg2NzU.NTA3NDUzNw*MzIwOTY2NTA(NTM5NTQ3OA"
        }
        self.last_played_song = None
        self.dict = enchant.Dict("en_US")
        self.engine = pyttsx3.init()
        self.recognizer = sr.Recognizer()
        self.deepL = pipeline("translation_en_to_da")
        threading.Thread(target=self.remember_last_played_song).start()
        threading.Thread(target=self.authenticate_youtube).start()
        openai.api_key = 'YOUR_OPENAI_API_KEY'  # Erstat med din egen API-nøgle

    def answer_question(self, question):
        corrected_question = self.correct_text(question)
        if "oversæt" in corrected_question.lower():
            translated_text = self.translate_text(corrected_question)
            return translated_text
        elif "wikipedia" in corrected_question.lower():
            return self.search_wikipedia(corrected_question)
        elif "åbn" in corrected_question.lower():
            return self.open_website(corrected_question)
        elif "kvadratroden af" in corrected_question.lower():
            return self.calculate_square_root(corrected_question)
        elif "arealet af en trekant" in corrected_question.lower():
            return self.calculate_triangle_area(corrected_question)
        elif "alarm" in corrected_question.lower():
            return self.set_alarm(corrected_question)
        elif "google" in corrected_question.lower():
            return self.google_search(corrected_question)
        elif any(word in corrected_question.lower() for word in ["beregne", "udtryk", "ligning", "afledede", "integral"]):
            return self.handle_math_operations(corrected_question)
        elif "aktier" in corrected_question.lower():
            return self.stock_analysis(corrected_question)
        else:
            responses = [
                "Jeg er ikke sikker. Vil du have mig til at søge det for dig?",
                "Det kunne jeg ikke finde information om.",
                "Ja, det ser sådan ud.",
                "Nej, det tror jeg ikke."
            ]
            return random.choice(responses)

    def correct_text(self, text):
        corrected_text = []
        for word in text.split():
            if not self.dict.check(word):
                suggestions = self.dict.suggest(word)
                if suggestions:
                    corrected_text.append(suggestions[0])
                else:
                    corrected_text.append(word)
            else:
                corrected_text.append(word)
        corrected_text = " ".join(corrected_text)
        blob = TextBlob(corrected_text)
        corrected_text = str(blob.correct())
        return corrected_text

    def translate_text(self, text):
        try:
            translated_text = self.deepL(text)
            return translated_text
        except Exception as e:
            print("Fejl under oversættelse:", e)
            return "Kunne ikke oversætte teksten."

    def search_wikipedia(self, query):
        try:
            wiki_wiki = wikipediaapi.Wikipedia('en')
            page = wiki_wiki.page(query)
            return page.summary[:200]
        except Exception as e:
            print("Fejl under Wikipedia-søgning:", e)
            return "Kunne ikke finde information på Wikipedia."

    def open_website(self, question):
        try:
            for site_name, site_url in self.data_sources.items():
                if site_name.lower() in question.lower():
                    webbrowser.open(site_url)
                    return f"Åbner {site_name.capitalize()}"
            return "Jeg kunne ikke forstå, hvil
