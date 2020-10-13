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

### 步驟ㄧ 虛擬環境準備
Lamdba的虛擬環境準備我們會需要launch一台EC2(w/i Amazon Linux AMI)來準備，原因是lambda上的一些相依性在其他作業系統上可能會有相容性的問題，
例如我第一次是將lambda package佈建在我的MAC上，結果在出現以下error -->(ImportError: cannot import name '_imaging' from 'PIL'),
但這問題使用了AWS EC2 Amazon Linux image就獲得改善。
* launch EC2 with Amazon Linux image
* 進入EC2環境進行python & venv & PIP 基本元件安裝
```
sudo yum update -y
sudo yum install python3 python3-dev python3-venv
```
```
wget https://bootstrap.pypa.io/get-pip.py
sudo python3 get-pip.py
```
* 建立虛擬環境
```
cd /home/ec2-user
mkdir my-project
cd my-project
python3 -m venv venv
source venv/bin/activate
```
* 安裝Google相關packages
```
pip install google-cloud-storage
pip install --upgrade google-cloud-speech
```

* 安裝AWS相關packages
```
pip install Pillow boto3
```
### 步驟二 Google API service account(key)申請 以及撰寫lambda主程式
虛擬環境準備到一段落，接下來我們要申請Google Speech-to-text service account(key or credential)與開始撰寫lambda的主程式
* 申請Google Speech-to-text Service account
進入GCP帳號--> 建立一個project --> 進入speech-to-text console --> 選擇建立憑證 (角色記得選擇project的擁有者)
![](https://github.com/garliceric817/LambdaCallGoogleSpeechAPI/raw/master/Images/image1.png)   
* 選擇新建立的憑證-->產生金鑰JSON，並存於本機，這階段就算完成
![](https://github.com/garliceric817/LambdaCallGoogleSpeechAPI/raw/master/Images/Image2.png) 
* Lambda Function code ，建立一個檔案"lambda_function.py"
```
cd my-project/venv/lib/python3.7
vim lambda_function.py
```
* 貼上下面之程式碼並儲存，值得注意的是google的環境變數 "GOOGLE_APPLICATION_CREDENTIALS"＝ Google JSON金鑰的路徑，本機的話好解決，
但在lambda終會有點讓人不知所措，我是在後續直接將JSON金鑰放置在cd my-project/venv/lib/python3.7/site-packages/裡，結果也表示可以
抓的到
```
import boto3
import os
import sys
import uuid
from urllib.parse import unquote_plus
from PIL import Image
import PIL.Image

s3_client = boto3.client('s3')

os.environ["GOOGLE_APPLICATION_CREDENTIALS"]="my-project-final-1582980663902-a9536d561e3e.json"

def transcribe_file(speech_file, text_file):
    """Transcribe the given audio file asynchronously."""
    import io
    import os
    from google.cloud import speech
    from google.cloud.speech import enums
    from google.cloud.speech import types
    client = speech.SpeechClient()

    with io.open(speech_file, 'rb') as audio_file:
        content = audio_file.read()

    audio = types.RecognitionAudio(content=content)
    config = types.RecognitionConfig(
        encoding=enums.RecognitionConfig.AudioEncoding.FLAC,
        sample_rate_hertz=48000,
        language_code='cmn-Hant-TW')

    operation = client.long_running_recognize(config, audio)

    print('Waiting for operation to complete...')
    response = operation.result(timeout=90)

    # Each result is for a consecutive portion of the audio. Iterate through
    # them to get the transcripts for the entire audio file.
    for result in response.results:
        # The first alternative is the most likely one for this portion.
        print(u'Transcript: {}'.format(result.alternatives[0].transcript))
        print('Confidence: {}'.format(result.alternatives[0].confidence))
        with io.open(text_file, "w") as f:
            f.write(u'Transcript: {}'.format(result.alternatives[0].transcript))


def lambda_handler(event, context):
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = unquote_plus(record['s3']['object']['key'])
        tmpkey = key.replace('/', '')
        download_path = '/tmp/{}{}'.format(uuid.uuid4(), tmpkey)
        upload_path = '/tmp/resized-{}'.format(tmpkey)
        s3_client.download_file(bucket, key, download_path)
        transcribe_file(download_path, upload_path)
        s3_client.upload_file(upload_path, '{}-resized'.format(bucket), text.txt)
```
* 這邊要特別注意的是S3 Event Triiger 所發出的JSON檔案格式，以及範例碼萃取個items的方式
```
{
  "Records":[
    {
      "eventVersion":"2.0",
      "eventSource":"aws:s3",
      "awsRegion":"us-west-2",
      "eventTime":"1970-01-01T00:00:00.000Z",
      "eventName":"ObjectCreated:Put",
      "userIdentity":{
        "principalId":"AIDAJDPLRKLG7UEXAMPLE"
      },
      "requestParameters":{
        "sourceIPAddress":"127.0.0.1"
      },
      "responseElements":{
        "x-amz-request-id":"C3D13FE58DE4C810",
        "x-amz-id-2":"FMyUVURIY8/IgAtTv8xRjskZQpcIZ9KG4V5Wp6S7S/JRWeUWerMUE5JgHvANOjpD"
      },
      "s3":{
        "s3SchemaVersion":"1.0",
        "configurationId":"testConfigRule",
        "bucket":{
          "name":"sourcebucket",
          "ownerIdentity":{
            "principalId":"A3NL1KOZZKExample"
          },
          "arn":"arn:aws:s3:::sourcebucket"
        },
        "object":{
          "key":"HappyFace.jpg",
          "size":1024,
          "eTag":"d41d8cd98f00b204e9800998ecf8427e",
          "versionId":"096fKKXTRTtl3on89fVO.nfljtsv6qko"
        }
      }
    }
  ]
}

```
* 接下來就可以來打包環境跟lambda主程式，最後應該要看到三個檔案如下圖
```
cd $VIRTUAL_ENV/lib/python3.7/site-packages
zip -r9 ${OLDPWD}/function.zip .
cd ${OLDPWD}
zip -g function.zip lambda_function.py ##把lambda主程式加入環境壓縮檔中
deactivate ##退出虛擬環境
```
![](https://github.com/garliceric817/LambdaCallGoogleSpeechAPI/raw/master/Images/Image3.png) 

* 接下來我們要將 "function.zip"上傳至S3的bucket中 (隨便一個，只是要當Lambda的匯入源)，S3 CLI uplaod file 範例如下
```
aws s3 cp function.zip s3://my-bucket/
```
這邊EC2的權限設定要注意，我一開始使用EC2toS3FullAccess Role也無法成功upload，最後還是用埋access key到EC2才成功，要注意一下。

* 創建Lambda，進入Lamnbda console，選定function code source為S3 bucket，最後貼上function.py 在S3的URL就完成了
* 在lambda創建完畢後，在lambda console designer頁面加上S3 bucket trigger，設定資訊如下圖
![](https://github.com/garliceric817/LambdaCallGoogleSpeechAPI/raw/master/Images/Image4.png) 

* 最後再lambda這邊要注意的是allocate的memory and 執行時間，畢竟audio檔是要先存到memory中，若調整得不夠大，那lambda執行事會發生錯誤。
lambda的執行角色也要給對應的權限（非常重要，給不對連cloudWatch的log也不會出來）
* 最後就可以上傳範例的音檔至source-bucket中，並在source-bucket-resized bucket中得到名為text.txt
* 若結果沒有跑出，可以去Lambda console中的"Monitor", 中的“View Log in CloudWatch" 即可知道程式的是哪一部有問題。CloudWatch 在程式執
行成功以及失敗的畫面如下
![失敗](https://github.com/garliceric817/LambdaCallGoogleSpeechAPI/raw/master/Images/Image6.png)
![成功](https://github.com/garliceric817/LambdaCallGoogleSpeechAPI/raw/master/Images/Image7.png) 

### 結論
大概就這樣啦。不過實測後發現這套解法再將音訊檔轉成文字檔時效果並不好，且Google API對音源有很多限制如檔案大寫必須小於10MB（大約一分鐘的影片長度）
，檔案格式我寫死只讀flac，這些都是可以再修改的部分。

reference[1]: https://docs.aws.amazon.com/zh_tw/lambda/latest/dg/with-s3-example-deployment-pkg.html<br>
reference[2]: https://cloud.google.com/speech-to-text/docs/libraries#client-libraries-install-python<br>
reference[3]: https://cloud.google.com/python/setup#linux<br>
reference[4]類似範例:https://medium.com/@talktopoorni/invoking-google-cloud-platform-gcp-api-from-aws-lambda-417547cf3f3e
    
    
    
    
    
    

