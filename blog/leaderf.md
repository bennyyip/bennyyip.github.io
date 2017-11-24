extends: post.liquid

title:   安利 LeaderF
date:    24 Nov 2017 22:49:25 +0800
route:   blog
---


# LeaderF 是什麼
[LeaderF][3] 是一個用於模糊搜索的 Vim 插件（fuzzy finder），可以用來找文件、打開的 buffer、最近打開的文件（MRU）等等。甚至，你可以給它寫拓展，找你想找的東西（我寫過兩個 LeaderF 插件，後面會提到）。

![NameOnly Mode][1]

![FullPath Mode][2]

# 爲什麼是 LeaderF
其實同類的 Vim 插件很多，光我用過的就有 CtrlP，Denite，fzf，它們各有各的缺點：
- CtrlP：純 Vim 實現，速度奇慢。
- Denite： CtrlP 繼任者。它使用 Python 實現，但是反應速度依然不夠快。並且根據我給它和 LeaderF 開發插件的經驗，寫 LeaderF 的插件比較容易。
- fzf：fzf 一個用 Golang 寫的 fuzzy finder 程序，vim 插件只是把它集成了，性能相當不錯。它的問題在於有外部依賴以及配置麻煩，因此我沒選擇用它。

那 LeaderF 有哪些優勢呢？
- 使用 Python 和 Vimscript 實現，只要需要 Vim 版本大於 7.4 並且編譯了 Python 支持。
- LeaderF 提供了 C 的 fuzz match 算法，可以用來替換原來用 Python 寫的版本。儘管 Python 版本已經非常快了，作者宣稱有 10 倍的提速。
- 使用緩存。只有第一次查詢的時候會取獲取數據，後續都只是返回緩存好的數據，除非按 F5 讓它刷新數據。
- 拓展性好，寫拓展相當方便。
- 同時支持 regex 和 fuzz 兩種模式。

# 拓展
LeaderF 的作者提供了一個拓展的例子 [LeaderF-marks][4]，是一個跳轉到 mark 的插件。
我照貓畫虎，做了 [LeaderF-ghq][6] 和 [LeaderF-github-stars][5]。
[ghq][7] 是一個管理 GitHub 倉庫的一個軟件，我寫的插件是用來快速跳轉到這些倉庫。
GitHub Stars，顧名思義，用來在瀏覽器中打開你在 GitHub star 過的項目。

# Color Scheme
我不喜歡 LeaderF 的默認配色，做了一個 [gruvbox 配色][8]。
如果你也在用 gruvbox，可以考慮使用這個配色，因爲它依賴 gruvbox。

![leaderf gruvbox](/img/leaderf-gruvbox.png) 

  [1]: https://github.com/Yggdroot/Images/blob/master/leaderf/leaderf_1.gif
  [2]: https://github.com/Yggdroot/Images/blob/master/leaderf/leaderf_2.gif
  [3]: https://github.com/Yggdroot/LeaderF
  [4]: https://github.com/Yggdroot/LeaderF-marks
  [5]: https://github.com/bennyyip/LeaderF-github-stars
  [6]: https://github.com/bennyyip/LeaderF-ghq
  [7]: https://github.com/motemen/ghq
  [8]: https://github.com/bennyyip/dot-vim/blob/master/after/autoload/leaderf/colorscheme/gruvbox.vim


