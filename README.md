# LambdaCallGoogleSpeechAPI
Step by step: How to use Lambda to call Google Speech-to-text API<br>
## 概述:
    因最近專案需求，打算寫個使用AWS Lambda 呼叫Google API (Speech-to-text)來達成中文音檔轉成文字檔的project，整套流程為將音檔(.FLAC)
    上傳至S3 source bucket，接著S3 會設定Event 觸發lambda執行呼叫Google API的動作將人聲音檔存成文字檔（字幕）並且儲存在另一個S3 
    bucket-resized。其中了解由file upload 至S3 觸發的event所發出的JSON格式以及Lambda如何去擷取這段JSON message是非常重要的。請務必
    多看幾眼。
### 預先準備 
本次lab 所使用到的範本有兩個部分，一個是AWS 使用Lambda & S3 做成的縮圖範例，連結如"reference[1] AWS image resize sample code"，另一
個部分是使用google speech-to-text API 範例，連結如 “reference[2] Speech-to-Text Client Libraries"，其他相關的環境設定請參考 
“reference[3] Setting up a Python development environment"

### 步驟一
Lamdba的虛擬環境準備我們會需要launch一台EC2(w/i Amazon Linux AMI)來準備，原因是lambda上的一些相依性在其他作業系統上可能會有相容性的問題，
例如我第一次是將lambda package佈建在我的MAC上，結果在出現以下error -->(ImportError: cannot import name '_imaging' from 'PIL'),
但這問題使用了AWS EC2 Amazon Linux image就獲得改善。
* launch EC2 with Amazon Linux image
* 進入EC2環境進行環境佈建
```
sudo yum update
sudo yum install python3 python3-dev python3-venv
```
```
wget https://bootstrap.pypa.io/get-pip.py
sudo python3 get-pip.py
```

    
reference[1]: https://docs.aws.amazon.com/zh_tw/lambda/latest/dg/with-s3-example-deployment-pkg.html
reference[2]: https://cloud.google.com/speech-to-text/docs/libraries#client-libraries-install-python
reference[3]: https://cloud.google.com/python/setup#linux
    
    
    
    
    
    

