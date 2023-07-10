---
title: 用 Cowrie 來架 SSH Honeypot
date: 2023-6-13 17:15:00 +0800
categories: [Implement]
tags: [implement, honeypot, cowrie]
---

# 好久不見

好久沒更新文章了，原本要在 Medium 做技術與筆記相關的文章發表，結果自從跑去做其他更有趣的事情之後，就把 Medium 放一邊了，哈哈<br>

今年興起了自己架站的想法，所以就把 Medium 的文章遷移到這裡了，看來原本被荒廢的 Blog 是要強勢回歸了<br><br>

由於筆者最近部屬一台 Cowrie 並暴露在 Internet 上，目的是收集並整理成最新的 SSH 帳號、密碼情資，所以就誕生出這篇文章啦<br>

---

# 實作之前，需要做哪些準備?

## 知識

本篇的內容相對粗淺，所以只要對 Linux 的操作有概念就可以開始本次的實作，但若想點個相關的技能點的話: <br>

1. **Linux 操作**<br>
Linux 訓練唯一推薦鳥哥[^vbird-linux_basic_train]，把前 10 章學完對本篇而言就很夠用了
2. **Honeypot 概念**[^wikipedia-honeypot] <br>
光是 Google: honeypot、蜜罐、誘捕系統 以上關鍵字，就可以取得相當充沛的資訊

## 設備

- 效能需求低，筆者使用 VMware 架設以下規格的 VM:
    - 2 Core CPU
    - 4 GB Memory
    - 20 GB Disk
    - Ubuntu 20.04

>不一定要參照筆者的規格，只需要 Ubuntu 或 Debian 的 Linux 機器就好
{: .prompt-tip }

---

# Honeypot/Cowrie[^cowrie-documentation]

- Honeypot(蜜罐) 是指刻意用來偵測或抵禦資安攻擊的陷阱，可以部屬在機器或服務(Service)上，而本篇將會透過 Cowrie 實作 SSH 服務的 Honeypot
- Cowrie 提供高互動性的 Shell 模擬環境來捕獲與紀錄攻擊者進入機器後的行為，使攻擊者在認為成功登入時，實際上卻誤入了事先部屬好的模擬環境之中，但仔細推敲其實仍會發現模擬環境與真實環境不同的地方

# Cowrie 部屬教學

事不宜遲，接下來一步一步帶領部屬的流程

1. **首先，當然先安裝所有的依賴套件:** <br>
`sudo apt-get install -y git python3-venv libssl-dev libffi-dev build-essential libpython3-dev python3-minimal authbind virtualenv`
2. **(Optional)建立一個名叫 cowrie 的 User:** <br>
這並非必要步驟，但強烈建議使用限縮權限的使用者來部屬 Cowrie<br>
`sudo adduser --disabled-password cowrie`
    
    ![Desktop View](/assets/img/2023-06-08-deploy_a_ssh_honeypot_with_cowrie/figure_1.png){: .normal }
    
3. **切換使用者至 cowrie:** <br>
`sudo su - cowrie` 
4. **把 Cowrie 透過 git 給 clone 下來:** <br>
`git clone http://github.com/cowrie/cowrie`
5. **將工作目錄切換至 Cowrie 的目錄下:** <br>
原先應該在 `/home/cowrie` 的目錄下，或你要用來部屬 Cowrie 的使用者的 home 目錄下<br>
`cd ./cowrie` 
6. **透過 venv 建立名為 cowrie-env 虛擬環境，用以獨立運行 Cowrie 的部屬環境與套件:** <br>
`python3 -m venv cowrie-env` 
7. **啟用上一步建立的虛擬環境:** <br>
你會注意到啟用虛擬環境後，在 $(Prompt) 符號前，會多了 `(cowrie-env)` 的提示(如同下方附圖)，這表示目前已切換至指定的虛擬環境中(順帶一提，離開虛擬環境的指令是 `deactivate`)<br>
`source cowrie-env/bin/activate`
    
    ![Desktop View](/assets/img/2023-06-08-deploy_a_ssh_honeypot_with_cowrie/figure_2.png){: .normal }
    
8. **在虛擬環境內，將 pip 這個套件管理器升級到最新:** <br>
`python3 -m pip install --upgrade pip`
9. **在虛擬環境內，依指定的套件清單文件(requirements.txt)來安裝指定的 python 依賴套件與版號:**<br>
`python3 -m pip install --upgrade -r requirements.txt`
10. **再加碼安裝額外的套件與指定版本:** <br>
實際上 cowrie 的官方文件並沒有這一段，但若是全依照官方文件部屬會發現最後運行 cowrie 時會跳出錯誤。
筆者最初經過一陣爬文 debug 後，透過此指令安裝指定的套件與版號來解決此議題<br>
`python3 -m pip install cryptography==36.0.2 pyOpenSSL==22.0.0` 
11. **運行 cowrie:** <br>
`bin/cowrie start`
12. **為了測試 Cowrie 是否順利運行，接下來請透過 ssh 連到部屬 Cowrie 的 2222 port 上吧:** <br>
由於 Cowrie 預設是在 2222 port  ，所以請注意測試時要接入 2222 port <br>
帳號使用 root，而密碼就任意輸入就好<br>
`ssh root@localhost -p 2222`
13. **若成功連入名為 svr04 的環境內，恭喜，Cowrie 已經部屬並運行了**

> 為甚麼明明沒有設定 User/Password ，就可以直接登入呢?因為 Honeypot 的目的之一就是蒐集攻擊者的特徵，那當然要讓攻擊者有辦法進到環境好好的操作一番，而 Cowrie 預設就有準備幾個常見的 User (root, tomcat, oracle) ，可以透過任意密碼登入。
{: .prompt-info}

# (Optional) 將 cowrie 設定在 22 port 上

畢竟是用作 ssh 的 honeypot，讓 cowrie 與 ssh 的特徵統一很重要，接下來將 Cowrie 設定在 ssh 的 default port 上，而我們將會透過 iptable 的 prerouting 來進行設置，簡而言之就是將原本要連到 22 port(ssh) 的封包重新導向到 2222 port(cowrie) 上

>這並非必要步驟，如果建立 Cowrie 僅是用作個人測試或是練習用途，那透過 2222 port 就可以連入了
{: .prompt-info }

1. 因為接下來要執行的指令需要高權限，所以請先確保執行指令的使用者能夠使用 `sudo` 指令，或是使用 root 進行
2. 必須先將機器上的 22 port 給換到別的 port 上，在這個範例中我們設定的是 8888 port，否則 cowrie 一旦先設定過來，你就無法透過 ssh 連回到機器內了: <br>
`sudo iptables -t nat -A PREROUTING -p tcp --dport 8888 -j REDIRECT --to-port 22`
3. 再來將 2222 port 的 cowrie 給導向到 22 port，以偽裝成一個正常的 ssh service，就大功告成了: <br>
`sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222` 

# Cowrie 的 Shell 模擬環境測試

- 模擬環境的測試效果可以參考下圖:
    
    ![Desktop View](/assets/img/2023-06-08-deploy_a_ssh_honeypot_with_cowrie/figure_3.png){: .normal }
    
    **測試流程解說:** <br>
    
    1. 成功登入 Cowrie，可以觀察到在預設配置下 SSH 模擬環境的 Hostname 是 svr04
    2. 嘗試 ping 8.8.8.8，
    相信大家都會用這個方法來測試機器網路是否能夠對外
    3. 嘗試 ping 1.2.3.4
    這時候會發現，不應該 ping 得到結果的 IP(1.2.3.4) ，竟然煞有其事出現了結果
    4. 嘗試 ping 不存在的域名
    就連亂打的 Domain 都有 IP 可以 Ping 出結果
    
    **測試結論:** <br>
    
    - 可以觀察到在模擬環境中，操作指令的流程與指令與真實環境並無明顯差異
    - 但畢竟是模擬環境，所以執行 `ping` 這類需要向外互動的指令時，Cowrie 也只能”假裝”你可以互動並取得數據，使得不應該成功的 `ping 1.2.3.4` 出現了成功的錯覺
    - 實際上，這個議題在 Sandbox(沙盒)[^wikipekia-sandbox] 之中也會出現。

<br><br><br>

若是要盡可能地隔離 Cowrie 的環境，筆者建議可以透過 Docker 來部屬，而且 Cowrie 還有很多配置可以實現客製化需求，包含啟用 Telnet、傳送告警訊息至 Discord 等等功能。

但部屬相關的教學就佔了很大的篇幅了，而筆者希望單篇文章的閱讀量可以控制在 15 分鐘內，所以針對客製化 Cowrie 的部分，會分開在 <這邊應該是個連結> 中介紹。 

謝謝各位，各位掰掰👋

---

# Reference

[^wikipedia-honeypot]: [Wikipedia-Honeypot(computing)](https://en.wikipedia.org/wiki/Honeypot_(computing))

[^cowrie-documentation]: [Cowrie-Documentation](https://cowrie.readthedocs.io/en/latest/README.html#documentation)

[^wikipekia-sandbox]: [Wikipekia-Sandbox](https://en.wikipedia.org/wiki/Sandbox_(computer_security))

[^vbird-linux_basic_train]: [蔡德明，鳥哥私房菜-Linux 基礎學習篇訓練教材 - 目錄彙整](https://linux.vbird.org/linux_basic_train/centos7/unit01.php)
