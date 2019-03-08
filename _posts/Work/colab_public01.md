---
layout: post
title : 구글 colab, 구글 drive를 이용하여 공공데이터 가져오기
comments: true
categories: Work
---

```python
# 구글드라이브 마운트

from google.colab import drive
drive.mount('/content/gdrive')

```


    ---------------------------------------------------------------------------

    ModuleNotFoundError                       Traceback (most recent call last)

    <ipython-input-1-a9f87284c043> in <module>
          1 # 구글드라이브 마운트
          2 
    ----> 3 from google.colab import drive
          4 drive.mount('/content/gdrive')


    ModuleNotFoundError: No module named 'google.colab'



```python
# 구글드라이브 마운트 확인
ll /content/gdrive/"My Drive"/"Colab Notebooks"
```


```python

from urllib.request import urlopen
import pandas as pd
from bs4 import BeautifulSoup
import sqlite3
# from html_table_parser import parser_functions as parser
 
 
def collect_land(ym,lawd_cd):
 
    API_KEY = "UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D"
    url="http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev"
 
    url=url+"?LAWD_CD="+lawd_cd+"&DEAL_YMD="+ym+"&numOfRows=1000000"+"&serviceKey="+API_KEY
        
    resultXML = urlopen(url)
    result = resultXML.read()
    xmlsoup = BeautifulSoup(result, 'lxml-xml')
 
    te=xmlsoup.findAll("item")
    sil=pd.DataFrame()
    
    print(url)
    
    for t in te:
        try:        
          거래금액 = t.find("거래금액").text
          건축년도 = t.find("건축년도").text
          년 = t.find("년").text
          도로명 = t.find("도로명").text
          도로명건물본번호코드 = t.find("도로명건물본번호코드").text
          도로명건물부번호코드 = t.find("도로명건물부번호코드").text
          도로명시군구코드 = t.find("도로명시군구코드").text
          도로명코드 = t.find("도로명코드").text
          법정동 = t.find("법정동").text
          법정동본번코드 = t.find("법정동본번코드").text
          법정동부번코드 = t.find("법정동부번코드").text
          법정동시군구코드 = t.find("법정동시군구코드").text
          법정동읍면동코드 = t.find("법정동읍면동코드").text
          법정동지번코드 = t.find("법정동지번코드").text
          아파트 = t.find("아파트").text
          월 = t.find("월").text
          일 = t.find("일").text
          전용면적 = t.find("전용면적").text
          지역코드 = t.find("지역코드").text
          층 = t.find("층").text        
          지번 = t.find("지번").text
                
          temp = pd.DataFrame(([[
              거래금액
              ,건축년도
              ,년
              ,도로명
              ,도로명건물본번호코드
              ,도로명건물부번호코드
              ,도로명시군구코드
              ,도로명코드
              ,법정동
              ,법정동본번코드 
              ,법정동부번코드 
              ,법정동시군구코드
              ,법정동읍면동코드
              ,법정동지번코드 
              ,아파트
              ,월
              ,일
              ,전용면적
              ,지번
              ,지역코드
              ,층
          ]]),
                              columns=[
                                  "거래금액"
                                  ,"건축년도"
                                  ,"년"
                                  ,"도로명"
                                  ,"도로명건물본번호코드"
                                  ,"도로명건물부번호코드"
                                  ,"도로명시군구코드"
                                  ,"도로명코드"
                                  ,"법정동"
                                  ,"법정동본번코드"
                                  ,"법정동부번코드"
                                  ,"법정동시군구코드"
                                  ,"법정동읍면동코드"
                                  ,"법정동지번코드"
                                  ,"아파트"
                                  ,"월"
                                  ,"일"
                                  ,"전용면적"
                                  ,"지번"
                                  ,"지역코드"
                                  ,"층"
                              ])
          sil=pd.concat([sil,temp])

        except:
          pass
     
    sil=sil.reset_index(drop=True)
    
    return sil


def sqlite_append(sil_trade):
  con = sqlite3.connect("real_trade.db")

  sil_trade.to_sql('lent', con, if_exists='append', index=False)

  con.close()
  
  
if __name__=="__main__":
 

    # 지역코드 가져오기
    # https://www.mois.go.kr/frt/bbs/type001/commonSelectBoardArticle.do?bbsId=BBSMSTR_000000000052&nttId=61552


    code=pd.read_excel("KIKcd_B.20180122.xlsx")



    code_seo=code[(code["시도명"]=="서울특별시")] #| (code["시도명"]=="경기도")]

    code_seo = code_seo[code_seo["시군구명"].isnull() == False]

    code_seo = code_seo[code_seo["읍면동명"].isnull() == True]
    
    #code_seo = code_seo[code_seo["법정동코드"].astype(str).str[0:5] < "11140"]

    code_seo["ji_code"] = code_seo["법정동코드"].astype(str).str[0:5]



    


ym=list(["201804","201805","201806","201807","201808","201809","201810","201811","201812","201901"])
# ym=list(["201812"])

for m in ym:
  
  for co in code_seo["ji_code"]:
    
    temp=collect_land(m,co)
    
    sil_trade=pd.DataFrame()    
    sil_trade=pd.concat([sil_trade,temp])
    sqlite_append(sil_trade)
    
    print(co+", "+str(len(temp))+" is compleded")
    
  print("*"+str(m)+" is completed")



```

    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11110&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11110, 43 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11140&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11140, 76 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11170&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11170, 111 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11200&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11200, 100 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11215&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11215, 73 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11230&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11230, 147 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11260&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11260, 185 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11290&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11290, 343 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11305&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11305, 120 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11320&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11320, 216 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11350&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11350, 405 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11380&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11380, 231 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11410&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11410, 197 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11440&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11440, 127 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11470&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11470, 227 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11500&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11500, 281 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11530&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11530, 319 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11545&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11545, 87 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11560&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11560, 233 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11590&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11590, 146 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11620&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11620, 179 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11650&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11650, 156 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11680&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11680, 131 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11710&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11710, 194 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11740&DEAL_YMD=201804&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11740, 157 is compleded
    *201804 is completed
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11110&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11110, 58 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11140&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11140, 90 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11170&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11170, 118 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11200&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11200, 107 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11215&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11215, 60 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11230&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11230, 152 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11260&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11260, 214 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11290&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11290, 310 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11305&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11305, 137 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11320&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11320, 221 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11350&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11350, 451 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11380&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11380, 249 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11410&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11410, 157 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11440&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11440, 148 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11470&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11470, 262 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11500&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11500, 328 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11530&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11530, 311 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11545&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11545, 94 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11560&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11560, 206 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11590&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11590, 172 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11620&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11620, 206 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11650&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11650, 163 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11680&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11680, 136 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11710&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11710, 160 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11740&DEAL_YMD=201805&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11740, 187 is compleded
    *201805 is completed
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11110&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11110, 55 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11140&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11140, 83 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11170&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11170, 105 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11200&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11200, 106 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11215&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11215, 62 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11230&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11230, 172 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11260&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11260, 216 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11290&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11290, 363 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11305&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11305, 154 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11320&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11320, 260 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11350&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11350, 504 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11380&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11380, 302 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11410&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11410, 193 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11440&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11440, 155 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11470&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11470, 303 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11500&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11500, 300 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11530&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11530, 373 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11545&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11545, 103 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11560&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11560, 207 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11590&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11590, 199 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11620&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11620, 325 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11650&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11650, 163 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11680&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11680, 142 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11710&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11710, 183 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11740&DEAL_YMD=201806&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11740, 207 is compleded
    *201806 is completed
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11110&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11110, 65 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11140&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11140, 87 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11170&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11170, 185 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11200&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11200, 179 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11215&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11215, 110 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11230&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11230, 224 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11260&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11260, 177 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11290&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11290, 357 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11305&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11305, 202 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11320&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11320, 328 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11350&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11350, 671 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11380&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11380, 320 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11410&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11410, 203 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11440&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11440, 335 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11470&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11470, 420 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11500&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11500, 481 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11530&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11530, 432 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11545&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11545, 138 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11560&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11560, 325 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11590&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11590, 271 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11620&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11620, 250 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11650&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11650, 286 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11680&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11680, 300 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11710&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11710, 403 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11740&DEAL_YMD=201807&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11740, 278 is compleded
    *201807 is completed
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11110&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11110, 112 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11140&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11140, 136 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11170&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11170, 222 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11200&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11200, 606 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11215&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11215, 267 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11230&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11230, 531 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11260&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11260, 459 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11290&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11290, 745 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11305&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11305, 321 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11320&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11320, 852 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11350&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11350, 1884 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11380&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11380, 514 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11410&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11410, 428 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11440&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11440, 509 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11470&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11470, 821 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11500&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11500, 1070 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11530&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11530, 796 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11545&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11545, 320 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11560&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11560, 493 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11590&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11590, 462 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11620&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11620, 357 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11650&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11650, 598 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11680&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11680, 733 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11710&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11710, 948 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11740&DEAL_YMD=201808&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11740, 779 is compleded
    *201808 is completed
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11110&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11110, 68 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11140&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11140, 92 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11170&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11170, 106 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11200&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11200, 286 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11215&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11215, 176 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11230&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11230, 238 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11260&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11260, 282 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11290&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11290, 403 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11305&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11305, 207 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11320&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11320, 409 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11350&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11350, 1029 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11380&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11380, 282 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11410&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11410, 195 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11440&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11440, 219 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11470&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11470, 325 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11500&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11500, 317 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11530&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11530, 400 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11545&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11545, 194 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11560&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11560, 168 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11590&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11590, 149 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11620&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11620, 181 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11650&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11650, 268 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11680&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11680, 307 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11710&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11710, 540 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11740&DEAL_YMD=201809&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11740, 359 is compleded
    *201809 is completed
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11110&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11110, 29 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11140&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11140, 49 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11170&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11170, 52 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11200&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11200, 106 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11215&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11215, 54 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11230&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11230, 146 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11260&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11260, 163 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11290&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11290, 164 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11305&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11305, 88 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11320&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11320, 185 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11350&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11350, 367 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11380&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11380, 135 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11410&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11410, 110 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11440&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11440, 106 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11470&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11470, 129 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11500&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11500, 166 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11530&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11530, 213 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11545&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11545, 109 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11560&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11560, 123 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11590&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11590, 74 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11620&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11620, 135 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11650&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11650, 115 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11680&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11680, 136 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11710&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11710, 172 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11740&DEAL_YMD=201810&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11740, 138 is compleded
    *201810 is completed
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11110&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11110, 29 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11140&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11140, 24 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11170&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11170, 42 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11200&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11200, 47 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11215&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11215, 35 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11230&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11230, 123 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11260&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11260, 70 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11290&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11290, 96 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11305&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11305, 46 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11320&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11320, 108 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11350&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11350, 174 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11380&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11380, 65 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11410&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11410, 50 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11440&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11440, 46 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11470&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11470, 82 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11500&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11500, 77 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11530&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11530, 114 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11545&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11545, 63 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11560&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11560, 69 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11590&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11590, 43 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11620&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11620, 59 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11650&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11650, 64 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11680&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11680, 70 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11710&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11710, 108 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11740&DEAL_YMD=201811&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11740, 76 is compleded
    *201811 is completed
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11110&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11110, 13 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11140&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11140, 16 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11170&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11170, 23 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11200&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11200, 38 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11215&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11215, 30 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11230&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11230, 82 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11260&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11260, 64 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11290&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11290, 78 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11305&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11305, 40 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11320&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11320, 100 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11350&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11350, 185 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11380&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11380, 81 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11410&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11410, 45 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11440&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11440, 40 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11470&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11470, 58 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11500&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11500, 83 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11530&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11530, 87 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11545&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11545, 43 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11560&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11560, 48 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11590&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11590, 33 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11620&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11620, 53 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11650&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11650, 56 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11680&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11680, 79 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11710&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11710, 94 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11740&DEAL_YMD=201812&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11740, 64 is compleded
    *201812 is completed
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11110&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11110, 18 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11140&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11140, 17 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11170&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11170, 16 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11200&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11200, 32 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11215&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11215, 19 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11230&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11230, 84 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11260&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11260, 44 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11290&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11290, 69 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11305&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11305, 18 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11320&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11320, 80 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11350&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11350, 116 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11380&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11380, 57 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11410&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11410, 48 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11440&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11440, 34 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11470&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11470, 66 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11500&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11500, 57 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11530&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11530, 78 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11545&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11545, 40 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11560&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11560, 43 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11590&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11590, 33 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11620&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11620, 68 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11650&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11650, 30 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11680&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11680, 43 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11710&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11710, 55 is compleded
    http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev?LAWD_CD=11740&DEAL_YMD=201901&numOfRows=1000000&serviceKey=UmeOG8fXLWmdW2DwQMpFHgFwx%2BkwkYDtw16TtQeaP4xzbqtKCv08tZUAY8UcykdaFQ6Q92pbC4jJruoenAfXjA%3D%3D
    11740, 67 is compleded
    *201901 is completed



```python
import sqlite3

# SQLite DB 연결
conn = sqlite3.connect("/content/gdrive/My Drive/Colab Notebooks/real_trade.db")
 
# Connection 으로부터 Cursor 생성
cur = conn.cursor()
 
# SQL 쿼리 실행
#cur.execute("select * from lent")
data = cur.execute("select 년, 월, count(*) from lent group by 년, 월")
#data = cur.execute("select * from lent where 지역코드 = '11110' and 월 = '12' order by 아파트")
#cur.execute("drop table lent")
#cur.execute("delete from lent where 월 = '4'")
#conn.commit()
     

names = list(map(lambda x: x[0], cur.description))
print(names)

# 데이타 Fetch
rows = cur.fetchall()
for row in rows:
    print(row)


# Connection 닫기
conn.close()
```
