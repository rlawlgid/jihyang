# jihyang
# ai_cptocurrency_project
한양대학교 AI와암호화폐이야기 프로젝트
한양대학교 <AI와암호화폐이야기>

## Project 1

import time
import requests
import pandas as pd
import csv

import os
from datetime import datetime
from pytz import timezone

"""
  데이터를 저장하기 위한 경로 생성
"""
# working directory
current_path = os.getcwd()
file_path = f"{current_path}/jh_assignment/data"

# Make a directory
if (not os.path.isdir(f"{current_path}/jh_assignment")):
    os.mkdir(f"{current_path}/jh_assignment")
if (not os.path.isdir(file_path)):
    os.mkdir(file_path)

"""
  orderbook 데이터를 파일에 저장하는 함수
"""
# csv 파일 컬럼명
columns = ["price", "quantity", "type", "timestamp"]
# Orberbook Collection method
def orderbook_collection():
    book = {}
    response = requests.get('https://api.bithumb.com/public/orderbook/BTC_KRW/?count=5')
    book = response.json()
    data = book['data']

    # orderbook response 에 있는 bids, asks 데이터 추출 및 정렬
    bids = (pd.DataFrame(data['bids'])).apply(pd.to_numeric,errors='ignore')
    asks = (pd.DataFrame(data['asks'])).apply(pd.to_numeric,errors='ignore')
    bids.sort_values('price', ascending=True, inplace=True)
    asks.sort_values('price', ascending=True, inplace=True)
    bids = bids.reset_index()
    del bids['index']
    # type 값 할당 (bids=0, asks=1)
    bids['type'], asks['type'] = 0, 1

    # bids, asks 데이터 합치기
    df = pd.concat([bids, asks])

    # 현재 시간 구하기
    timestamp_now = datetime.now()
    df['timestamp'] = timestamp_now

    # 파일 이름 생성
    file_name = f"{timestamp_now.strftime('%Y-%m-%d')}-exchange-market-orderbook.csv"

    # orderbook 데이터 csv에 파일에 저장
    # if -> 파일이 없는 경우, 새로운 파일에 저장
    if (not os.path.exists(f"{file_path}/{file_name}")):
      df.to_csv(f"{file_path}/{file_name}", columns=columns, sep=',', index=False)
    # else -> 이미 파일이 있는 경우, 데이터 추가
    else:
      df.to_csv(f"{file_path}/{file_name}", header=False ,sep=',', index=False, mode='a')

    # Sleep 1 second
    time.sleep(1)


# 계속 orderbook_collection 함수 호출 (조건이 True이기 때문에)
while(True):
    orderbook_collection()
