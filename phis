from telethon import TelegramClient
from googletrans import Translator
import requests
import time
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from bs4 import BeautifulSoup

# Thông tin đăng nhập Telegram API (dùng số điện thoại của bạn)
api_id = 
api_hash = 
phone_number = 

# Khởi tạo Telethon client
client = TelegramClient('', api_id, api_hash)

# Khởi tạo Google Translator
translator = Translator()

#Này phần ids để truy xuất dữ liệu nhất định
entity_ids = [
    "CMD-URDFUEUPC",
    "IND-MZG3VDMDS",
    "IND-LH0PMFUBT",
    "FOR-QMFF6NJKG",
    "EQ-0C00004EP4"
]

fx = "https://portal.fpmarketsint.com/sc-VN/login"
base_url = "https://site.recognia.com/fpmarkets/presto/marketbuzz_dashboard?sid=Pf4am3y9tSg4ZqXxmDYk%252BvHRsW8aKQNq&entityId={}&tcTab=news"

# Sử dụng Selenium để trích xuất thông tin từ trang web ở đây mình tạo cổng 9999
chrome_options = Options()
chrome_options.debugger_address = "127.0.0.1:9999"
driver = webdriver.Chrome(options=chrome_options)

# Lưu trữ tiêu đề đã gửi trong bộ nhớ (tạm thời)
sent_titles = set()

# Hàm kiểm tra tiêu đề đã gửi hay chưa
def check_title(title):
    if title in sent_titles:
        print("This title has already been sent. Skipping...")
        return False
    else:
        sent_titles.add(title)
        return True

# Hàm gửi yêu cầu POST với thông tin đăng nhập
def send_post_request():
    url = "https://portal.fpmarkets.com/sc-VN/login"
    data = {
        "_token": "",
        "email": "",
        "password": "",
        "user_pin": ""
    }
    headers = {
        "Content-Type": "application/x-www-form-urlencoded",
        "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
        "Accept-Encoding": "gzip, deflate, br, zstd",
        "Accept-Language": "en,en-US;q=0.9,vi;q=0.8",
        "Origin": "https://portal.fpmarkets.com",
        "Referer": "https://portal.fpmarkets.com/sc-VN/login",
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36 Edg/134.0.0.0",
    }

    response = requests.post(url, data=data, headers=headers)
    print(f"POST request sent. Status code: {response.status_code}")

# Hàm trích xuất dữ liệu
def extract_data(entity_id):
    # Tạo URL mới với entityId thay đổi
    url = base_url.format(entity_id)

    driver.get(fx)
    time.sleep(10)
    send_post_request()
    time.sleep(5)
    driver.get(url)
    time.sleep(5)

    # Đợi phần tử <div> có role='button' và class='tc-news-title ng-star-inserted' để nhấn
    wait = WebDriverWait(driver, 20)
    button = wait.until(EC.element_to_be_clickable((By.XPATH, "//div[@role='button' and contains(@class, 'tc-news-title ng-star-inserted')]")))

    # Nhấn vào phần tử
    button.click()

    # Đợi một chút để trang tải
    time.sleep(5)

    # Sau khi nhấn, tìm phần tử có lớp 'tc-news-reader-scroll' để trích xuất thông tin
    news_content = driver.find_element(By.XPATH, "//div[@class='tc-news-reader-scroll']")

    # Lấy "raw" HTML của phần tử
    raw_html = news_content.get_attribute("outerHTML")
    soup = BeautifulSoup(raw_html, 'html.parser')

    # Trích xuất tiêu đề bài viết
    title = soup.find('h3', class_='tc-news-reader-title').text.strip()

    # Trích xuất ngày tháng
    date = soup.find('span', class_='ng-star-inserted').text.strip()

    # Trích xuất nội dung bài viết
    content = soup.find('div', class_='tc-news-reader-content-section').find('p').text.strip()

    # Trích xuất link "Story Continues"
    link_element = soup.find('a', class_='tc-news-reader-article-link')
    story_link = link_element['href'] if link_element else None

    # Gửi yêu cầu POST sau khi trích xuất xong
    

    return title, date, content, story_link

# Hàm dịch tiêu đề, ngày tháng và nội dung
def translate_text(text, target_lang='vi'):
    try:
        # Dịch văn bản bằng Google Translate
        translated_text = translator.translate(text, dest=target_lang).text
        return translated_text
    except Exception as e:
        print(f"Error translating text: {e}")
        return text  # Trả về văn bản gốc nếu lỗi

# Hàm gửi tin nhắn
async def send_message():
    group_name = "@eluptktnews"  # Thay bằng tên nhóm hoặc username nhóm
    group = await client.get_entity(group_name)

    # Lặp qua các entityId và gửi tin nhắn
    while True:  # Vòng lặp vô hạn để duy trì chạy liên tục
        for entity_id in entity_ids:
            print(f"Processing entityId: {entity_id}")
            title, date, content, story_link = extract_data(entity_id)

            # Kiểm tra xem tiêu đề đã gửi chưa
            if not check_title(title):
                continue  # Nếu tiêu đề đã được gửi, bỏ qua và chuyển qua entityId tiếp theo

            # Dịch các phần cần thiết
            translated_title = translate_text(title)
            translated_date = translate_text(date)
            translated_content = translate_text(content)


            # Gửi tin nhắn vào nhóm
            await client.send_message(group, message, parse_mode='Markdown')
            print(f"Sent message: {message}")

            time.sleep(10)  # Đợi 10 giây trước khi xử lý entityId tiếp theo

        # Đợi một thời gian (10 giây) trước khi quay lại và xử lý lại từ đầu
        print("Waiting 10 seconds before restarting the process...")
        time.sleep(10)

# Hàm chính để khởi động client
async def main():
    # Đăng nhập vào client
    await client.start(phone_number)

    # Chạy client và gửi tin nhắn
    await send_message()

    # Đóng client khi xong
    await client.disconnect()

# Chạy hàm main
if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
