# LambdaCallGoogleSpeechAPI
Step by step: How to use Lambda to call Google Speech-to-text API<br>
## 概述：<br>
    因最近專案需求，打算寫個使用AWS Lambda 呼叫Google API (Speech-to-text)來達成中文音檔轉成文字檔的project，整套流程為將音檔(.FLAC)
    上傳至S3 source bucket，接著S3 會設定Event 觸發lambda執行呼叫Google API的動作將人聲音檔存成文字檔（字幕）並且儲存在另一個S3 bucket-resized。
<b>步驟一</b>
    

