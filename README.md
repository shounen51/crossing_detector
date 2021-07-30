# ROI偵測 crossing_detector
> 可以對單個攝影機繪製多個任意多邊形的ROI，並且對每個ROI設定排程偵測是否有物件在ROI內，相關設定以及偵測方式可參考下方說明

需要套件:
* cv2
* numpy

### Public Functions
|return|function name|
| ------------- | ------------------------------ |
||crossing_detector(init)|
||add_area_list|
||add_area_dict|
||del_area|
|list|get_area_list|
|numpy.array|detector|
||save_area|
|numpy.array|draw_area|


init(self, cam_name:str="cam_name")
------
> 初始化

### 參數
* **cam_name** <font color="#0000ff">(str)</font>: 一支攝影機需要一個crossing_detector實例，所以當多個攝影機時就需要為每個實例設置攝影機名稱來區別。

### 回傳
None

add_area_list(self, areaName:str, Npoints:list, alertType:str, day:str, hour:str, sec:str)
------
> 為該攝影機新增一個區域，區域的相關設定個別傳入參數

### 參數
* **areaName** <font color="#0000ff">(str)</font>: 區域的名稱，單攝影機可以繪製多個區域，可以為每個區域命名做區分，請不要留空。
* **Npoints** <font color="#0000ff">(list)</font>: 區域的各頂點，list內的每個元素為(x,y):tuple，x和y的範圍是0~1。
* **alertType** <font color="#0000ff">(str)</font>: 預留擴充功能用，暫時無用處，建議填入1。
* **day** <font color="#0000ff">(str)</font>: 排程用，長度為7的01字串，詳細參考最下方區域設定文檔說明。
* **hour** <font color="#0000ff">(str)</font>: 排程用，以24小時制設定，中間以逗號隔開，詳細參考最下方區域設定文檔說明。
* **sec** <font color="#0000ff">(str)</font>: 排程用，設定該區域被入侵多久會觸發警報，詳細參考最下方區域設定文檔說明。

### 回傳
None

add_area_dict(self, _dict)
------
> 為該攝影機新增一個區域，區域的相關設定以json格式傳入，相關格式說明請參考最下方區域設定文檔說明

### 參數
* **_dict** <font color="#0000ff">(dict)</font>: 將透過crossing_detector.save_area()所匯出的.txt文檔讀取後直接匯入。
### 回傳
None

del_area(self, name)
------
> 刪除指定的ROI

### 參數
* **name** <font color="#0000ff">(str)</font>: 欲刪除的ROI名稱。
### 回傳
None

get_area_list(self)
------
> 獲得該攝影機已設置的區域

### 參數
None
### 回傳
* <font color="#0000ff">(list)</font>: 區域的list，元素為區域的名稱。

detector(self, ori_bbox_xyxy, size:tuple=(1,1))
------
> 偵測該攝影機的區域是否有被入侵，每個frame都需要呼叫，如果沒有物件需要判別時傳入空array。

### 參數
* **ori_bbox_xyxy** <font color="#0000ff">(numpy.array)</font>: 二維陣列。該區域的物件座標，第一個維度是物件，第二個維度是物件的xy座標。
* **size** <font color="#0000ff">(tuple)</font>: 該物件的xy座標系的size，若以正規化至0~1則不用傳入。
### 回傳
* <font color="#0000ff">(numpy.array)</font>: 二維陣列。檢視是否有區域被哪個物件踏入而觸發警報。第一個維度是物件，第二個維度是區域，當區域被觸發時會回傳Ture，否則False。
### 範例
```
[[False True True]
 [False False True]]
```
該攝影機有繪製兩個區域，當前frame有三個物件。
第一個攝影機未觸發警報。
第二個區域已觸發警報，並且第二個物件正在該區域中。
第三個區域已觸發警報，並且第二和第三個物件正在該區域中。

save_area(self, path, area_name)
------
> 將已繪製的區域儲存成.txt檔，相關格式說明請參考最下方區域設定文檔說明

### 參數
* **path** <font color="#0000ff">(str)</font>: 儲存路徑，需指名文檔名稱以及副檔名，例如 `./a.txt`。
* **area_name** <font color="#0000ff">(tuple)</font>: 欲儲存的區域名稱，一個文檔對應一個區域。
### 回傳
None

draw_area(self, img)
------
> 將該攝影機已繪製的區域畫在圖片上，區域若未啟動為灰色，啟動時會以入侵時間判斷由綠轉紅，觸發警報時為紅色

### 參數
* **img** <font color="#0000ff">(numpy.array)</font>: 攝影機畫面的圖片。
### 回傳
* **img** <font color="#0000ff">(numpy.array)</font>: 繪製區域後的圖片。

# 攝影機區域設定文檔說明
> 一個區域可被儲存為一個.txt文檔，內容為json格式。

### keys說明
* **cam**: 攝影機名稱。
* **areaName**: 區域名稱。
* **area**: 紀錄一個區域用的參數，使用者可無視。
* **alertType**: 為未來擴充功能而留的參數，暫無作用，建議設置1。
* **day**: 紀錄一個星期哪幾天需要啟動，第一個位置是星期一，假設工作日才需啟動則設置1111100。全天是1111111
* **hour**: 紀錄一天幾點到幾點需要啟動，24小時制，假設晚上10點到早上6點啟動則設置22,6。全天是0,24
* **sec**: 當此區域被入侵幾秒後會觸發警報。
### 範例
```json=
{
    "cam": "cam0",
    "areaName": "AAA",
    "area": {
        "abs": [
            [
                0.372,
                0.059
            ],
            [
                -8.087,
                7.469
            ],
            [
                0.457,
                0.349
            ],
            [
                -2.482,
                2.218
            ]
        ],
        "points": [
            [
                0.7564814814814815,
                0.34074074074074073
            ],
            [
                0.8759259259259259,
                0.3851851851851852
            ],
            [
                0.8333333333333334,
                0.7296296296296296
            ],
            [
                0.6361111111111111,
                0.6395061728395062
            ]
        ]
    },
    "alertType": "1",
    "day": "1111111",
    "hour": "0,24",
    "sec": "3"
}
```
