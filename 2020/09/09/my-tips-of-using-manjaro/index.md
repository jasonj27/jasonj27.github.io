# 使用Manjaro linux 的小訣竅


<!--more-->

## Yay - Aur helper in Arch linux and Manjaro

雖然Manjaro有內建Pamac來管理系統上的軟體，但是以我的使用經驗來說，Commamd Line的工具用起來還是比較少錯誤。我使用的程式是Yay，可以順便管理透過AUR安裝的程式。

### 設定安裝程式時使用的鏡像站伺服器(mirror server)

正確的設定可以在安裝或更新程式的時候，提高下載的速度

```shell
pacman-mirrors --country Taiwan
```

### 設定Yay選項的預設值

設定這些選項預設值可以在更新AUR上程式的時候，避免過程被中斷

```shell
 yay --nocleanmenu --nodiffmenu --noeditmenu  --save
```
## Chrome的語言設定
為了在使用Manjaro遇到問題時，可以找到更多問題的解答。我的系統語言時設定成英文。

Chrome預設的語言會使用系統設定，但是這樣會影響一些網站的語言，所以需要另外設定Chrome的語言。

[解答](https://askubuntu.com/questions/202670/change-google-chrome-language)

> In your `/usr/share/applications/google-chrome.desktop` change the "exec" command to something like this:
> ```
> sh -c "LANGUAGE=zh_TW /usr/bin/google-chrome-stable %U"
> ```

## Rime - 中文輸入法
待更新...
