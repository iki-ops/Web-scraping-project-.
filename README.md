# Web-scraping-project-.
From selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from bs4 import BeautifulSoup
import time
import os

# Налаштування Selenium
chrome_options = Options()
chrome_options.add_argument("--headless")

# Отримуємо шлях до chromedriver з змінної середовища
chromedriver_path = os.getenv('CHROMEDRIVER_PATH')
if not chromedriver_path:
    raise Exception("Змінна середовища 'CHROMEDRIVER_PATH' не знайдена. Вкажіть шлях до chromedriver.exe.")

service = Service(chromedriver_path)

# Запуск браузера
driver = webdriver.Chrome(service=service, options=chrome_options)

# Базовий URL сайту без номера сторінки
base_url = "https://spaces.im/sz/knigi/fxntezi/java-knigi/p"

def parse_page(driver, search_word):
    html_content = driver.page_source
    soup = BeautifulSoup(html_content, "html.parser")
    paragraphs = soup.find_all(['p', 'div', 'span', 'h1', 'h2', 'h3', 'a'])

    found = False
    for i, paragraph in enumerate(paragraphs):
        paragraph_text = paragraph.get_text()
        if search_word.lower() in paragraph_text.lower():
            found = True
            word_start = paragraph_text.lower().find(search_word.lower())
            context = paragraph_text[max(0, word_start - 30):word_start + len(search_word) + 30]
            print(f"Знайдено слово '{search_word}' в абзаці {i + 1}:")
            print(f"Контекст: {context}")
            print(f"Повний абзац: {paragraph_text}\n")

    if not found:
        print(f"Слово '{search_word}' не знайдено на цій сторінці.")
    return found

# Слово, яке шукаєте
search_word = "Государь и его коммандос"

page_number = 1
max_pages = 5

while page_number <= max_pages:
    # Формування URL для сторінки з суфіксом p{page_number}
    current_url = f"{base_url}{page_number}"
    print(f"Перевірка URL: {current_url}")
    driver.get(current_url)

    # Оновлення сторінки для забезпечення правильного завантаження контенту
    driver.refresh()
    time.sleep(5)  # Затримка для повного завантаження сторінки

    # Перевірка, чи завантажився контент
    WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.TAG_NAME, "body"))
    )

    print(f"Парсинг сторінки {page_number}...")
    found = parse_page(driver, search_word)

    if found:
        break

    page_number += 1

print("Парсинг завершено.")
driver.quit()
