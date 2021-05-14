# service worker大概怎麼運作

[good ref](https://www.slideshare.net/grigs/designing-progressive-web-apps)


### What is service worker?
:::info
A service worker is a script that your browser runs **in the background**, separate from a web page, opening the door to features that don't need a web page or user interaction. Today, they already include features like **push notifications** and **background sync**. In the future, service workers might support other things like **periodic sync** or **geofencing**.
:::

> [reference link] https://developers.google.com/web/fundamentals/primers/service-workers


#### 小知識們
- 在service worker之前有另一款"AppChache",但是存在許多問題,而 service worker 因應設計成避開這些問題的樣子
- Service worker 是 [JS worker](#什麼是-JSweb-Worker), 故無法直接存取 DOM 
    > Service Worker 應是基於 JS worker 上而開發的
- Service worker 是 network 在client端的 proxy , 讓你可以控制你的 page 發出的 request
- Service worker 在不用時會終止, 下次使用時會重來 restart, 所以無法透過global state 去保存狀態, 如果需要保留及重用跨 cycle 的資料, 可以透過 [IndexedDB API](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API) 存取
- Service worker 大量使用到 Promises
- geoFencing 不是geolocation 而是地理圍欄, 能做到的事情跟beacon很類似
    
:::info
Instead, a service worker can communicate with the pages it controls by responding to messages sent via the [postMessage](https://html.spec.whatwg.org/multipage/workers.html#dom-worker-postmessage) interface, and those pages can manipulate the DOM if needed.
:::


### 什麼是 JS(web) Worker?
JS(web) Worker 可以理解成達成 Multiple Threads 的一個重要幕後功臣 , 可以利用 `postMessage` \ `onMessage` 跟 Main Thread 溝通, 將需要長時間運算且不含 Window 或 DOM Element 操作的程式碼放在 Web Workers，好處是不阻塞 Main Thread 而讓速度變快, performance 變好


### Indexed DB 是什麼?
> IndexedDB is a low-level API for client-side storage of significant amounts of structured data, including files/blobs.

> While Web Storage is useful for storing smaller amounts of data, it is less useful for storing larger amounts of structured data.

用來儲存資料到用戶端的瀏覽器。IndexedDB適合用來儲存大量非結構式的資料。

1. 非結構式的意思就是不限定格式，使用起來就像javascript中的object
2. Key-Value資料庫，IndexedDB存取資料的方式，是透過Key來讀取Value。
3. 非同步的方式存取
4. 一個App只有一個資料庫
5. 基於交易性質的資料庫模型，再IndexedDB上進行的操作都是以Transaction(交易)的方式執行，假如存取過程中，有一個動作失敗，則整筆交易會取消，不會只存入一半的資料，防止資料不完全。

### service worker Life Cycle

0. Register 
>Registering a service worker will cause the browser to start the service worker install step in the background.
1. Installing
It will cache some static assets
>  If any of the files fail to download and cache, then the install step will fail and the service worker won't activate, it'll try again next time.

>  it means if it does install, you know you've got those static assets in the cache.
- `event.waitUntil( *do add caches* )` 避免fetch 事件跟cache同時進行
- `cache.addAll([...])` 多個不同Caches新增的語法

2. Activated
> 注意，當有多個 Service Worker 時，新安裝的 Service Worker 並不會馬上啟動，必須關閉舊有的所有 Service Worker 後，新的才能啟動。為了節省 Service Worker 的空間，在這裡也可刪除舊Cache的資料。

> this is a great opportunity for handling any management of old caches
- `cache.delete()`

3. Fetch
 - fetch
    1. 註冊Service Worker後，第一次執行install之後並不會觸發fetch，因此必須重新整理頁面。
    2. Service Worker就像一個用戶端的proxy，當現在請求都經過Service Worker, 能抓取的資源像image、css和js等，都是我們能抓取的資源,
    3. `event.respondWith` 可以將頁面發出的請求劫持(hijack)HTTP request，讓我們利用respondWith()處理後續回應, 便可以拿來應用在查詢是否有Cache可提供

![](https://i.imgur.com/1C0eHWt.png)

### 為什麼要存個東西到Cache？
- 網路不穩
- 沒有訊號(電梯、地下室等)
- 假Wi-fi 有時候可能只是短暫性的網路不穩，但是網站瀏覽到一半之後，如果網站直接斷掉，就會影響使用者的興致，但在不穩的情況下，我們能維持網站最基本UI的維運，當走到有網路的地方，再更新網站的資訊，讓網站一直維持在運作的情況。
