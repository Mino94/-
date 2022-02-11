Naver / Daum
포탈에 "연관" 키워드 검색시 연관 검색어 크롤링

BeautifulSoup
requests
[DBP본부] 운영/소개 > [소셜 채널 관리] 연관검색어 스크래핑 > 스크린샷 2022-02-07 오전 10.45.57.png

[DBP본부] 운영/소개 > [소셜 채널 관리] 연관검색어 스크래핑 > 스크린샷 2022-02-07 오전 10.49.09.png

재귀함수를 사용하여 Depth 1로 연관검색어를 스크래핑 한다.



```

def naver_crawling_related_word(word_list, n):
    if n==0:
        print("크롤링 종료")
    else:
        merge_df = pd.DataFrame()

        for i in range(len(word_list)):
            temp_df = pd.DataFrame()
            temp_list = []

            url = f'https://search.naver.com/search.naver?where=nexearch&query={word_list[i]}'

            response = requests.get(url)
            soup = BeautifulSoup(response.text, 'lxml')

            ul_inner = soup.find('ul', {'class':'lst_related_srch _list_box'})

            try:
                keyword_list = soup.find('ul', {'class':'lst_related_srch _list_box'}).find_all('li')
            except:
                if pd.isna(ul_inner):
                    try:
                        keyword_list = soup.find('ul', {'class':'lst_related_srch _list_box'}).find_all('li')
                    except:
                        keyword_list = []

            # span_list 가 공백일 때 예외 처리
            if len(keyword_list) == 0:
                pass
            else:
                for k, word in enumerate(keyword_list):
                    if len(keyword_list[k].text.replace(word_list[i],'').strip()) == 1:
                        temp_list.append(keyword_list[k].text.strip())
                    else:
                          temp_list.append(keyword_list[k].text.strip())

                temp_df['sub_word'] = temp_list
                temp_df['sup_word'] = word_list[i]
                temp_df = pd.DataFrame(temp_df, columns=['sup_word', 'sub_word'])

                merge_df = pd.concat([merge_df, temp_df], axis=0)
        naver_dictionary[n] = merge_df
        # 재귀호출
        pass_word_list = merge_df['sub_word'].values

        print('네이버 크롤링 시작')
        naver_crawling_related_word(pass_word_list, n-1)

def daum_crawling_related_word(word_list, n):
    if n==0:
        print("크롤링 종료")
    else:
        merge_df = pd.DataFrame()

        for i in range(len(word_list)):
            temp_df = pd.DataFrame()
            temp_list = []

            url = f'https://search.daum.net/search?w=tot&DA=YZR&t__nil_searchbox=btn&sug=&sugo=&q={word_list[i]}'

            response = requests.get(url)
            soup = BeautifulSoup(response.text, 'lxml')

            div_inner = soup.find('div', {'class':'list_keyword type2'})

            try:
                div_inner = soup.find('div', {'class':'list_keyword type2'})
                span_list = div_inner.find_all('span', {'class':'wsn'})
            except:
                if pd.isna(div_inner):
                    try:
                        div_inner = soup.find('div', {'class':'list_keyword'})
                        span_list = div_inner.find_all('span', {'class':'wsn'})
                    except:
                        span_list = []

            # span_list 가 공백일 때 예외 처리
            if len(span_list) == 0:
                pass
            else:
                for k, word in enumerate(span_list):
                    if len(span_list[k].text.replace(word_list[i],'').strip()) == 1:
                        temp_list.append(span_list[k].text)
                    else:
                        temp_list.append(span_list[k].text)

                temp_df['sub_word'] = temp_list
                temp_df['sup_word'] = word_list[i]
                temp_df = pd.DataFrame(temp_df, columns=['sup_word', 'sub_word'])

                merge_df = pd.concat([merge_df, temp_df], axis=0)

        daum_dictionary[n] = merge_df

        # 재귀호출
        pass_word_list = merge_df['sub_word'].values
        print('다음 크롤링 시작')
        daum_crawling_related_word(pass_word_list, n-1)  
```



Google
requests_html(https://docs.python-requests.org/projects/requests-html/en/latest/)
css flex box의 태그 값들을 크롤링 할 수있다. (google, bing)
[DBP본부] 운영/소개 > [소셜 채널 관리] 연관검색어 스크래핑 > 스크린샷 2022-02-07 오전 10.49.18.png [DBP본부] 운영/소개 > [소셜 채널 관리] 연관검색어 스크래핑 > 스크린샷 2022-02-07 오전 10.46.37.png


```
# google
import requests
import urllib
import pandas as pd
from requests_html import HTML
from requests_html import HTMLSession

def get_source(url):
    """Return the source code for the provided URL. 

    Args: 
        url (string): URL of the page to scrape.

    Returns:
        response (object): HTTP response object from requests_html. 
    """

    try:
        session = HTMLSession()
        response = session.get(url)
        return response

    except requests.exceptions.RequestException as e:
        print(e)

def get_results(query):
    query = urllib.parse.quote_plus(query)
    response = get_source("https://www.google.co.uk/search?q=" + query)
    
    return response

def parse_results(response):
    
    css_identifier_result = ".s75CSd"
    css_identifier_title = "b"
#     css_identifier_link = ".yuRUbf a"
#     css_identifier_text = ".IsZvec"
    
    results = response.html.find(css_identifier_result)

    output = []
    for result in results:
        text= result.find(css_identifier_title, first=True).text
        output.append(text)
    return output

def google_search(query):
    response = get_results(query)
    return parse_results(response)

keywords = ['더존비즈온']
for keyword in keywords:
    print(google_search(keyword))
    
go_li = google_search(keyword)
google_df = pd.DataFrame(go_li)
google_df['sup_word'] = keywords[0]
google_df.columns = ['sub_word1','sup_word']
google_df
```


연관 검색어 데이터 시각화 / DB 적재
Networx 라이브러리 사용하여 시각화
neo4j 사용 연관검색어 저장(조회수 반영이 안됨)



```
def netword_drawing(re_df):
    print("그리기 시작")
    G = nx.Graph()
    
    for sup in list(re_df['keyword'].values):
        G.add_node(sup)
        
        for sub1 in list(re_df['sub_word'].values):
            G.add_node(sub1)
            G.add_edge(sup, sub1)
            
#             for sub2 in list(set(sub_df2['keyword'].values)):
#                 G.add_node(sub2)
#                 G.add_edge(sub1, sub2)
                
    color_map = []
    for node in G:
        if node == re_df['keyword'].values[0]:
            color_map.append('red')
        else:
            color_map.append('green')      
    
    plt.figure(figsize=(20,20))
#     d = dict(G.degree(list(set(re_df['views']))))
#     pos = nx.shell_layout(G)
    d = dict(zip(re_df['sub_word'], re_df['views']))
    print(d)
    
       
    nx.draw(G, nodelist=list(d), node_size=[int(v)/30 for v in d.values()], edge_cmap=plt.cm.OrRd,font_family='AppleGothic', font_size=20, node_color = color_map, with_labels = True, font_weight='bold')

    plt.axis('off')
    plt.savefig('networx_picture_'+re_df['keyword'].values[0]+'.png')
    plt.show()

netword_drawing(re_df)

```





[DBP본부] 운영/소개 > [소셜 채널 관리] 연관검색어 스크래핑 > image2022-2-7_11-21-5.png
