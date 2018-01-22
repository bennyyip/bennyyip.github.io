title: Linux 初體驗及中州韻配置小記
published_date: "2017-02-10 00:20:47 +0800"
layout: post.liquid
data:
  route: blog
---
# 緣起
近期打算遷移至 Arch Linux， 在虛擬機器體驗了幾天，等開學之後配部桌上型電腦再正式叛逃過去。 
在這幾天裡，我試用了相當數目的新軟體(vim之類的早在用的就不說了），在這裡列個清單備忘 

---

- urxvt：終端仿真器
- tmux：終端復用器
- moc：音訊播放器
- music box：網易雲終端版
- i3：window manager
- conky：kickass system monitor
- feh：圖片瀏覽器
- zathura：vi-like PDF reader
- dmenu：launcher
- 中州韻（Rime）：中文輸入法
- ydcv：Yaodao Dictionary Console Version
- stow：使用符號連結（symbolic link）管理軟體包的工具，我主要用它和 git 一起管理配置檔案。

上訴大部分軟體都是沒有 GUI 的，活在終端的感覺真好。

高中時曾在 Windows 7 下短暫使用過一陣時間的 Rime，但是由於對 dota 2 支援不好<del>垃圾 dead game</del>，沒繼續用下去。
後來到了 Windows 10 在 modern app 下無法使用，漸漸將它遺忘。  
講真，Rime 是我在 PC 上用過最爽的輸入法，如果你沒試過而且在使用 Linux 或者 macOS，請務必試試中州韻/鼠鬚管（Rime 在這兩個平臺上的中文名）。  
我這次更新網誌的動機很大程度上是因爲中州韻打字實在太爽快了XD

# 中州韻的安裝及配置
事不宜遲，我們現在就切入正題
我選擇使用 `fcitx-rime` 這個包，原因是我認識的 Linux 用家裡用 fcitx 的比 ibus 多，我相信他們的選擇。
```bash
sudo yaourt -S fcitx fcitx-im fcitx-configtool fcitx-rime
```
其中 fcitx-configtool 是 fcitx 的 GUI 配置工具。安裝好之後需要通過它用中州韻。

首先中州韻的自帶詞庫不夠豐富，我找來了官方提供的一份詞庫（https://github.com/rime-aca/dictionaries ）   
按照 README.md 的說明，`luna_pinyin.dict` 資料夾下的所有檔案複製到配置目錄（~/.config/fcitx/rime/)，然後再做後續的配置。

需要配置的檔案有以下三個：
- `double_pinyin_flypy.custom.yaml`
- `default.custom.yaml` 
- `luna_pinyin.extended.yaml` 

```yaml
# luna_pinyin.extended.yaml 

---
name: luna_pinyin.extended
version: "2015.12.02"
sort: by_weight
use_preset_vocabulary: true
#此處爲明月拼音擴充詞庫（基本）默認鏈接載入的詞庫，有朙月拼音官方詞庫、明月拼音擴充詞庫（漢語大詞典）、
#明月拼音擴充詞庫（詩詞）、明月拼音擴充詞庫（含西文的詞彙）。如果不需要加載某个詞庫請將其用「#」註釋掉。
#雙拼不支持 luna_pinyin.cn_en 詞庫，請用戶手動禁用。
import_tables:
  - luna_pinyin
  - luna_pinyin.hanyu
  - luna_pinyin.poetry
  # - luna_pinyin.cn_en
...
```

雙拼用家可以將上面的個檔案的 `luna_pinyin.cn_en` 一行註釋掉。 

```yaml
# double_pinyin_flypy.custom.yaml

patch: 
  # 雙拼碼不轉換成全拼碼 
  "translator/preedit_format": {}
  # 載入朙月拼音擴充詞庫
  "translator/dictionary": luna_pinyin.extended
```

```yaml 
# default.custom.yaml 

patch:
  schema_list:
    - schema: double_pinyin_flypy

  ascii_composer:
    switch_key:
      Shift_L: noop
      Shift_R: commit_code

  key_binder:
    bindings:
      - { when: has_menu, accept: Shift_L,  send: 2 }
      - { when: has_menu, accept: Control_R, send: 3 }
      - { when: has_menu, accept: equal, send: Page_Down }
      - { when: has_menu, accept: minus, send: Page_Up }

```

我討厭使用左 shift 切換中英文（因爲常常誤觸），於是把它禁用掉了，換成右 shift。  
這也是我喜歡 Rime 的一大原因，它提供了大量的配置選項，你可以把它調教成你喜歡的樣子。 
我用過的大部分輸入法都不能設定成只用右 shift 切換中英文。  
另外，我將左 shift、右 ctrl 分別設定成上屏第二、三個候選詞，較爲方便。
未來可能把結合輔碼，將每頁候選詞減少到三個。

# For Vim User
這篇網誌是在 Vim 中完成的，我抄了下面的一段程式碼到我的 Vim 配置裡。它實現了進入普通模式自動切換成無輸入法的狀態，而在插入模式保持著上一次進入普通模式前的狀態。

```Vim
"##### auto fcitx  ###########
let g:input_toggle = 0
function! Fcitx2en()
   let s:input_status = system("fcitx-remote")
   if s:input_status == 2
      let g:input_toggle = 1
      let l:a = system("fcitx-remote -c")
   endif
endfunction

function! Fcitx2zh()
   let s:input_status = system("fcitx-remote")
   if s:input_status != 2 && g:input_toggle == 1
      let l:a = system("fcitx-remote -o")
      let g:input_toggle = 0
   endif
endfunction

set ttimeoutlen=150
"退出插入模式
autocmd InsertLeave * call Fcitx2en()
"進入插入模式
autocmd InsertEnter * call Fcitx2zh()
"##### auto fcitx end ######
```
