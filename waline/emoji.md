## Waline Emoji

Waline的Emoji是可以自建的(见[创建自己的表情预设 | Waline](https://waline.js.org/cookbook/customize/emoji.html#编写预设信息)),如果想要把别人的Emoji下载下载的话，使用爬虫稍微方便一点

```python
import json
import os.path

import requests

header = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) '
}
# 需要获取的列表,/结尾
urls = [
          'https://unpkg.com/@waline/emojis@1.1.0/bilibili/',
          'https://unpkg.com/@waline/emojis@1.1.0/bmoji/',
          'https://unpkg.com/@waline/emojis@1.1.0/qq/',
          'https://unpkg.com/@waline/emojis@1.1.0/tieba/',
          'https://unpkg.com/@waline/emojis@1.1.0/tw-emoji/',
          'https://unpkg.com/@waline/emojis@1.1.0/weibo/',
        ]

# 获取Emoji的info.json
def get_emojis_info(url):
    dir = url.split('/')[-2]
    if not os.path.exists(dir):
        os.mkdir(dir)
    r = requests.get(url + 'info.json', headers=header)
    with open(dir + '/info.json', 'w', encoding='utf-8') as f:
        f.write(r.text)
    return r.json()

# 获取Emoji
def get_emojis(url, emoji_info):
    dir = url.split('/')[-2]
    if not os.path.exists(dir):
        os.mkdir(dir)
    prefix = emoji_info['prefix']
    types = emoji_info['type']
    items = emoji_info['items']
    for item in items:
        filename = prefix + item + '.' + types
        r = requests.get(url + filename, headers=header)
        with open(dir + '/' + filename, 'wb') as f:
            f.write(r.content)

# 保存README文件和许可证(可选)
def save_info(url):
    dir = url.split('/')[-2]
    r_url = url + 'LICENSE'
    r = requests.get(r_url, headers=header)
    with open(dir+'/LICENSE', 'w', encoding='utf-8') as f:
        f.write(r.text)
    r_url = url + 'README.md'
    r = requests.get(r_url, headers=header)
    with open(dir+'/README.md', 'w', encoding='utf-8') as f:
        f.write(r.text)

if __name__ == '__main__':
    for url in urls:
        emoji_info = get_emojis_info(url)
        get_emojis(url, emoji_info)
        save_info(url)

```

