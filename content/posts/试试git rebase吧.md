---
title: "试试git Rebase吧"
date: 2021-09-29T21:52:02+08:00
draft: false
original: false
categories: 
  - 工具
tags: 
  - git
---

# 背景

最近工作上接触到git rebase命令，之前也听说过，但是基本没使用过，因为 git pull,merge,commit,push 足以混了。今天找到一篇rebase个人感觉比较好的教程，所以偷偷转过来。一下是内容 [原文](https://medium.com/starbugs/use-git-interactive-rebase-to-organize-commits-85e692b46dd)：

不知道大家有沒有這個壞習慣：就是平常在開發 side project 時雖然會用 Git 做版控，但 commit message 都是亂寫一通（反正也沒人看嘛XD），什麼 Update code、Add some files 都出來了，光看 message 根本不知道做了什麼

而 commit 的大小方面，有些 commit 肥到不行，一次就修改個四五百行，好像恨不得一口氣寫完整個專案，因此根本無法進行 code review；而有些 commit 卻又小到只是修個 typo，明明不需要獨立成一個 commit，在寫其他功能時看到順便修一下就好了

以上這些情況如果只發生在自己的 side project 那就沒差。但如果是真的要在團隊內跟人合作，或是要參與 Github 上破千個 contributor 的開源專案，那這種類型的 commit 保證被幹譙到飛起來，不然就是被丟在一旁根本沒人想幫你 review

![](/试试gitrebase吧/1_JXZ3N6S_lGehIPPw1Xt-Vg.png)

但同樣身為工程師，我也知道腦袋進入開發模式之後，一晃眼就是幾千行起跳，中間根本沒時間慢慢 commit；或是有時太專心開發根本沒發現程式碼有 typo，只好另外開一個 commit 來修 typo

所以今天的主題就是要教大家怎麼用 Git rebase 整理 commit，除了避免送出 PR 之後面臨根本沒人理你的窘境之外，commit 整齊一點也會讓 bug 更好找、心情更愉悅，可說是一舉多得啊～

# git add --patch

在講 rebase 之前，這邊要先介紹一個待會會用到的指令 git add --patch，可以用來把一個檔案切成好幾次 commit

譬如說我想把這次 index.js 新增的內容分次提交，那就下 git add -p index.js，接著按 e 進到編輯畫面，有 + 開頭的就是你這次新增的內容

![](/试试gitrebase吧/1_fNYbZrf6Z1cImX3Dq1XGeg.png)

這時就可以把想 add 的加號留著，其他不想 add 的部分刪掉，如此一來 git 就只會把有加號的那幾行加進 staging area，而其他刪掉的部分就繼續留在 unstaged 狀態

![](/试试gitrebase吧/1_I7MAki4MiyYRq9044-gLxQ.png)

若是習慣用 GUI 的話，大部分的 IDE 也會提供 add patch 功能，只要把想加入 Staging 選起來，然後點 Stage Selected Ranges 就好了～

![](/试试gitrebase吧/1_Ov3qCb1JyQ8rsRTq03_SRw.png)

# Git rebase

接著就要進入本文的主題 git rebase ，怕大家單純用看的不好懂，所以我建了一個 repo 讓大家可以跟著操作，如果現在是用電腦的話可以先 clone 下來

```
git clone https://github.com/Larry850806/rebase-demo
```

clone 下來後專案內的 master 有 README.md、file1、file2 … file5 共 6 個檔案，而歷史紀錄長得像下圖：一開始先是 Init，接著連續新增五個檔案，接著就要用這個專案來嘗試 interactive rebase


> 因為是 clone 下來的，所以會連 commit ID 都跟圖中一樣哦

# 任務一：調整 commit 順序

沒問題之後我們就開始進行第一次 rebase，這次的目標是要嘗試調整 commit 的順序

首先下指令 git rebase -i 4a16df，-i 是 interactive 的意思，而 4a16df 是第一個 Init 的 commit ID，代表我要用 interactive rebase 來調整 Init 之後的 commit，按下 Enter 後就會看到這個編輯畫面（在 Vim 裡面）

![](/试试gitrebase吧/1_exrIRTUI81Pj-v_MEPpRjg.png)

這個畫面很重要：意思是現在的 master 是從 Init 開始，按照順序把這些 commit 的變更疊加起來而成的（從上到下）

所以想要變更 commit 的順序也很簡單，只要調整 pick 的順序，把 Add file3 移到最下面，Git 就會按照 1 -> 2 -> 4 -> 5 -> 3 的順序把 commit 疊加起來，產生最新的 master

![](/试试gitrebase吧/1_ZXACyo6QtHWZEZBXVVr6Sg.png)

儲存退出後就完成了第一次 rebase，很簡單吧！完成後 master 會變成下面那個 branch ，順序是 1 -> 2 -> 4 -> 5 -> 3，從歷史紀錄也可以看出來 Add file3 確實變成最後一個 commit 了

![](/试试gitrebase吧/1_SJF8O_SsMtHBFjOj5f2mTA.png)

結束 rebase 之後，之前 master 的所在位置會被改名叫 ORIG_HEAD，所以如果對於 rebase 後的結果不滿意的話，只要下 git reset --hard ORIG_HEAD 就能回到之前 1 -> 2 -> 3 -> 4 -> 5 的順序

所以要調整 commit 的順序，其實就是進入 interactive rebase，接著調整一下 pick 的順序就可以了～如果對於怎麼操作還是不太清楚，也可以看看這個我錄的三十秒小短片，看完應該就會囉


![](/试试gitrebase吧/gitrebasereorder.gif)


# 任務二：刪除 commit

剛剛有提到 git rebase 時 git 會按照順序把 commit 疊加起來，所以要刪掉某一個 commit 也很簡單，就是在 rebase 時把前面的 pick 指令改成 drop 就好了。當 git 從上往下讀取到 drop 指令，他就會直接把那個 commit 丟掉，只把 pick 開頭的 commit 撿去用

舉個例子，假如我今天要把 Add file4 跟 Add file5 兩個 commit 刪掉，那就一樣下 git rebase -i 4a16df，接著把不要的 commit 改成 drop

![](/试试gitrebase吧/1_64R457PzH6wbHOzcKPJgJA.png)

rebase 執行完之後，因為 Add file3 是最後一個 commit，所以 master 就會被移到 Add file3 的地方，file4 跟 file5 也會從資料夾中消失

![](/试试gitrebase吧/1_h9XjroipNi-DK5H0Y5VNHA.png)

而 ORIG_HEAD 就跟先前一樣被 Git 放到 rebase 之前的位置，所以如果不小心誤刪了很重要的 commit，不要緊張，只要 git reset --hard ORIG_HEAD 就好了～

到這邊大家應該對於 rebase 更有概念了，簡單來說就是先挑一個 rebase 的基準點（在這邊是 Init），然後就可以對他後面的 commit 進行各種調整，那 rebase 還有支援哪些指令呢？讓我們繼續看下去～

# 任務三：修改 message

除了調整順序、刪掉 commit 之外，interactive rebase 也有一個指令 reword 可以用來修改 commit message

譬如說我想把 Add file4 改成 Finish file4，那就在 rebase 時把 pick 改成 reword，那 Git 在套用那個 commit 時就會自動打開你的編輯器（我是 Vim）讓你改，改完之後他再繼續 pick 後面的 Add file5

![](/试试gitrebase吧/1_wvPM8nnhojA6MHPhl7Yj0A.png)

改完的歷史紀錄長這樣：因為 commit ID 有部分是經由 message hash 而成，所以 reword 並不會改寫原本的 commit，而是產生另外一個新的 commit 跟 branch，並且把 master 移到新的 branch 之上

![](/试试gitrebase吧/1_mBFZngRecwWAa_FFzDc41g.png)

若對於 reword 的結果不滿意，那就 reset 回去 ORIG_HEAD 就好了，反正所有 rebase 前的 commit 都會儲存在 .git 資料夾裡面（想不見都很難XD），rebase 只是把 master 換到另外一個 branch 而已

如果覺得看圖不夠清楚的話，可以看看這個 20 秒小短片，看完就知道怎麼修 message 了～

![](/试试gitrebase吧/gitrebasereword.gif)

> 說實話這大概是我最常用到的 rebase 指令，因為我在開發時 message 基本上都是亂寫一通（有很多人也是吧XD），如果不修一下直接 push 上去，隔天睡起來我可能就不知道那個 commit 在幹嘛了

# 任務四：合併 commit

有時若在送出 PR 之前發現一些小 typo，或是覺得 commit 太小太散了，那也可以用 rebase 來合併多個 commit

譬如說我想把 Add file3 跟 Add file4，那就把 Add file4 的 pick 改成 squash，代表我想要把他跟上一個 commit(Add file3) 合併

![](/试试gitrebase吧/1_Ugy9Bws4qHlozV2swZbQcw.png)

執行之後就會看到下圖的畫面，只要把原本的兩個 commit message 刪掉，然後寫上 Add file3–4 就好了～

![](/试试gitrebase吧/gitrebasesquash.gif)

rebase 結束後 Git 會產生一個新的 Add file3-4，他的變更內容就是原本兩個 commit 變更內容的總和（就這樣，超簡單XD）。然後 ORIG_HEAD 一樣會在原地，不滿意隨時可以 hard reset 回去

![](/试试gitrebase吧/1_DoblRP8KBfQb-S3yB1sEAg.png)

想看實際操作的話可以看看這個小短片，用起來真的很簡單～

![](/试试gitrebase吧/gitrebasesquash1.gif)

# 任務五：拆分 commit

接著就要進入今天的大魔王：拆分 commit。因為一個 commit 可以有好幾種拆法，譬如說五五分、三七分等等，不像合併就是兩個 commit 湊起來就完成了，所以過程也複雜許多

如果我想要把剛剛的 Add file3–4 拆回來，那就要用到 edit 指令，他的功能是這樣的：Git 在遇到 edit 指令時，他會先套用那個 commit，接著就先暫停下來，一直到我下 git rebase --continue 才會繼續 rebase

所以作法上會是這樣：我要等 Git 在套用完 Add file3–4 之後會暫停，然後馬上用 reset 把 Add file3–4 拆掉，接著手動分別 commit 兩個檔案

![](/试试gitrebase吧/1_GXkqfyfw42AapHFRhHoXwg.png)

如下圖，在執行完 edit b19b0e5 Add file3–4 之後，Git 會套用 Add file3-4 並且暫停下來，我們要趁這個卡在中間的時候把 Add file3–4 拆成 Add file3 跟 Add file4

![](/试试gitrebase吧/1_5J1ZPJ8JlMBUsptwDqBSGg.png)

因為目前是暫停在 b19b0e5(Add file3-4) 結束之後，所以這時可以直接用 git reset @^ 把 Add file3–4 拆掉，讓 file3 跟 file4 掉回 unstage 區

![](/试试gitrebase吧/1_LUaEEcfNZB4KWhiM6jXvWg.png)

拆掉後，再分兩次 commit 他們兩個檔案，到這裡我們已經把 Add file3–4 拆成 Add file3 跟 Add file4 了

![](/试试gitrebase吧/1_BVzs7u5ESTrumwuCNtTnnA.png)

但因為 rebase 還沒完全結束，剛剛是做到一半暫停，後面還有一個 pick 8c96f5 Add file5 要做，所以要下 git rebase --continue 讓 rebase 繼續，完成後看 git log 就可以看到 Add file3 跟 Add file4 又被拆回來了～

![](/试试gitrebase吧/1_-JyoZU_aMvccT0EdttjCpA.png)

edit 指令算是 interactive rebase 裡面比較複雜的，所以我也錄了一個小短片，看影片跟著動作做幾次就會覺得沒那麼難了

![](/试试gitrebase吧/gitrebaseedit.gif)

一開始用 edit 時常常會搞到暈頭轉向不知道自己在哪個 commit，但熟了之後就會發現他真的很神，可以讓你在歷史紀錄裡面飛天遁地，想改哪就改哪，也可以對過去 commit 做一些奇怪的事（修改時間、作者、訊息等等）

另外，雖然這邊沒有示範，不過只要用一樣的方法搭配開頭講的 git add --patch，就可以把一個巨大檔案拆成好幾個 commit 囉

#  總結

今天關於 rebase 的介紹就到這裡，總共講了 pick、drop、reword、squash、edit 五個指令以及他們的 use case，這些指令不用硬記（我有時也會忘記XD），因為 rebase 的介面會把所有可用的指令都列出來（下圖），如果曾經用過看一眼就會想起來了～

![](/试试gitrebase吧/1_1m21EuQu4sU6ch3PJMKcyQ.png)

另外，雖然有些指令像 fixup、exec 沒有講到，但只要看一下他的敘述再試一下應該就會用了，反正如果不小心 rebase 壞了，那大不了再 hard reset 回 ORIG_HEAD 就好XD，就像什麼事情都沒發生過，我想這種近乎時光機的功能就是使用 Git 最大的好處吧

最後再提醒一下，因為 rebase 會變更歷史紀錄，所以最好是在 push 之前就做好 rebase，不然就是確保要這個 branch 只有你自己在修改，否則你也 rebase、他也 rebase，最後只會把歷史紀錄搞得一團亂哦XD

就這樣，希望今天的內容對大家有幫助，如果對 Git rebase 的其他用法有興趣的話，可以再看看底下的延伸閱讀～