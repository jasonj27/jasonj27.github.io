# 更新雙重開機(Windows, Linux)設定的shell script


<!--more-->

## 用途

一般來說，如果在安裝Windows之後再安裝Ubuntu或Manjaro這些可以自動安裝的Linux，開機時應該就會出現GRUB的選單可以選擇要進入Windows或Linux。也可以在BIOS中設定預設要進入的系統。

但是在某些筆記型電腦上，只有安裝一顆硬碟時，就算在BIOS中選擇Linux作為預設值，開機時還是會進入Windows。

唯一可以設定Linux作為預設開機的解決方式。是把Windows的開機efi檔更名，再把Linux(GRUB)的開機efi檔複製一份到原來Windows開機efi的相同位置。這樣就算是系統執行的是Windows的efi檔，還是可以照我們想要的從Linux開機。另外還要更新Linux多重開機管理程式GRUB設定檔中Windows的efi檔位置，這樣才可以在GRUB選單中正確啟動Windows。

但是，在做完這些設定後，Windows更新時有機會重設Windows的開機efi。有時候Linux更新Kernel的時候，也會重設GRUB的設定檔。所以這些時候就要重複上面這些設定。

這個shell script就是將上面這些步驟自動化。

## Script
{{< gist jasonj27 3552a983f07f9f23e37d1660250b7dfb >}}

## 使用方法
1. 把上面code複製一份到想存放的目錄中
2. chmod 讓檔案變成可以執行
```shell
chmod +x grub.sh
```
3. 執行
```shell
./grub.sh
```
