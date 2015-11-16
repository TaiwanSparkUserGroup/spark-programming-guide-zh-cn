* 參考[swift正體中文專案](https://github.com/tommy60703/the-swift-programming-language-in-traditional-chinese)的貢獻方式
貢獻力量
--------
如果想做出貢獻的話，你可以：

- 幫忙校對，挑錯別字、語病等等
- 提出修改建議
- 提出術語翻譯建議

翻譯建議
--------
如果你願意一起校對的話，請仔細閱讀：

- 使用 markdown 進行翻譯，文件名必須使用英文，因為中文的話 gitbook 編譯的時候會出問題
- 引號請使用「」和『』
- fork 過去之後新建一個分支進行翻譯，完成後 pull request 這個分支，沒問題的話我會合併到 master 分支中
- 有其他任何問題都歡迎發 issue

關於術語
--------
翻譯術語的時候請參考這個流程：

- 盡量保證與台灣習慣術語和已翻譯的內容一致
- 盡量先搜尋，一般來說程式語言的大部分術語是一樣的，可以參考[這個網站](http://jjhou.boolan.com/terms.htm)
- 如果以上兩條都沒有找到合適的結果，請自己決定一個合適的翻譯或者直接使用英文原文，後期校對的時候會進行統一
- 校稿時，若有發現沒有被翻譯成台灣術語的大陸術語，可以將它新增到 translation.json 中
- 可以主動提交替換過的文本給我，或是僅提交新增過的 translation.json 也可，我會再進行文本的替換
- 請務必確定提交的翻譯對照組不會造成字串循環替代（ex: 因為「類」->「類別」，造成下次再執行自動翻譯時「類別」又變成「類別別」）

對翻譯有任何意見都歡迎發 issue，我看到了會盡快回覆

參考流程
--------
有些朋友可能不太清楚如何幫忙翻譯，我這裡寫一個簡單的流程，大家可以參考一下：

1. 首先 fork 我的項目(在專案畫面右上角)
2. 把 fork 過去的項目也就是你的項目 clone 到你的本機端
3. 在命令行運行 ``git branch develop`` 來創建一個新分支
4. 運行 ``git checkout develop`` 來切換到新分支
5. 運行 ``git remote add upstream https://github.com/TaiwanSparkUserGroup/spark-programming-guide-zh-tw.git`` 把我的庫添加為遠端庫
6. 運行 ``git remote update`` 更新
7. 運行 ``git fetch upstream`` 拉取我的庫的更新到本地
8. 運行 ``git merge upstream/master`` 將我的更新合併到你的分支

這是一個初始化流程，只需要做一遍就行，之後請一直在 develop 分支進行修改(可以針對不同的修改開不同的分支)。

每次要修改貢獻之前，請重複 6、7、8 步，將master更新到最新狀態之後，再開新的分支進行修改。

提交新版本
--------
如果你有更新項目需要提交，請參考以下流程：

1. commit 你的更新，並於更新訊息簡單說明更新的內容。如果更新很多檔案，請分批提交
2. 首先 push 到你的Github
3. 然後登錄 GitHub，在你的 repo 的首頁可以看到一個 ``pull request`` 按鈕，點擊它，填寫一些說明資訊，然後提交即可。

常見問題
--------
Q: merge 衝突啦 怎麼辦
A: 每次要修改前，請先重新pull遠端的程式庫，保證目前是最新的版本。如果發生錯誤請參考[這篇](https://github.com/doggy8088/Learn-Git-in-30-days/blob/master/docs/17%20%E9%97%9C%E6%96%BC%E5%90%88%E4%BD%B5%E7%9A%84%E5%9F%BA%E6%9C%AC%E8%A7%80%E5%BF%B5%E8%88%87%E4%BD%BF%E7%94%A8%E6%96%B9%E5%BC%8F.markdown)
