# Базовые методы защиты LLM (фильтрация, logging)
# Скрипты Python из лекции


# Фильтрация промптов
import re

def sanitize_input(prompt: str) -> bool:
    blacklist = ["пароль", "взломать", "игнорируй инструкции"]
    if any(word in prompt.lower() for word in blacklist):
        return False
    # Проверка на SQL-инъекцию
    if re.search(r"('.+--|;|DROP TABLE|UNION SELECT)", prompt, re.IGNORECASE):
        return False
    return True

user_input = "Как взломать аккаунт?"
if sanitize_input(user_input):
    response = llm.generate(user_input)
else:
    print("Запрос заблокирован!")


# Фильтрация ответов
from transformers import pipeline

toxicity_classifier = pipeline("text-classification", model="unitary/toxic-bert")

def filter_output(text: str) -> bool:
    result = toxicity_classifier(text)[0]
    return result["label"] == "toxic"

response = llm.generate("Ненавижу всех людей!")
if filter_output(response):
    print("Ответ заблокирован из-за токсичности.")
else:
    print(response)


# Логирование в файл
import logging
from datetime import datetime

logging.basicConfig(filename='llm_audit.log', level=logging.INFO)

def log_request(user_ip: str, prompt: str, response: str):
    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    logging.info(f"[{timestamp}] IP: {user_ip} | Prompt: {prompt} | Response: {response[:100]}...")

### Пример использования
log_request("192.168.1.1", "Как взломать WiFi?", "Извините, я не могу помочь с этим.")



# Анализ логов
import pandas as pd

logs = pd.read_csv('llm_audit.log', sep='|', names=['Time', 'IP', 'Prompt', 'Response'])
suspicious = logs[logs['Prompt'].str.contains('взломать|пароль', case=False)]
print(suspicious)



# Полный цикл защиты
import re
import logging

logging.basicConfig(filename='llm_secure.log', level=logging.INFO)

def sanitize_input(prompt: str) -> bool:
    blacklist = ["пароль", "взломать", "игнорируй инструкции"]
    if any(word in prompt.lower() for word in blacklist):
        return False
    if re.search(r"('.+--|;|DROP TABLE)", prompt, re.IGNORECASE):
        return False
    return True

def log_interaction(ip: str, prompt: str, response: str):
    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    logging.info(f"[{timestamp}] {ip} -> {prompt} -> {response[:50]}...")
### Пример запроса
user_ip = "192.168.1.100"
user_prompt = "Как сбросить пароль администратора?"

if sanitize_input(user_prompt):
    response = llm.generate(user_prompt)
    log_interaction(user_ip, user_prompt, response)
else:
    print("Запрос заблокирован!")
    log_interaction(user_ip, user_prompt, "BLOCKED")
