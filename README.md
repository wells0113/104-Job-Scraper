# 104-Job-Scraper
Python自動化爬蟲工具，用於搜尋指定關鍵字與地區的職缺，並輸出成 CSV 檔案與社覺化呈現職缺共通屬性。支援互動式地區選擇、錯誤處理與職缺詳細解析。

## 使用技術
- Python 3
- requests
- BeautifulSoup4
- pandas
- re / os / time 模組

## 功能特色
- 自動擷取 104 網站職缺資料 （可指定關鍵字與地區）
![image](https://github.com/user-attachments/assets/25890075-0618-4ba4-8f66-a4e386f24204)

- 解析詳細職缺條件（學歷、薪資、技能、語言等）
- 輸出為可分析的 CSV 表格
![image](https://github.com/user-attachments/assets/d9d9a0b8-185b-43c0-bc12-e0710969255f)

- 自動開啟結果，友善使用者互動介面
- 視覺化呈現爬取職缺的共通特性 （擅長工具、工作技能等）

## 執行方式

```bash
python 104_job_scraper.py

import requests
import pandas as pd
import time
import re
import os
import webbrowser
import platform
from bs4 import BeautifulSoup

headers = {
    "User-Agent": "Mozilla/5.0",
    "Referer": "https://www.104.com.tw/jobs/search/"
}

area_map = {
    "台北市": "6001001000",
    "新北市": "6001002000",
    "桃園市": "6001005000",
    "台中市": "6001008000",
    "台南市": "6001010000",
    "高雄市": "6001012000",
    "不限地區": "6001000000"
}

def fetch_jobs(keyword, pages=4, area="6001008000"):
    all_jobs = []
    for page in range(pages):
        print(f"🔍 擷取第 {page+1} 頁...")
        url = "https://www.104.com.tw/jobs/search/list"
        params = {
            "ro": "0",
            "kwop": "7",
            "keyword": keyword,
            "order": "11",
            "asc": "0",
            "page": page + 1,
            "mode": "s",
            "jobsource": "2018indexpoc",
            "area": area
        }
        res = requests.get(url, headers=headers, params=params)
        if res.status_code == 200:
            data = res.json()
            jobs = data.get("data", {}).get("list", [])
            all_jobs.extend(jobs)
        else:
            print(f"❌ 第 {page+1} 頁擷取失敗")
        time.sleep(1)
    return all_jobs

def extract_job_id(url):
    match = re.search(r'/job/(\w+)', url)
    return match.group(1) if match else None

def fetch_job_detail(job_id):
    url = f"https://www.104.com.tw/job/ajax/content/{job_id}"
    res = requests.get(url, headers=headers)
    if res.status_code == 200:
        data = res.json()
        return data.get("data", {})
    else:
        return None

def parse_job(detail):
    specialties = detail.get("condition", {}).get("specialty", [])
    specialty_str = ', '.join(s.get("description", "") for s in specialties) if specialties else "無"

    skills = detail.get("condition", {}).get("skill", [])
    skill_str = ', '.join(s.get("description", "") for s in skills) if skills else "無"

    # 避免 BeautifulSoup 空值錯誤
    job_description_html = detail.get("jobDetail", {}).get("jobDescription", "")
    job_description = BeautifulSoup(job_description_html, "html.parser").get_text(strip=True) if job_description_html else "無"

    return {
        "職缺名稱": detail.get("header", {}).get("jobName", "無"),
        "公司名稱": detail.get("header", {}).get("custName", "無"),
        "工作待遇": detail.get("jobDetail", {}).get("salary", "無"),
        "學歷要求": detail.get("condition", {}).get("edu", "無"),
        "工作經歷": detail.get("condition", {}).get("workExp", "無"),
        "科系要求": ', '.join(detail.get("condition", {}).get("major", [])) or "無",
        "語文條件": ', '.join(
            f"{lang['language']}({lang['ability']})"
            for lang in detail.get("condition", {}).get("language", [])
        ) if detail.get("condition", {}).get("language") else "無",
        "擅長工具": specialty_str,
        "工作技能": skill_str,
        "其他條件": detail.get("condition", {}).get("other", "無"),
        "工作內容": job_description
    }

def open_file(filepath):
    try:
        if platform.system() == "Darwin":  # macOS
            os.system(f"open \"{filepath}\"")
        elif platform.system() == "Windows":
            os.startfile(filepath)
        elif platform.system() == "Linux":
            os.system(f"xdg-open \"{filepath}\"")
    except Exception as e:
        print(f"❌ 無法自動開啟檔案：{e}")

def main():
    start_time = time.time()

    # 使用者輸入
    keyword = input("請輸入職缺關鍵字（例如：資料分析、Python）：").strip()
    print("\n📍 請選擇地區：")
    for idx, city in enumerate(area_map.keys(), 1):
        print(f"{idx}. {city}")
    while True:
        try:
            area_idx = int(input("➡️ 輸入地區編號："))
            area = list(area_map.values())[area_idx - 1]
            break
        except (ValueError, IndexError):
            print("⚠️ 請輸入有效的地區編號！")

    # 擷取職缺列表
    jobs = fetch_jobs(keyword, pages=4, area=area)
    print(f"\n📦 共擷取到 {len(jobs)} 筆職缺")

    result = []
    for job in jobs:
        job_url = job.get("link", {}).get("job", "")
        job_id = extract_job_id(job_url)
        if not job_id:
            print(f"⚠️ 跳過：無法解析 ID → {job}")
            continue
        print(f"→ 處理職缺：{job.get('jobName')}（ID: {job_id}）")
        detail = fetch_job_detail(job_id)
        if detail:
            job_info = parse_job(detail)
            result.append(job_info)
        else:
            print(f"❌ 抓取職缺 {job_id} 詳細內容失敗")
        time.sleep(0.5)

    # 儲存結果
    df = pd.DataFrame(result)
    filename = f"104_job_list_{keyword}.csv"
    df.to_csv(filename, index=False, encoding="utf-8-sig")
    print(f"\n✅ 儲存成功：{filename}")

    # 自動開啟
    full_path = os.path.abspath(filename)
    open_file(full_path)

    # 統計時間
    duration = round(time.time() - start_time, 2)
    print(f"\n🎉 完成！共成功擷取 {len(result)} 筆職缺，耗時 {duration} 秒。")

if __name__ == "__main__":
    main()

```
