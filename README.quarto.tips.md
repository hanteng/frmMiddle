
## 圖片檔案儲存慣例

專案根目錄下專用目錄（例如 `images/` 或 `img/` 或 `assets/`）的慣例，那麼在 Quarto 的 Markdown 語法中引用這些圖片的方法如下：

關鍵在於使用**相對路徑**，從您正在編輯的 `.qmd` 檔案指向圖片檔案。

我們假設您的專案結構如下：

```
my-quarto-project/
├── _quarto.yml             # 主要設定檔
├── index.qmd               # 您的主頁面
├── about.qmd               # 另一個頁面
├── posts/
│   └── first-post/
│       └── index.qmd       # 一個位於自己目錄中的部落格文章
├── images/                 # <--- 建議的圖片資源目錄
│   ├── shared-logo.png     # 用於多個頁面
│   ├── about-illustration.svg
│   └── posts/              # 專為文章準備的圖片子目錄
│       └── first-post-diagram.png
│       └── another-post-image.jpg
└── styles.css
```

### 1. 基本 Markdown 圖片語法

標準的 Markdown 圖片語法可以完美運作，您只需要提供正確的相對路徑。

Markdown

```
![圖片的替代文字](圖片路徑/圖片名稱.副檔名 "可選的滑鼠懸停標題")
```

#### 範例 1: 從 `index.qmd` 或 `about.qmd`（都在根目錄）引用圖片

如果您正在編輯 `index.qmd` 或 `about.qmd`：

Markdown

```
# 我的首頁

歡迎來到我的網站！這是一個漂亮的插圖：

![一個可愛的共用插圖](images/shared-logo.png "共用 Logo")

這是一個關於頁面專用的插圖：
![關於頁面的插圖](images/about-illustration.svg)
```

**說明：** 從 `index.qmd`（或 `about.qmd`），`images/` 資料夾可以直接存取。

#### 範例 2: 從 `posts/first-post/index.qmd` 引用圖片

如果您正在編輯 `posts/first-post/index.qmd`：

Markdown

```
# 我的第一篇部落格文章

這篇文章有一個圖表：

![第一篇文章的圖表](images/posts/first-post-diagram.png)

這是再次顯示的共用 logo：
![網站共用 Logo](images/shared-logo.png)
```

**說明：**

- 要找到 `first-post-diagram.png`：路徑 `images/posts/first-post-diagram.png` 是**相對於專案根目錄**的，而不是相對於 `first-post` 目錄內的 `index.qmd`。這是 Quarto 中一個重要的區別。
    
- 要找到 `shared-logo.png`：類似地，`images/shared-logo.png` 也指相對於專案根目錄的圖片。
    

Quarto 關於路徑的重要行為：

Quarto 在處理路徑方面通常很聰明：

- **不以 `/` 開頭且非相對（例如 `../`）的路徑：** 這些路徑通常被視為相對於目前 `.qmd` 檔案的 _輸出目錄_，或者如果檔案解析器能夠判斷，則相對於專案根目錄處理。
    
- **以 `/` 開頭的路徑（從根目錄絕對路徑）：** 這些路徑指的是 _網站輸出_ 的根目錄，而不是您專案的檔案系統根目錄。因此，`/images/logo.png` 將正確指向 `您的網站.com/images/logo.png`。這通常是引用共用資產最可靠的方式。
    

**為了清晰和穩健性，我們重新建議的路徑設定：**

對於共用資產，使用以 `/` 開頭的路徑（表示從**網站根目錄**開始）通常是最萬無一失的，因為它能確保無論您的 `.qmd` 檔案巢狀多深，路徑都是正確的。

### 2. 建議的穩健路徑設定（從網站根目錄的絕對路徑）

最穩健的共用資產引用方式是使用以 `/` 開頭的路徑，這表示您**已發佈網站的根目錄**。

```
my-quarto-project/
├── _quarto.yml
├── index.qmd
├── about.qmd
├── posts/
│   └── first-post/
│       └── index.qmd
├── images/               # 這裡的資產將被複製到輸出的 `/images`
│   ├── shared-logo.png
│   ├── about-illustration.svg
│   └── posts/
│       └── first-post-diagram.png
└── styles.css
```

#### 範例 1（修訂版）：從 `index.qmd` 或 `about.qmd`

Markdown

```
# 我的首頁

歡迎來到我的網站！這是一個漂亮的插圖：

![一個可愛的共用插圖](/images/shared-logo.png "共用 Logo")

這是一個關於頁面專用的插圖：
![關於頁面的插圖](/images/about-illustration.svg)
```

#### 範例 2（修訂版）：從 `posts/first-post/index.qmd`

Markdown

```
# 我的第一篇部落格文章

這篇文章有一個圖表：

![第一篇文章的圖表](/images/posts/first-post-diagram.png)

這是再次顯示的共用 logo：
![網站共用 Logo](/images/shared-logo.png)
```

**為什麼開頭的 `/` 很好用：**

- 當 Quarto 渲染您的網站時，它會將您的 `images/` 目錄複製到 `_site/images/`（或 `docs/images/` 等，取決於您的 `output-dir` 設定）。
    
- 像 `/images/shared-logo.png` 這樣以 `/` 開頭的路徑，將會從您網站上的**任何頁面**正確解析為 `您的網站.com/images/shared-logo.png`，無論該頁面巢狀有多深。
    

### 3. 特定於文章/文件的圖片（與 `.qmd` 檔案同層）

如果您有只被單一 `.qmd` 檔案使用的圖片，並且該 `.qmd` 檔案位於自己的目錄中（例如部落格文章），那麼將這些圖片直接放在該目錄中通常會很方便。

```
posts/
└── my-cool-post/
    ├── index.qmd
    └── post-diagram.png  # 該文章專用的圖片
```

在 `posts/my-cool-post/index.qmd` 中：

Markdown

```
# 我的酷文章

這是這篇文章獨有的圖表：

![酷文章圖表](post-diagram.png)
```

**說明：** 由於 `post-diagram.png` 與 `index.qmd` 在**同一個目錄**中，您可以直接使用它的檔名，或者 `./post-diagram.png`。

### 總結：

- **對於共用資產（logo、通用插圖等）：** 將它們放在專案根目錄下一個專用資料夾（例如 `images/`）。使用從網站根目錄開始的路徑引用它們，例如 `![替代文字](/images/我的圖片.png)`。這是最穩健的方法。
    
- **對於特定於巢狀文件的資產（例如部落格文章）：** 如果該文件位於自己的資料夾中，您可以將圖片直接放在該資料夾中，並僅使用檔名引用它們，例如 `![替代文字](圖表.png)`。
    

這種靈活的方法允許您同時實現高度組織化的共用資產和便利的每個文件專屬資產管理。