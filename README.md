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
current_path = os.getcwd()
file_path = f"{current_path}/jh_assignment/data"

if (not os.path.isdir(f"{current_path}/jh_assignment")):
    os.mkdir(f"{current_path}/jh_assignment")
if (not os.path.isdir(file_path)):
    os.mkdir(file_path)

"""
  orderbook 데이터를 파일에 저장하는 함수
"""

columns = ["price", "quantity", "type", "timestamp"]

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

    df = pd.concat([bids, asks])

    timestamp_now = datetime.now()
    df['timestamp'] = timestamp_now

    file_name = f"{timestamp_now.strftime('%Y-%m-%d')}-exchange-market-orderbook.csv"

    if (not os.path.exists(f"{file_path}/{file_name}")):
      df.to_csv(f"{file_path}/{file_name}", columns=columns, sep=',', index=False)
    else:
      df.to_csv(f"{file_path}/{file_name}", header=False ,sep=',', index=False, mode='a')

    time.sleep(1)

while(True):
    orderbook_collection()
