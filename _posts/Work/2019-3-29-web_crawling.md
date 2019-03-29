---
layout: post
title : requests와 BeautifulSoup으로 웹 크롤링
comments: true
categories: Python
---


### requests로 html 소스 가져오기

```python
import requests

## HTTP GET Request
req = requests.get('https://kaiser1227.github.io/')

## HTML 소스 가져오기
html = req.text
## HTTP Header 가져오기
header = req.headers
## HTTP Status 가져오기 (200: 정상)
status = req.status_code
## HTTP가 정상적으로 되었는지 (True/False)
is_ok = req.ok

html
```
### beautifulSoup로 html 소스 가져오기
```python
import requests
from bs4 import BeautifulSoup

## HTTP GET Request
req = requests.get('https://kaiser1227.github.io/')
## HTML 소스 가져오기
html = req.text
## BeautifulSoup으로 html소스를 python객체로 변환하기
## 첫 인자는 html소스코드, 두 번째 인자는 어떤 parser를 이용할지 명시.
## 이 글에서는 Python 내장 html.parser를 이용했다.
soup = BeautifulSoup(html, 'html.parser')

soup
```
### html 특정 태그 값 조회하기
```python
## parser.py
import requests
from bs4 import BeautifulSoup

req = requests.get('https://kaiser1227.github.io/')
html = req.text
soup = BeautifulSoup(html, 'html.parser')
## CSS Selector를 통해 html요소들을 찾아낸다.
my_titles = soup.select(
    'h2'
    )

print(my_titles)

## my_titles는 list 객체
for title in my_titles:
    ## Tag안의 텍스트
    print(title.text)
    ## Tag의 속성을 가져오기(ex: href속성)
    print(title.get('h2'))
```
