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
