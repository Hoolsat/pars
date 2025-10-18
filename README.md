# parsv2
import requests
from bs4 import BeautifulSoup
import json
import datetime
import time

class NewsParser:
    def __init__(self):
        self.sources = {
            'habr': 'https://habr.com/ru/rss/all/all/',
            'lenta': 'https://lenta.ru/rss/news',
            'bbc': 'https://feeds.bbci.co.uk/russian/rss.xml'
        }
    
    def parse_rss(self, url):
        """Парсит RSS ленту"""
        try:
            headers = {
                'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
            }
            response = requests.get(url, headers=headers, timeout=10)
            soup = BeautifulSoup(response.content, 'xml')
            
            news_items = []
            for item in soup.find_all('item')[:10]:  # Берем 10 последних новостей
                title = item.title.text if item.title else 'Без названия'
                link = item.link.text if item.link else '#'
                description = item.description.text if item.description else ''
                pub_date = item.pubDate.text if item.pubDate else ''
                
                news_items.append({
                    'title': title,
                    'link': link,
                    'description': description[:200] + '...' if len(description) > 200 else description,
                    'pub_date': pub_date,
                    'source': url
                })
            
            return news_items
            
        except Exception as e:
            print(f"Ошибка парсинга {url}: {e}")
            return []
    
    def get_all_news(self):
        """Получает новости со всех источников"""
        all_news = []
        
        for source_name, source_url in self.sources.items():
            print(f"📡 Получаем новости с {source_name}...")
            news = self.parse_rss(source_url)
            all_news.extend(news)
            time.sleep(1)  # Пауза между запросами
        
        # Сортируем по дате (новые сначала)
        all_news.sort(key=lambda x: x['pub_date'], reverse=True)
        return all_news
    
    def save_to_json(self, news_items, filename=None):
        """Сохраняет новости в JSON файл"""
        if not filename:
            timestamp = datetime.datetime.now().strftime("%Y%m%d_%H%M")
            filename = f"news_{timestamp}.json"
        
        with open(filename, 'w', encoding='utf-8') as f:
            json.dump(news_items, f, ensure_ascii=False, indent=2)
        
        print(f"💾 Новости сохранены в {filename}")
    
    def display_news(self, news_items):
        """Отображает новости в консоли"""
        print(f"\n📰 Последние новости ({len(news_items)} шт.)")
        print("=" * 50)
        
        for i, news in enumerate(news_items, 1):
            print(f"{i}. {news['title']}")
            print(f"   📅 {news['pub_date']}")
            print(f"   🔗 {news['link']}")
            print(f"   📝 {news['description']}")
            print()

def main():
    parser = NewsParser()
    
    while True:
        print("\n=== Парсер новостей ===")
        print("1. Получить свежие новости")
        print("2. Сохранить в файл")
        print("3. Выйти")
        
        choice = input("Выберите действие: ")
        
        if choice == "1":
            news = parser.get_all_news()
            parser.display_news(news)
            
            save = input("Сохранить в файл? (y/n): ")
            if save.lower() == 'y':
                parser.save_to_json(news)
                
        elif choice == "2":
            news = parser.get_all_news()
            parser.save_to_json(news)
            
        elif choice == "3":
            print("До свидания!")
            break
        else:
            print("Неверный выбор!")

if __name__ == "__main__":
    main()
