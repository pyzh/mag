## 將 Wikidown 推上 github 成為開源靜態網站

雖然 github 是開放原始碼的人用  git 版本管理系統發布專案的地方，但是卻也提供了 Github-Pages (gh-pages) 這一個版本可以讓人架設網站，在本文中，我們透過 wikidown 示範如何將專案發布到 github 上，並且架設成靜態網站的方法。

雖然 wikidown 已經存在於 <https://github.com/ccckmit/wikidown.js> 這個網址上了，但是為了要重新示範一次，我們將創建一個新的專案，稱為 wikidown2 ，其網址將會位於 <https://github.com/ccckmit/wikidown2> 當中。

首先讓我們檢視一下專案的內容，以下是我們用檔案管理員檢視尚未進行 git 初始化前的 wikidown 資料夾內容，您可以看到裡面只有專案的檔案與資料夾。

![圖、wikidown 專案內容的檢視](wikidownExplorer.jpg)

```
user@ASUS /d/git
$ cd wikidown2

user@ASUS /d/git/wikidown2
$ ls
LICENSE   README.md  node_modules   package.json  wikiServer.js
Procfile  app.json   npm-debug.log  web

user@ASUS /d/git/wikidown2
$ git init
Initialized empty Git repository in d:/git/wikidown2/.git/

user@ASUS /d/git/wikidown2 (master)
$ git add -A
warning: LF will be replaced by CRLF in web/db/pmag201503/md/focus3.
The file will have its original line endings in your working directory.
warning: LF will be replaced by CRLF in web/js/highlight.min.js.
The file will have its original line endings in your working directory.

user@ASUS /d/git/wikidown2 (master)
$ git commit -am "commit1"
[master (root-commit) 76bce1f] commit1
warning: LF will be replaced by CRLF in web/db/pmag201503/md/focus3.
The file will have its original line endings in your working directory.
warning: LF will be replaced by CRLF in web/js/highlight.min.js.
The file will have its original line endings in your working directory.
 117 files changed, 65987 insertions(+)
 create mode 100644 .directory
 create mode 100644 .gitignore
 create mode 100644 LICENSE
 create mode 100644 Procfile
 create mode 100644 README.md
 create mode 100644 app.json
 create mode 100644 npm-debug.log
 create mode 100644 package.json
 create mode 100644 web/config.js
 create mode 100644 web/css/bootstrap-theme.css
 create mode 100644 web/css/bootstrap-theme.css.map
....
 create mode 100644 web/wikidown.html
 create mode 100644 web/wikidown1.html
 create mode 100644 wikiServer.js

user@ASUS /d/git/wikidown2 (master)
$ git remote add origin https://github.com/ccckmit/wikidown2.git

user@ASUS /d/git/wikidown2 (master)
$ git status
On branch master
nothing to commit, working directory clean
```

到目前為止，這些 git 指令的動作都還是在本地端執行的，所以即使不連上網路也是可以正常執行。

但是接下來的動作，就要開始上傳了，所以我們要先進入 github 帳號創建出  wikidown2 這個專案，方法是在 github 網站右上角的 + 號上按一下，然後在網頁中填入下列訊息。

![圖、在 github 帳號內建立 wikidown2 的專案](wikidown_github_create.jpg)

接著在按下 create repository 這個按鈕後，該專案就建立了。然後我們就可以透過下列的 git push origin master 指令將 master 專案版本上傳到 github 上。

```
user@ASUS /d/git/wikidown2 (master)
$ git push origin master
Username for 'https://github.com': ccckmit
Password for 'https://ccckmit@github.com':
Counting objects: 97, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (89/89), done.
Writing objects: 100% (97/97), 1.10 MiB | 119.00 KiB/s, done.
Total 97 (delta 10), reused 0 (delta 0)
To https://github.com/ccckmit/wikidown2.git
 * [new branch]      master -> master
```

![圖、已經上傳完畢後的 wikidown2 專案(master版)](wikidown_github_project_page.jpg)

對於一般性的專案而言，這樣就已經完成了 github 專案的上傳動作了。

不過如果要將 github 當作靜態網站使用，那麼就還繼續要創建一個 gh-pages 的版本上傳上去，因為 git hub 規定只有在 gh-pages 這個分枝版本的內容，才能夠在不出現 github 管理功能的畫面下以靜態網站的方式呈現。

因此、我們必須用 git checkout gh-pages 這個指令，將專案分枝到 gh-pages 這個版本，然後再用 git push origin gh-pages 這個指令將 gh-pages 版本推到位於 origin 指定位址的 github 專案當中，這樣才算完整的發布專案與靜態網站。

```
user@ASUS /d/git/wikidown2 (master)
$ git branch gh-pages

user@ASUS /d/git/wikidown2 (master)
$ git status
On branch master
nothing to commit, working directory clean

user@ASUS /d/git/wikidown2 (master)
$ git checkout gh-pages
Switched to branch 'gh-pages'

user@ASUS /d/git/wikidown2 (gh-pages)
$ git merge master
Already up-to-date.

$ git push origin gh-pages
Username for 'https://github.com': ccckmit
Password for 'https://ccckmit@github.com':
Total 0 (delta 0), reused 0 (delta 0)
To https://github.com/ccckmit/wikidown2.git
 * [new branch]      gh-pages -> gh-pages
```

如此、就大功告成了，您可以在下列網址看到剛剛上傳靜態網站的 wikidown.html 網頁，並且透過 wikidown 系統瀏覽我們已經上傳的那些 markdown 文件，把 github 當成維基網誌系統來用了。

* <http://ccckmit.github.io/wikidown2/web/wikidown.html>

以下是我們輸入上述網址後所看到的畫面：

![圖、在 github 上的 wikidown2 靜態網站首頁](wikidown2homepage.jpg)

這樣，我們除了可以將專案上傳到 github 分享給開放原始碼社群之外，也可以透過 github 創建靜態網站，把 wikidown 當成維基網誌系統來用，可以說是一舉數得阿！

### 後記

以下是筆者將 wikidown.js 系統上傳到 github 成為靜態網站的教學錄影，對於想瞭解 github 上傳過程的人而言，應該會很有幫助。

* [YouTube影片：如何使用 git 與 github 建立開放原始碼專案與網站 -- 以 wikidown.js 為例 ]( http://youtu.be/ZzCDx28-P9Y)



