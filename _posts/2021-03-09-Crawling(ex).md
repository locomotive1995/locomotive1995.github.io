# 네이버 뉴스 데이터 가져오기

> 1. 네이버 오픈 API로 기사 URL 크롤링
> 2. bs4 + selenium으로 기사 제목 및 내용 크롤링


```python
!pip install selenium
```

    Collecting selenium
      Using cached selenium-3.141.0-py2.py3-none-any.whl (904 kB)
    Requirement already satisfied: urllib3 in c:\users\gur95\anaconda3\envs\test\lib\site-packages (from selenium) (1.25.9)
    Installing collected packages: selenium
    Successfully installed selenium-3.141.0
    


```python
import os
import sys
import urllib.request
import requests

news_data = []
page_count = 3

client_id = "b_LrFl9tpgS1zHKDc8C5"
client_secret = "gFBdayr4YI"
encText = urllib.parse.quote("파이썬") #검색할 단어 입력, 실제 url에 붙여주는 역할

for idx in range(page_count):
    # json 결과
    url = "https://openapi.naver.com/v1/search/news?query=" + encText + "&start=" + str(idx * 10 + 1)
    # url = "https://openapi.naver.com/v1/search/blog.xml?query=" + encText # xml 결과
    
    request = urllib.request.Request(url)
    request.add_header("X-Naver-Client-Id",client_id) #client id랑
    request.add_header("X-Naver-Client-Secret",client_secret) #client 비밀번호 기입
   #request 헤더에 아이디와 비밀번호 전달
    response = urllib.request.urlopen(request)#접속 완료
    rescode = response.getcode()#코드를 받아옴
#200이면 제대로 된거, 코드로 송수인이 제대로 된건지 확인하는 절차
    
    if(rescode==200):
    #    response_body = response.read()
        result = requests.get(response.geturl(),
                              headers={"X-Naver-Client-Id":client_id,
                                       "X-Naver-Client-Secret":client_secret}
                             )
        news_data.append(result.json())
    #    print(response_body.decode('utf-8'))
    else:
        print("Error Code:" + rescode)
        #requests.get : url을 던저ㅈ주면 그 url안에 내용을 가져와 주는 함수
        #파이썬에서 .json은 딕셔너리 처럼 만들어주는?
```


```python
#print(news_data)
print(len(news_data))
#print(news_data[0])
#print(news_data[0]['items'])
#print(len(news_data[0]['items']))
#print(news_data[0]['items'][2])
print(news_data[0]['items'][7]['link'])
```

    3
    http://www.cctoday.co.kr/news/articleView.html?idxno=2106110
    


```python
#mobile로 접속하면 크롤링이 쉬움

```

## 가져온 URL이 네이버 뉴스인지 확인하기.


```python
naver_news_link = []

#정확히는 json으로 만든 거에서 하나씩 들고옴
#페이지별로 들고온 내용을 구분지을 거임 (2차원 행렬화)
for page in news_data:
    #print(page)
    page_news_link = []
    #for page이므로 page['items']
    for item in page['items']:
        #print(item)
        temp_link = item['link']
        #print(temp_link)
        if "naver" in temp_link: #naver뉴스만 사용하겠다
            page_news_link.append(temp_link)
    
    naver_news_link.append(page_news_link)
        

# 사이트 확인하기에 편한 코드 구조.
for page in naver_news_link:
    for link in page:
        print(link)
```

    https://news.naver.com/main/read.nhn?mode=LSD&mid=sec&sid1=101&oid=008&aid=0004554472
    https://news.naver.com/main/read.nhn?mode=LSD&mid=sec&sid1=101&oid=082&aid=0001073360
    https://news.naver.com/main/read.nhn?mode=LSD&mid=sec&sid1=105&oid=092&aid=0002215560
    https://news.naver.com/main/read.nhn?mode=LSD&mid=sec&sid1=101&oid=011&aid=0003880292
    https://news.naver.com/main/read.nhn?mode=LSD&mid=sec&sid1=102&oid=421&aid=0005207416
    https://news.naver.com/main/read.nhn?mode=LSD&mid=sec&sid1=101&oid=009&aid=0004758616
    https://news.naver.com/main/read.nhn?mode=LSD&mid=sec&sid1=105&oid=138&aid=0002099683
    https://news.naver.com/main/read.nhn?mode=LSD&mid=sec&sid1=102&oid=022&aid=0003558167
    


```python
import pandas as pd
import numpy as np
from selenium import webdriver
from tqdm import tqdm_notebook
import requests
import pickle
import re
import ast

from bs4 import BeautifulSoup 
from urllib.request import urlopen
import urllib
import time
```

    C:\Users\gur95\anaconda3\envs\test\lib\site-packages\selenium\webdriver\firefox\firefox_profile.py:208: SyntaxWarning: "is" with a literal. Did you mean "=="?
      if setting is None or setting is '':
    


```python
# 가상 크롬드라이버를 불러옴.
# 윈도우 10의 경우 chromedriver.exe
driver = webdriver.Chrome('C:/Users/gur95/Desktop/croling/chromedriver.exe')
```

## 크롤링 결과 확인하기


```python
naver_news_title = []
naver_news_content = []


for n in tqdm_notebook(range(len(naver_news_link))):
    #print(n)
    news_page_title = []
    news_page_content = []
    
    for idx in tqdm_notebook(range(len(naver_news_link[n]))):
        
        
    ########### 긁어온 URL로 접속하기 ############    
        try:
            driver.get(naver_news_link[n][idx])
            print(naver_news_link[n][idx])
            
        except:
            print("Timeout!")
            continue
        #try except 중간에 빽 날까
        
        try:
            response = driver.page_source
            
        except UnexpectedAlertPresentException:
            driver.switch_to_alert().accept()
            print("게시글이 삭제된 경우입니다.")
            continue
        
        soup = BeautifulSoup(response, "html.parser")
        
        ###### 뉴스 타이틀 긁어오기 ######
        
        title = None
        
        try:
            item = soup.find('div', class_="article_info")
            title = item.find('h3', class_="tts_head").get_text()
            #print(title)

        except:
            title = "OUTLINK"
        
        #print(title)
        news_page_title.append(title)
        
        
        ###### 뉴스 본문 긁어오기 ######
        
        doc = None
        text = ""
        #바디를 긁어옴        
        data = soup.find_all("div", {"class" : "_article_body_contents"})
        if data:
            for item in data:

                text = text + str(item.find_all(text=True)).strip()
                text = ast.literal_eval(text)#ast는 복잡.. 출력방식을 깔끔하게 바꾸기 위해 사용
                doc = ' '.join(text)
   
        else:
            doc = "OUTLINK"
            
        news_page_content.append(doc.replace('\n', ' '))

                
    naver_news_title.append(news_page_title)
    naver_news_content.append(news_page_content)

    time.sleep(2)#막힐까봐
    
    
print(naver_news_title[0])
print("==================================")
print(naver_news_content[0])
```

    <ipython-input-8-b9e698756c31>:5: TqdmDeprecationWarning: This function will be removed in tqdm==5.0.0
    Please use `tqdm.notebook.tqdm` instead of `tqdm.tqdm_notebook`
      for n in tqdm_notebook(range(len(naver_news_link))):
    


    HBox(children=(FloatProgress(value=0.0, max=3.0), HTML(value='')))


    <ipython-input-8-b9e698756c31>:10: TqdmDeprecationWarning: This function will be removed in tqdm==5.0.0
    Please use `tqdm.notebook.tqdm` instead of `tqdm.tqdm_notebook`
      for idx in tqdm_notebook(range(len(naver_news_link[n]))):
    


    HBox(children=(FloatProgress(value=0.0, max=3.0), HTML(value='')))


    https://news.naver.com/main/read.nhn?mode=LSD&mid=sec&sid1=101&oid=008&aid=0004554472
    https://news.naver.com/main/read.nhn?mode=LSD&mid=sec&sid1=101&oid=082&aid=0001073360
    https://news.naver.com/main/read.nhn?mode=LSD&mid=sec&sid1=105&oid=092&aid=0002215560
    
    


    HBox(children=(FloatProgress(value=0.0, max=3.0), HTML(value='')))


    https://news.naver.com/main/read.nhn?mode=LSD&mid=sec&sid1=101&oid=011&aid=0003880292
    https://news.naver.com/main/read.nhn?mode=LSD&mid=sec&sid1=102&oid=421&aid=0005207416
    https://news.naver.com/main/read.nhn?mode=LSD&mid=sec&sid1=101&oid=009&aid=0004758616
    
    


    HBox(children=(FloatProgress(value=0.0, max=2.0), HTML(value='')))


    https://news.naver.com/main/read.nhn?mode=LSD&mid=sec&sid1=105&oid=138&aid=0002099683
    https://news.naver.com/main/read.nhn?mode=LSD&mid=sec&sid1=102&oid=022&aid=0003558167
    
    
    ['코로나 한파에도…AI·빅데이터 채용공고는 38%↑', '통계청장-유엔통계처장 영상회의…유엔글로벌 플랫폼 참여 논의', 'C언어 여전히 인기… 어셈블리·비주얼 베이직도 각광']
    ==================================
    ["   본문 내용     TV플레이어     // TV플레이어     // flash 오류를 우회하기 위한 함수 추가 function _flash_removeCallback() {}   \t \t[머니투데이 이재윤 기자]   구인구직 매칭 플랫폼 사람인은 지난해 자사등록 직종별 채용 공고를 분석한 결과 전년대비 전 직종에서 지원공고가 감소했다고 9일 밝혔다.  특히 △서비스(-36.3%p) △교육(-28.2%p) △영업·고객상담(-21.4%p) 분야에서 감소폭이 컸다. 서비스 분야에서는 △여행·관광·항공(-73.6%p) △호텔·카지노·콘도(-52.1%p) △안내·도우미·나레이터(-51.6%p) △미용·피부관리·애견(-44.8%p) △레저·스포츠(-41.1%p) 직종의 감소폭이 두드러졌다. 다만 돌봄 부족 등으로 가사·청소·육아 직종은 오히려 2019년 대비 4.4%p 성장했다. 감소폭이 적은 직종은 △특수계층·공공(-7.7%p) △IT·인터넷(-9%p) △디자인(-14.6%p) △미디어(-15.3%p) △전문직(-15.4%P) 등으로 조사됐다. 특히 IT·인터넷 직종에선 △AI·빅데이터(+28.8%p) △동영상·편집·코덱(+18.4%p) 분야 공고가 눈에 띄게 증가했다. 특히 AI·빅데이터 분야 소분류 상에서도 △파이썬(+51.5%p) △머신러닝(+45.2%p) △AI(+45.1%p) 등에서 두 자리수의 증가폭을 보였다. 교육 분야에서는 대면이 필수적인 △학습지·과외·방문(-43.3%p)이나 △유치원·보육(-39.6%p) △초중고·특수학교(-33.4%p) △외국어·어학원(-30.3%p) 직종이 크게 감소했다. 임민욱 사람인 팀장은 “산업 전반에 걸친 변화가 직종별 채용에 미치는 영향이 큰 만큼 평소 경제 동향이나 향후 성장성이 높은 직종이 무엇인지에 대해서도 꾸준히 관심을 가지는 것이 필요하다.”라고 말했다. article_split 이재윤 기자 mton@mt.co.kr ▶부동산 투자는 [부릿지] ▶조 변호사의 가정상담소 ▶줄리아 투자노트   <저작권자 ⓒ '돈이 보이는 리얼타임 뉴스' 머니투데이, 무단전재 및 재배포 금지> \t  // 본문 내용   ", '   본문 내용     TV플레이어     // TV플레이어     // flash 오류를 우회하기 위한 함수 추가 function _flash_removeCallback() {}    류근관 통계청장이 9일 오전 9시에 스테판 슈바인페스트 유엔통계처장(오른쪽)과의 영상회의에서 한국 통계청-유엔과의 협력 확대에 관한 의견을 교환하고 있다. 통계청 제공 류근관 통계청장은 9일 오전(한국시간) 스테판 슈바인페스트 유엔통계처장과 영상회의를 갖고 기관 간 협력 확대 방안에 대한 의견을 교환했다. 한국통계청과 유엔통계처는  2010 년부터 매년 국제회의를 공동 개최해 오고 있으며, 올해는 ‘위기 상황에서 자료와 통계사회의 역할’이라는 주제로 오는 8월  30 일부터 9월 1일까지 개최할 예정이다. 통계청은 유엔통계처가 전세계 통계사회의 협업을 위해 구축하고 있는 유엔 글로벌 플랫폼( UNGP )에 영국·네덜란드·아랍에미리트 통계청과 공동 참여하는 등 다양한 방식의 협업체계를 모색하고 있다. UNGP 란 새로운 데이터 소스의 공식통계 활용 및 지속가능발전목표 이행을 위한 국제공조 지원 클라우드 서비스 시스템을 말한다. 선박데이터, 항공데이터, 위성 데이터, 모바일 데이터, 스캐너 데이터 등에 대해 웹 상에서 사용이 가능하고 R, 파이썬, 줄리아 등 다양한 언어를 제공한다. 슈바인페스트 통계처장은 류청장이 지난 1일 제 52 차 유엔통계위원회에서 소개한 ‘ K- 통계시스템’ 구축계획을 환영하며 개인정보 보호, 민간 빅데이터 활용 등 통계청의 고유한 기술과 경험이  UNGP  구축에도 크게 도움될 것이라며 적극적 공유와 지원을 요청했다. 류청장은  UNGP  참여는 새로운 빅데이터 서비스 플랫폼을 경험하는 기회로  K- 통계시스템 업그레이드에도 기여할 것이라며 양 기관의 혁신적 사업을 성공적으로 추진하기 위한 지속적 공조와 상호 협력체계 구축을 제안했다. 김덕준 기자  casiopea @ busan.com ▶ 네이버에서 부산일보 구독하기 클릭! ▶ 부산닷컴 회원가입. 회원 전환하면 부산일보 지면보기 무료이벤트 ▶ 부산일보 홈 바로가기    // 본문 내용   ', "   본문 내용     TV플레이어     // TV플레이어     // flash 오류를 우회하기 위한 함수 추가 function _flash_removeCallback() {}    티오베, 3월 프로그래밍 커뮤니티 지수 발표 (지디넷코리아=남혁우  기자)C언어를 비롯해 고전 프로그래밍 언어가 최근 주목받고 있다. 소스코드 품질평가 기업인 티오베는 프로그래밍 언어순위인 3월 티오베 프로그래밍 커뮤니티 지수(티오베 인덱스)를 8일(현지시간) 발표했다. 이번 달에는 C언어를 비롯해 어셈블리, 클래식 비주얼 베이직 등 고전 프로그래밍 언어가 주목받았다. 티오베 인덱스 3월 프로그래밍 언어 인기 순위(이미지=티오베) C언어는  15.33 %의 비율을 기록하며 지난달에 이어 1위를 유지했다. 코로나 19 가 발생한 지난해부터 꾸준히 높은 관심을 얻는 추세다. 어셈블리는  12 위에서 9위, 클래식 비주얼 베이직은  18 위에서  12 위, 델파이는  20 위에서  14 위로 상승했다. 고전 프로그래밍 언어의 인기는 지난해 코로나 19 로 인한 것으로 분석된다. 기업 및 공공 부서의 업무 급증으로 기존에 활용하지 않던 낙후된 전산 시스템의 업무 비중이 커졌기 때문이다. 또한 C언어는 해당 언어를 주로 활용한 의료기기의 사용증가 때문이라는 분석도 나오고 있다. 코로나 19 가 지속되는 기간 동안에는 C언어의 관심도 이어질 전망이다. 한편, 티오베 인덱스 순위 2위는  10.45 %의 기록한 자바가 올랐다. 지난해 1위를 유지해온 자바는  7.33 %라는 큰 하락폭을 보이며 2위를 기록했다. 인공지능, 데이터과학 분야에서 주로 활용되는 파이썬은 3위를  10.31 %를 차지했다. 이어서 C++과 C#이  6.52 %와  4.97 %로 4위와 5위를 유지했다. 티오베 인덱스는 구글, 야후, 아마존, 바이두, 유튜브 등 검색 엔진을 통해 검색된 수치에 특정 공식을 대입해 등급을 나누는 방식을 적용하고 있다. 남혁우 기자( firstblood @ zdnet.co.kr )   ▶ 지디넷코리아 '홈페이지'   ▶ 네이버 채널 구독하기 © 메가뉴스 &  ZDNET , A  RED   VENTURES   COMPANY , 무단전재-재배포 금지 \t  // 본문 내용   "]
    


```python
print(naver_news_title[0])
```

    ['코로나 한파에도…AI·빅데이터 채용공고는 38%↑', '통계청장-유엔통계처장 영상회의…유엔글로벌 플랫폼 참여 논의', 'C언어 여전히 인기… 어셈블리·비주얼 베이직도 각광']
    


```python
print(naver_news_content[0])
```

    ["   본문 내용     TV플레이어     // TV플레이어     // flash 오류를 우회하기 위한 함수 추가 function _flash_removeCallback() {}   \t \t[머니투데이 이재윤 기자]   구인구직 매칭 플랫폼 사람인은 지난해 자사등록 직종별 채용 공고를 분석한 결과 전년대비 전 직종에서 지원공고가 감소했다고 9일 밝혔다.  특히 △서비스(-36.3%p) △교육(-28.2%p) △영업·고객상담(-21.4%p) 분야에서 감소폭이 컸다. 서비스 분야에서는 △여행·관광·항공(-73.6%p) △호텔·카지노·콘도(-52.1%p) △안내·도우미·나레이터(-51.6%p) △미용·피부관리·애견(-44.8%p) △레저·스포츠(-41.1%p) 직종의 감소폭이 두드러졌다. 다만 돌봄 부족 등으로 가사·청소·육아 직종은 오히려 2019년 대비 4.4%p 성장했다. 감소폭이 적은 직종은 △특수계층·공공(-7.7%p) △IT·인터넷(-9%p) △디자인(-14.6%p) △미디어(-15.3%p) △전문직(-15.4%P) 등으로 조사됐다. 특히 IT·인터넷 직종에선 △AI·빅데이터(+28.8%p) △동영상·편집·코덱(+18.4%p) 분야 공고가 눈에 띄게 증가했다. 특히 AI·빅데이터 분야 소분류 상에서도 △파이썬(+51.5%p) △머신러닝(+45.2%p) △AI(+45.1%p) 등에서 두 자리수의 증가폭을 보였다. 교육 분야에서는 대면이 필수적인 △학습지·과외·방문(-43.3%p)이나 △유치원·보육(-39.6%p) △초중고·특수학교(-33.4%p) △외국어·어학원(-30.3%p) 직종이 크게 감소했다. 임민욱 사람인 팀장은 “산업 전반에 걸친 변화가 직종별 채용에 미치는 영향이 큰 만큼 평소 경제 동향이나 향후 성장성이 높은 직종이 무엇인지에 대해서도 꾸준히 관심을 가지는 것이 필요하다.”라고 말했다. article_split 이재윤 기자 mton@mt.co.kr ▶부동산 투자는 [부릿지] ▶조 변호사의 가정상담소 ▶줄리아 투자노트   <저작권자 ⓒ '돈이 보이는 리얼타임 뉴스' 머니투데이, 무단전재 및 재배포 금지> \t  // 본문 내용   ", '   본문 내용     TV플레이어     // TV플레이어     // flash 오류를 우회하기 위한 함수 추가 function _flash_removeCallback() {}    류근관 통계청장이 9일 오전 9시에 스테판 슈바인페스트 유엔통계처장(오른쪽)과의 영상회의에서 한국 통계청-유엔과의 협력 확대에 관한 의견을 교환하고 있다. 통계청 제공 류근관 통계청장은 9일 오전(한국시간) 스테판 슈바인페스트 유엔통계처장과 영상회의를 갖고 기관 간 협력 확대 방안에 대한 의견을 교환했다. 한국통계청과 유엔통계처는  2010 년부터 매년 국제회의를 공동 개최해 오고 있으며, 올해는 ‘위기 상황에서 자료와 통계사회의 역할’이라는 주제로 오는 8월  30 일부터 9월 1일까지 개최할 예정이다. 통계청은 유엔통계처가 전세계 통계사회의 협업을 위해 구축하고 있는 유엔 글로벌 플랫폼( UNGP )에 영국·네덜란드·아랍에미리트 통계청과 공동 참여하는 등 다양한 방식의 협업체계를 모색하고 있다. UNGP 란 새로운 데이터 소스의 공식통계 활용 및 지속가능발전목표 이행을 위한 국제공조 지원 클라우드 서비스 시스템을 말한다. 선박데이터, 항공데이터, 위성 데이터, 모바일 데이터, 스캐너 데이터 등에 대해 웹 상에서 사용이 가능하고 R, 파이썬, 줄리아 등 다양한 언어를 제공한다. 슈바인페스트 통계처장은 류청장이 지난 1일 제 52 차 유엔통계위원회에서 소개한 ‘ K- 통계시스템’ 구축계획을 환영하며 개인정보 보호, 민간 빅데이터 활용 등 통계청의 고유한 기술과 경험이  UNGP  구축에도 크게 도움될 것이라며 적극적 공유와 지원을 요청했다. 류청장은  UNGP  참여는 새로운 빅데이터 서비스 플랫폼을 경험하는 기회로  K- 통계시스템 업그레이드에도 기여할 것이라며 양 기관의 혁신적 사업을 성공적으로 추진하기 위한 지속적 공조와 상호 협력체계 구축을 제안했다. 김덕준 기자  casiopea @ busan.com ▶ 네이버에서 부산일보 구독하기 클릭! ▶ 부산닷컴 회원가입. 회원 전환하면 부산일보 지면보기 무료이벤트 ▶ 부산일보 홈 바로가기    // 본문 내용   ', "   본문 내용     TV플레이어     // TV플레이어     // flash 오류를 우회하기 위한 함수 추가 function _flash_removeCallback() {}    티오베, 3월 프로그래밍 커뮤니티 지수 발표 (지디넷코리아=남혁우  기자)C언어를 비롯해 고전 프로그래밍 언어가 최근 주목받고 있다. 소스코드 품질평가 기업인 티오베는 프로그래밍 언어순위인 3월 티오베 프로그래밍 커뮤니티 지수(티오베 인덱스)를 8일(현지시간) 발표했다. 이번 달에는 C언어를 비롯해 어셈블리, 클래식 비주얼 베이직 등 고전 프로그래밍 언어가 주목받았다. 티오베 인덱스 3월 프로그래밍 언어 인기 순위(이미지=티오베) C언어는  15.33 %의 비율을 기록하며 지난달에 이어 1위를 유지했다. 코로나 19 가 발생한 지난해부터 꾸준히 높은 관심을 얻는 추세다. 어셈블리는  12 위에서 9위, 클래식 비주얼 베이직은  18 위에서  12 위, 델파이는  20 위에서  14 위로 상승했다. 고전 프로그래밍 언어의 인기는 지난해 코로나 19 로 인한 것으로 분석된다. 기업 및 공공 부서의 업무 급증으로 기존에 활용하지 않던 낙후된 전산 시스템의 업무 비중이 커졌기 때문이다. 또한 C언어는 해당 언어를 주로 활용한 의료기기의 사용증가 때문이라는 분석도 나오고 있다. 코로나 19 가 지속되는 기간 동안에는 C언어의 관심도 이어질 전망이다. 한편, 티오베 인덱스 순위 2위는  10.45 %의 기록한 자바가 올랐다. 지난해 1위를 유지해온 자바는  7.33 %라는 큰 하락폭을 보이며 2위를 기록했다. 인공지능, 데이터과학 분야에서 주로 활용되는 파이썬은 3위를  10.31 %를 차지했다. 이어서 C++과 C#이  6.52 %와  4.97 %로 4위와 5위를 유지했다. 티오베 인덱스는 구글, 야후, 아마존, 바이두, 유튜브 등 검색 엔진을 통해 검색된 수치에 특정 공식을 대입해 등급을 나누는 방식을 적용하고 있다. 남혁우 기자( firstblood @ zdnet.co.kr )   ▶ 지디넷코리아 '홈페이지'   ▶ 네이버 채널 구독하기 © 메가뉴스 &  ZDNET , A  RED   VENTURES   COMPANY , 무단전재-재배포 금지 \t  // 본문 내용   "]
    


```python
print(len(naver_news_title[0]))
print(len(naver_news_content[0]))
```

    2
    2
    

## 크롤링 피클로 저장하기


```python
with open("naver_news_title.pk", "wb") as f:
    pickle.dump(naver_news_title, f)
    
with open("naver_news_content.pk", "wb") as f:
    pickle.dump(naver_news_content, f)
```


```python

```
