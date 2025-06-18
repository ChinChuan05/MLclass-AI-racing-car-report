# 第九組 AI賽車訓練報告統整

* 成員：張閔翔、謝進權、謝坤霖、黃正賢

## 簡介

本專案為 AWS DeepRacer 訓練車輛模型所設計的獎勵函數 "reward function"，目的是透過強化學習引導車輛沿著最佳路徑（racing line）以穩定且快速的方式完成賽道。程式中先定義多個輔助函數以計算車輛與賽道之間的幾何關係，例如與最佳路線的距離、方向偏差與最近的賽道點等，其中先後經歷五次重大調整。

主體的獎勵邏輯則根據車輛是否偏離賽道、是否接近最佳路徑、方向是否正確、速度是否與該段賽道匹配，以及轉彎時的穩定性進行加分或懲罰。針對不同的賽道區段，程式使用手動標記的 index 來細分速度與操控策略。當車輛完成整圈賽道時，根據步數給予額外獎勵，以鼓勵更高效率的行駛。整體設計目標是平衡穩定性與速度，促使模型學習到更優的駕駛策略。

---

## 訓練目標

* **穩定沿最佳路徑行駛**
  車輛應盡可能貼近手動標記的 optimal racing line，以減少偏移，提高效率。
* **維持適當速度行駛各區段**
  根據不同賽道區段的難易程度與彎道曲率，車輛應調整速度，達到速度-操控的最佳平衡。
* **減少轉向角與方向偏差**
  車輛應避免劇烈偏轉與方向錯誤，透過平穩轉向通過彎道，避免失控或偏離賽道。
* **在最少步數下完成賽道**
  模型目標是在固定時間內以最少的 decision steps 完成整圈，代表更有效率的行駛。
*

## 評估指標

* **Reward 值總和（Total Reward）**
  每次訓練或模擬跑完賽道後會累積 reward 值，越高代表模型行為越符合訓練目標。
* **完成進度（Progress %）**
  車輛在一趟模擬中完成的百分比，若頻繁無法跑完全圈，模型即可能尚未收斂或策略不佳。
* **步數（Steps）**
  完成賽道所花的 decision steps；理想上愈少步數愈好，代表學習出更有效率的路線與速度控制。
* **Off-track 次數或機率**
  車輛偏離賽道的頻率，用來評估模型穩定性與學習程度。
* **方向誤差（Direction difference）**
  車輛實際朝向與最佳路線方向的偏差大小，偏差大會導致懲罰並降低 reward。

---

## 第一次調整（初階）

* **目標：能夠正常行駛**

![image.png](https://raw.githubusercontent.com/ChinChuan05/MLclass-AI-racing-car-report/refs/heads/master/1.1.PNG)
**總結：**（１）避免完全為 0 的獎勵，讓強化學習模型仍能獲得微弱的回饋

　　　（２）車輛與中心線距離至少還有 5 公分的緩衝區，避免貼邊行駛

---

## 第二次調整（漸進式）

* **目標：漸進式獎勵，根據距離中心的遠近給不同獎勵**

![image.png](https://github.com/ChinChuan05/MLclass-AI-racing-car-report/blob/master/2.1.PNG?raw=true)

**總結：**（１）沒有硬性要求 "all\_wheels\_on\_track"，容錯率稍高。

　　　（２）根據車輛距離中心的程度逐級給分，有助於模型學習更穩定地靠近中心。

---

## 第三次調整（進階訓練）

**目標：提升整體速度與穩定性**

![image.png](https://github.com/ChinChuan05/MLclass-AI-racing-car-report/blob/master/3.1.PNG?raw=true)
**總結：**（１）可以讓車輛學習貼近最佳走線

　　　（２）是更貼近專業賽車手概念的 reward 設計邏輯。

---

## 第四次調整（手動優化）

* **目標：嘗試以手動微調獲得更優良模型**

  ![image.png](https://github.com/ChinChuan05/MLclass-AI-racing-car-report/blob/master/4.1.PNG?raw=true)

![image.png](images/https://github.com/ChinChuan05/MLclass-AI-racing-car-report/blob/master/4.2.PNG?raw=true)
**總結：**（１）具專注理想走線的精度學習
　　　（２）整合路線偏差、速度、出界檢查與路段特化，設計完整

---

## 第五次調整（最終版）

* **目標：結合先前四個模型優點進行優化**

  *此段為手動調整*

  ![image.png](https://github.com/ChinChuan05/MLclass-AI-racing-car-report/blob/master/5.1.PNG?raw=true)
  *此段為速度獎勵*![image.png](https://github.com/ChinChuan05/MLclass-AI-racing-car-report/blob/master/5.2.PNG?raw=true)
*此段為方向懲罰*

![image.png](https://github.com/ChinChuan05/MLclass-AI-racing-car-report/blob/master/5.3.PNG?raw=true)
**總結：**（１）透過多層次判斷邏輯來細緻獎勵車輛行為，是一個典型的手工微調式 DeepRacer 獎勵函數，適用　　　　　　　　

　　　　　　於進階車手針對特定賽道進行高效訓練與優化。

　　　（２）程式預先定義了一條「最佳路徑」（racing line），包含每個路點的座標、理想速度與行進時間，　　　　　　　　　　　　　　

　　　　　　作為訓練目標參考。

　　　（３）程式透過手動分類（如 `above_three`, `strong_left` 等），區分不同區段的特性（直線、急彎、左

　　　　　　右側等），並根據區段調整速度與轉向的獎勵/懲罰。


**總結：**（１）透過多層次判斷邏輯來細緻獎勵車輛行為，是一個典型的手工微調式 DeepRacer 獎勵函數，適用　　　　　　　　

　　　　　　於進階車手針對特定賽道進行高效訓練與優化。

　　　（２）程式預先定義了一條「最佳路徑」（racing line），包含每個路點的座標、理想速度與行進時間，　　　　　　　　　　　　　　

　　　　　　作為訓練目標參考。

　　　（３）程式透過手動分類（如 `above_three`, `strong_left` 等），區分不同區段的特性（直線、急彎、左

　　　　　　右側等），並根據區段調整速度與轉向的獎勵/懲罰。

![image.png](https://github.com/ChinChuan05/MLclass-AI-racing-car-report/blob/master/pic.PNG?raw=true)
*平均完成時間約為9秒*

---

## **統整**

設計獎勵函數過程中，我深刻體會到強化學習並非僅靠理論即可奏效，實作與不斷調整才是關鍵。每一段 reward function 的修改，都是在教導車輛「什麼是好行為」，背後反映出策略設計的邏輯思維與工程取捨。此外，觀察模型行為與評估獎勵函數之間的關聯，也提升對 AI 訓練迴圈的理解。透過實驗不同的策略組合，學會如何將數學邏輯轉化為具體行為控制，這不僅讓我對強化學習有更深層的掌握，也強化了我在 AI 領域中的應用能力與創造力。

此外我深有體會強化學習中「設計目標」的重要性。這段程式透過明確定義最佳行車路徑、不同區段的速度限制與車輛位置，讓模型能在訓練過程中逐步優化行駛策略，進而提升整體表現。特別是手動標記不同賽道特性的方法，讓我理解到即使是自動化訓練，也需要人為的經驗判斷與策略設計。整體而言，這段程式不僅展示了強化學習在實務應用上的靈活性，也讓我意識到 reward function 的細節往往是影響模型成效的關鍵。

**完成時間進步：**35->33->23->18->9(S)
