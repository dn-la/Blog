# [A2HS](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Add_to_home_screen)

- 使用WS的 `beforeinstallprompt` 來達成
- 可以依照使用者選擇flow來詢問 新增/不新增shortcut ?
    - 問題: `beforeinstallprompt` 會自然發生
    - 解法: 可以延遲它, 等到user確定才tigger [(ref1](https://github.com/mdn/pwa-examples/blob/master/a2hs/index.js#L36)[, ref2)](https://web.dev/customize-install/#in-app-flow)
    ```jsx=
    window.addEventListener('beforeinstallprompt', (e) => {
      // Prevent Chrome 67 and earlier from automatically showing the prompt
      e.preventDefault();
      // Stash the event so it can be triggered later.
      deferredPrompt = e;
      
    buttonInstall.addEventListener('click', (e) => {
    
      // Show the install prompt
      deferredPrompt.prompt();
      // Wait for the user to respond to the prompt
      deferredPrompt.userChoice.then((choiceResult) => {
        if (choiceResult.outcome === 'accepted') {
          console.log('User accepted the install prompt');
        } else {
          console.log('User dismissed the install prompt');
        }
      })
    });
    }
    ```
- Designer需要配合提供的圖片尺寸、色碼?什麼可以改？什麼不行？
  - Manifest 控制 檔名: `somefilename.webmanifest`
  - [Manifest fields ref](https://pwa-workshop.js.org/1-manifest/#manifest-fields)
  ```json=
  {
    "background_color": "purple", // 預載畫面
    "theme_color": "green", // status bar 底色
    "description": "Shows random fox pictures. Hey, at least it isn't cats.", 
    "display": "fullscreen", //詳見下方
    "icons": [
      //svg 檔案經過實測發現mobile無用
      {
        "src": "icon/fox-icon.png",
        "sizes": "192x192",
        "type": "image/png"
      },
      {
        "src": "icon/fox-icon.png",
        "sizes": "512x512",
        "type": "image/png"
      }
    ],
    "name": "Awesome fox pictures",
    "short_name": "Foxes",
    "start_url": "/pwa-examples/a2hs/index.html"
  }
  ```
  - different type of `display`
  
    ![display](https://i.imgur.com/PgoPw2m.jpg)
  - different type of `background_color`
  
    ![background_color](https://i.imgur.com/IaLu0rW.jpg)

  - different type of `background_color`
  
    ![theme_color](https://i.imgur.com/jeJjdmY.png)

- iOS/safari上是否支援？有什麼限制?
  - 基本上不支援PWA [ref](https://medium.com/@firt/progressive-web-apps-on-ios-are-here-d00430dee3a7)
    - The display: fullscreen and display: minimal-ui won’t work on iOS; fullscreen will trigger standalone, and minimal-ui will be just a shortcut to Safari. You can get something similar to fullscreen (the status bar will exist but over your app) using the cover-fit viewport extension or a deprecated meta tag though.standalone 則不一致
    - The theme-color to style the status bar won’t work; you can use the deprecated meta tag for black or white status bars, or you can use a CSS/HTML trick to emulate a theme-color.
    - Also, iOS is not taking the icons from the Web App Manifest, but from the apple-touch-icon link. If you don’t provide the link tag, a screenshot will be used for the icon
    - No manifest events will be fired, so if you are tracking installation through these channels, it won’t work on iOS (but you can check navigator.standalone instead).
    - There are bugs when your app runs in standalone mode.
  - User 只能手動加捷徑

- 一個網站新增多個捷徑？
  - ios基本上有獨立網址 就可以
  - 可以將`start_url`留空, 這樣`start_url`會依照使用者開啟的頁面添加 [ref](https://stackoverflow.com/questions/46203661/possible-to-have-multiple-manifest-json-for-pwa)
  - 利用`scope`達成多個manifest出現多個PWA(1個SW)  [ref](https://stackoverflow.com/questions/51280821/multiple-pwas-in-the-same-domain)

- 怎麼追蹤是從桌面捷徑打開的？
  - aOS`start_url`設置 utm code
  - iOS 可以利用[`window.navigator.standalone`](https://www.itread01.com/content/1496915766.html)來送