##### tags: `Squ1rrelCTF_2025`
# Squ1rrelCTF 2025 Web : go-getter

<style>
.red {
  color: red;
}
.blue {
  color: blue;
}
.purple {
  color: #7D3382;
}
.light_purple {
  color: #810cf5;
}
</style>

<span class=""></span>

## <span class="red">題目</span>

- ![image](https://hackmd.io/_uploads/B1rLkx-CJx.png)
- ![image](https://hackmd.io/_uploads/SJNSygbR1l.png)

## <span class="red">解題過程 (After Event)</span>

1. **進到靶機後可以發現有兩個選項可以選，那就直接選選看 ```...I want the flag``` 這個選項，結果會跳出 Access denied**

- ![image](https://hackmd.io/_uploads/SyA5xlW0Je.png)

&emsp;

2. **題目有給 source code ，看了一下發現這題的架構應該是：
```frontend(go) <-> middleware (go) <-> backend (python)```**

- **go 主要邏輯處理：**
- ![image](https://hackmd.io/_uploads/BkAHMxzkle.png)
- ![image](https://hackmd.io/_uploads/ByX0NxZ0Jg.png)
- ![image](https://hackmd.io/_uploads/HkpdmxZA1e.png)
- **python 主要邏輯處理：**
- ![image](https://hackmd.io/_uploads/Hk0s7lbCyg.png)

&emsp;

3. **這題的重點在於如何繞過 go 的 json req.body 檢查。
```json.Unmarshal()``` 在解析 struct 時，如果 JSON key 大小寫不同，也會嘗試匹配 struct 的欄位名，但是 ```json.loads()``` 會區分大小寫，所以可以寫出以下的 payload：**

- **```json.Unmarshal()``` 是 Golang 調用的， ```json.loads()``` 是 python 調用的**
**( python 的 ```get_json()``` 底層會調用 ```json.loads()```)**
- ![image](https://hackmd.io/_uploads/rk0qYgbR1g.png)
- flag🚩 : squ1rrel{p4rs3r?\_1_h4rd1y_kn0w_3r!}