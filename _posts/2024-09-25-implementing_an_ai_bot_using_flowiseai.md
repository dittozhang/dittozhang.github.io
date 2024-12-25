---
title: 透過 FlowiseAI 實作 AI Bot
date: 2024-9-25 23:00:00 +0800
categories: [Implement]
tags: [implement, flowiseai, series]
---
# ~~兄弟~~ 我回來啦

接下來筆者會針對在 HITCON 2024 舉辦的 AI Challenge 發表系列文章，其中的基礎就是 FlowiseAI[^website-flowiseai]

先發表這篇，讓大家可以對 Flowise 有點概念，也增加網路上 Flowise 的教學資源 

---

- FlowiseAI 是個簡化 LLM 應用開發流程的開源工具，可以協助開發者將多個 LLM 組件串接成一個流程，或是將不同的模型功能組合在一起
- 簡而言之，FlowiseAI 透過 GUI 拖曳元件的作業模式，來快速實作一個客製化的 LLM 流程或 AI 客服，減少了 Coding 的功夫，而且毋須付費、部署又快速，可以說是實作 LLM 相關需求的好幫手
- 本文將教學如何部屬 FlowiseAI 與實作一個簡易的 Chaflow，並且實現中翻英的功能

## 前置條件

1. 安裝 [Docker](https://www.docker.com/get-started/) 與 [Docker Compose](https://docs.docker.com/compose/)
2. 準備 [OpenAI API Key](https://platform.openai.com/api-keys)，並確保該 API Key 還有額度可以使用

## 如何部署

筆者推薦使用 Docker 進行部署，簡單又快速
- 首先下載 [FlowiseAI Repository](https://github.com/FlowiseAI/Flowise.git)
- 進入 `Flowise/docker`
- 從 .env.example 複製一個 .env 到同一個目錄下
- 依個人需求修改 .env 的組態設定，值得關心的有
    - DATABASE_ 相關
    - FLOWISE_USERNAME 與 FLOWISE_PASSWORD
- 當前目錄執行 `docker compose up -d` 就大功告成了
- 打開 [http://localhost:3000/](http://localhost:3000/) 開始使用

    > 若要關閉就回到 Flowise/docker 下 `docker compose down` 就可以了
    {: .prompt-info}

    ![Desktop View](/assets/img/2024-09-25-implementing_an_ai_bot_using_flowiseai/001.png){: .normal }
    

# 新增 Credential

由於接下來我們會使用 OpenAI 的 LLM，所以把 OpenAI 的 API key 設定上來
1. 點選左側 Credentials 進入介面
2. 點選 Add Credentials 來新增

    ![Desktop View](/assets/img/2024-09-25-implementing_an_ai_bot_using_flowiseai/002.png){: .normal }
    
3. 搜尋 `openai` 點選 OpenAI API
4. 設定 Credential Name 與 Key 並新增

    ![Desktop View](/assets/img/2024-09-25-implementing_an_ai_bot_using_flowiseai/003.png){: .normal }
    
5. 畫面就會出現剛剛設定的 Credential ，完成

    ![Desktop View](/assets/img/2024-09-25-implementing_an_ai_bot_using_flowiseai/004.png){: .normal }
    

# 開發 Chatflows
1. 點選左側 Chatflows 
2. 點選 Add New 新增

    ![Desktop View](/assets/img/2024-09-25-implementing_an_ai_bot_using_flowiseai/005.png){: .normal }
    
3. 開發新的 Chatflow 之前，先點儲存，並為此 Chatflow 命名

    ![Desktop View](/assets/img/2024-09-25-implementing_an_ai_bot_using_flowiseai/006.png){: .normal }
    
4. 新增節點 LLM Chain
    Chain 是 Chatflow 中最基礎的節點，接下來會新增其他節點與 LLM Chain 串接
    1. 點選左側 Add Node
    2. 搜尋 `LLM`
    3. 將搜尋結果的 LLM Chain 物件拖曳到畫面內

    ![Desktop View](/assets/img/2024-09-25-implementing_an_ai_bot_using_flowiseai/007.png){: .normal }
    
5. 新增節點 Language Model
    1. 點選左側 Add Node
    2. 搜尋 `chatopenai`
    3. 將搜尋結果的 ChatOpenAI 物件拖曳到畫面內

    ![Desktop View](/assets/img/2024-09-25-implementing_an_ai_bot_using_flowiseai/008.png){: .normal }
    
6. 新增節點 Chat Prompt Template 
    1. 點選左側 Add Node
    2. 搜尋 `prompt`
    3. 將搜尋結果的 Chat Prompt Template  物件拖曳到畫面內

    ![Desktop View](/assets/img/2024-09-25-implementing_an_ai_bot_using_flowiseai/009.png){: .normal }
    
7. 手動點選，將所有節點與 Chain 串接
    - ChatOpenAI(Output) → LLM Chain(Language Model) 
    - ChatPromptTemplate → LLM Chain(Prompt)
    
    ![Desktop View](/assets/img/2024-09-25-implementing_an_ai_bot_using_flowiseai/010.png){: .normal }
    
8. 設定節點 ChatOpenAI
    此教學中 Language Model 選的是 OpenAI ，而 ChatOpenAI 作為接口與指定的 LLM 進行溝通，傳送來自 Chain 的文字並回應自 LLM 生成的內容，需要設定以下
    - Connect Credential: 選擇剛剛設定好的 Credential 即可
    - Model Name: OpenAI 有不同的模型資源可以選擇，請依據應用場景的需求做選擇，此教學中，則是選擇經濟實惠的 gpt-4o-mini(latest)
    - Temperature[^docs-openai]: 此參數是調整 LLM 的隨機性(創造力)，數值上沒有絕對『沒有最好的參數，只有最適合的參數』此教學中，則是維持預設

9. 設定節點 Chat Prompt Template
    Chat Prompt Template 是與 LLM 互動的模板，供開發者設定 Prompt 以定義該 Chain 的功能，而 Prompt 可以透過變數來進行調整
    - System Message: `Please respond only in the {output_language} language. Do not explain what you are doing. Do not self reference. You are an expert translator. Translate {input_language} into {output_language} using native-level vocabulary and expressions.`
    - Human Message: `{input}`
    - Chat Prompt Template 的下方有一個 Format Prompt Values 按鈕，點選它以進行變數的設定

        ![Desktop View](/assets/img/2024-09-25-implementing_an_ai_bot_using_flowiseai/011.png){: .normal }
        
    - 因為我們在 System Message 與 Human Message 的內容中使用大括號設定了變數，所以在 Format Prompt Values 內會有 input_language, output_language, input 三個變數

    > 在 Format Prompt Values 先新增變數後，再設定 System Message/Human Message 的內容也是可以的
    {: .prompt-tip }

    ![Desktop View](/assets/img/2024-09-25-implementing_an_ai_bot_using_flowiseai/012.png){: .normal }

    - 設定變數
        - input_language: `Chinese`
        - output_language: `English`
        - input: 點選 `update this value` 欄位時，會跳出其他可選的變數，包含 『question: 使用者的對話訊息』與 『chat_history: 歷史對話紀錄』，請直接選擇 question

            ![Desktop View](/assets/img/2024-09-25-implementing_an_ai_bot_using_flowiseai/013.png){: .normal }
            
        - 完成之後的畫面應該是這樣，設定 Value 後記得要點選綠色勾✅以完成設定

            ![Desktop View](/assets/img/2024-09-25-implementing_an_ai_bot_using_flowiseai/014.png){: .normal }
            
10. 大功告成，但先儲存
    做了那麼多，使用前先點右上角的儲存按鈕完成儲存吧
    

# 使用 Chatflow

完成了開發，接著來使用剛剛完成的中翻英 AI Bot 吧，

1. 最直接的用法是在介面右上方點選 Chat 按鈕，使用對話介面與 AI Bot 對話

    ![Desktop View](/assets/img/2024-09-25-implementing_an_ai_bot_using_flowiseai/015.png){: .normal }
    
2. 照先前設定的 Prompt ，這個 AI Bot 的功能是將所有中文無條件翻譯為英文，而且不會多做解釋
    對話過程也很成功，它不會回覆任何問題，而是如預期地將所有輸入的中文翻譯為英文

    ![Desktop View](/assets/img/2024-09-25-implementing_an_ai_bot_using_flowiseai/016.png){: .normal }
    

# Chatflow 嵌入至其他服務

- FlowiseAI 支援將 Chatflow 透過 Code/API 來嵌入到其他服務的功能，可以輕易的在網站上嵌入開發好的 AI Bot，這個部分就不額外教學了，有興趣請參考[官方文件](https://docs.flowiseai.com/using-flowise/embed)

    ![Desktop View](/assets/img/2024-09-25-implementing_an_ai_bot_using_flowiseai/017.png){: .normal }

<br><br><br>

現在掌握 FlowiseAI 的基礎功能了，想想有什麼待解決的需求，試著用它來解決吧!

如果還想學習更多，也推薦參考官方 [Use Cases](https://docs.flowiseai.com/use-cases) 頁面，跟著指南實作各種實用功能

祝各位玩得愉快，各位掰掰👋

---

# Reference
[^website-flowiseai]: [FlowiseAI - Low code LLM Apps builder](https://flowiseai.com/)
[^docs-openai]: [OpenAI. How should I set the temperature parameter?](https://platform.openai.com/docs/guides/text-generation/how-should-i-set-the-temperature-parameter)
