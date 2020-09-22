# LambdaCallGoogleSpeechAPI
Step by step: How to use Lambda to call Google Speech-to-text API<br>
## 概述:
    因最近專案需求，打算寫個使用AWS Lambda 呼叫Google API (Speech-to-text)來達成中文音檔轉成文字檔的project，整套流程為將音檔(.FLAC)
    上傳至S3 source bucket，接著S3 會設定Event 觸發lambda執行呼叫Google API的動作將人聲音檔存成文字檔（字幕）並且儲存在另一個S3 
    bucket-resized。
### 預先準備
    本次lab 所使用到的範本有兩個部分，一個是AWS 使用Lambda & S3 做成的縮圖範例，連結如<b>reference[1]</b>
    
    
    
    
    
    

