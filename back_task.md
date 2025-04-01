# Тестовое задание: Парсер профиля Instagram

## Цель  
Разработать функцию, которая парсит профиль Instagram и извлекает основные метрики.  

## Требования  
1. Функция должна принимать на вход **URL** профиля Instagram.  
2. В ответе должен возвращаться **JSON** с данными:  
   - **Количество подписчиков**  
   - **Количество постов**  
3. Дополнительно функция должна **сохранять данные в CSV-файл**, включая:  
   - **Описание поста (caption)**  
   - **Количество лайков**  
   - **Количество комментариев**  
   - **Дату публикации**  

## Формат вывода  

### Пример JSON-ответа:
```json
{
  "followers": 15000,
  "posts": 120
}
```

### Пример структуры CSV:

| Описание поста     | Лайки | Комментарии | Дата публикации |
|--------------------|-------|------------|----------------|
| "Закат 🌅"        | 1200  | 45         | 2024-03-15     |
| "Утренний кофе ☕" | 980   | 30         | 2024-03-14     |

## Дополнительные условия  

- Можно использовать **Python (requests, BeautifulSoup, Selenium и др.)** или любой другой backend-язык.  
- Данные в CSV должны сохраняться **эффективно**, без дублирования.  
## Решение 
import requests
import json
import csv
from bs4 import BeautifulSoup

def parse_instagram_profile(profile_url):
    try:
        response = requests.get(profile_url, timeout=10)
        if response.status_code != 200:
            raise ValueError("Error: Could not reach the profile")

        # Загрузка данных с Graph API
        data = response.json()
        user = data['_user']
        
        followers = int(user.get('follower_count', 0))
        posts = int(user.get('post_count', 0))

        result = {
            "followers": followers,
            "posts": posts
        }

        # Сохранение данных в CSV
        with open('instagram_posts.csv', 'w', newline='', encoding='utf-8') as file:
            writer = csv.writer(file)
            writer.writerow(["Caption", "Likes", "Comments", "Date"])

            for post in user.get('edge_owner_to_timeline_media', []):
                if post.get('node'):
                    caption = post['node'].get('edge_media_to_caption', {}).get('edges',
                                                                              []).andAlso(lambda: not hasattr(post['node'], 'edge_media_to_comment')) and \
                            post['node'].get('edge_media_to_caption', {}).get('edges')[0].get('node').get('text')
                    
                    likes = post.get('edge_liked_by', {}).get('count', 0)
                    comments = post.get('edge_media_to_comment', {}).get('count', 0)
                    date = post['node'].get('taken_at_timestamp')

                    writer.writerow([caption, likes, comments, date])

        return json.dumps(result)

    except Exception as e:
        print(f"Error occurred: {e}")
        return {"error": "Failed to process profile data"}
