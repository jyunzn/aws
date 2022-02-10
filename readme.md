# S3

- [S3](https://docs.aws.amazon.com/zh_tw/AmazonS3/latest/userguide/Welcome.html) 的全稱: `Amazon Simple Storage Service ( S3 )`
- 就是一個拿來存數據的服務，主打非常穩定，不容易掛掉，非常適合流量大的場景
- 名詞解釋

    - `Bucket`

        - 就是一個裝 Object 的容器
        - 默認最多開一百個

    - `Object`

        - S3 中存儲的基本單位，基本上每傳一個檔案就有一個 object
        - Object 由 `Metadata` `Value` `Key` `Version ID` `Access control information` `Subresources` 組成
            - `Metadata`: 就是那些上傳日期，檔案類型之類的數據
            - `Value`:  就是檔案本身，S3 一個 object 最大 5T
            - `Key`: 就是檔名
                - S3 是一個扁平結構，不是樹狀的，所以檔名是包含前面的目錄，那些看起來像目錄的東西在這裡稱為 `前綴( prefix )`
                - 例如存在 `apple/` 下的 `1.txt` 的檔名是 `apple/1.txt`
            - `Version ID`:  S3 可以做版控，沒開的情況下所傳的檔案版本 id 的值都是 `null`

## 目錄

- [靜態託管懶人包](#靜態託管懶人包)
- [創建 Bucket](#創建-bucket)
- [上傳檔案](#上傳檔案)
- [安全](#安全)

    - [權限](#權限)
        - [Block public access](#block-public-access)
        - [ACL](#acl)
        - [Bucket 策略](#bucket-策略)

    - [加密 ( Default encryption )](#加密-default-encryption)
    - [log](#log)
- [生命週期](#生命週期)
- [CLI](#cli)


## 靜態託管懶人包

**[ STEP.1 ] 創建一個 bucket**

- 流程:

    1. 建立儲存貯體 ( Create bucket )
    2. 打名字 ( Bucket name )
    3. 建立儲存貯體 ( Create bucket )

- 注意事項:

    - 這名字寫了就改不了了
    - 靜態部署用的 bucket, 名字通常會用準備要部署的域名

- 大概是醬:

    ![create bucket](https://jyunzn.github.io/bf/images/wiki/s3/1.create-bucket.gif)

**[ STEP.2 ] 把檔案都傳到 bucket 裡面**

- 流程:

    1. 物件
    2. 拖曳上傳
    3. 等他跑完
    4. 關閉

- 大概是醬

    ![upload file](https://jyunzn.github.io/bf/images/wiki/s3/1.upload-file.gif)

    - 上傳之後，點擊文件就會看到有專屬 url 可以訪問惹
    - 但是默認情況下檔案是私有的，所以訪問會 forbidden



**[ STEP.3 ] 公開檔案權限**

- 流程:

    1. 關閉權限限制總開關

        1. 許可
        2. 封鎖公有存取權.編輯
        3. 把勾勾勾掉
        4. 儲存變更

    2. 寫儲存貯體政策規則

        - 規則參考

            ```json
            {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Sid": "PublicReadForGetBucketObjects",
                        "Effect": "Allow",
                        "Principal": "*",
                        "Action": "s3:GetObject",
                        "Resource": "arn:aws:s3:::example.com/*"
                    }
                ]
            }
            ```

        - 流程

            1. 許可
            2. 儲存貯體政策.編輯
            3. 參考規則貼上
            4. 把他給你的 arn 拿去將 `arn:aws:s3:::example.com/*`  換成  `<你的 arn>/*`

- 大概是醬:

    1. 關閉權限限制總開關
        ![block public access](https://jyunzn.github.io/bf/images/wiki/s3/1.block-public-access.gif)
    2. 寫儲存貯體政策規則
        ![bucket policy](https://jyunzn.github.io/bf/images/wiki/s3/1.bucket-policy.gif)



**[ STEP.4 ] 開啟靜態託管**

- 流程:

    1. 屬性
    2. 靜態網站託管
    3. 編輯
    4. 索引文件填一填
        - 那裡有兩個文件可以填: 網址根目錄重定向跟錯誤重定向的目標檔案
    5. 儲存

- 大概是醬

    ![static website hosting](https://jyunzn.github.io/bf/images/wiki/s3/1.static-website-hosting.gif)

**[ FINISH ]**

![finish](https://jyunzn.github.io/bf/images/wiki/s3/1.finish.gif)


## 創建 Bucket

- 名稱：

    - 他有一些規則，最重要就 `3 ~ 63 charset 的 小寫 數字 . - 組成` ，其他遇到你就會被紅字罵了

        ![blame me](https://jyunzn.github.io/bf/images/wiki/s3/2.blameme.png)

    - 這個名字是 `全球 S3 唯一的`

        - 就是說當有人用這名字時，你就不能用了
        - 刪掉一樣的名字後，amazon 還要處理時間才能用 ( 有時候要等超級久 )
        - 他文檔寫區域，不過應該是全球唯一的，看他生成的 url 就可以證明惹

            ![url](https://jyunzn.github.io/bf/images/wiki/s3/2.url.png)

    - 這名字創了就改不了了
    - 如果這個 bucket 是要做靜態部署用的，習慣上通常會直接用最終部署的 URL 來當名字

- 區域:

    ![region](https://jyunzn.github.io/bf/images/wiki/s3/2.region.gif)

    - 你這個 bucket 要存在哪個區域
    - 選擇區域要思考網路延遲，當地法律之類的
    - 區域選擇後，數據就會在這不會離開了，想移區域要自己去 clone


## 上傳檔案

- 如果是大文件，建議用 cli 或是 api 來傳，他會把文件拆成幾個小文件慢慢傳，如果傳送途中掛掉時，不用整個重傳

    ![upload](https://jyunzn.github.io/bf/images/wiki/s3/3.upload.png)



## 安全

### 權限

- 默認情況下 S3 的資源都是私有的
- 資源擁有者可以透過一些策略來授予其他人訪問權限
- 策略大致上分成兩類

    - 基於資源的策略，這個又分兩種
        - Bucket 策略
        - ACL ( 訪問控制列表, **Access Control List** )
            - 這是很早以前的東西，現在已經不推薦使用
    - 基於用戶的策略：通過身份和訪問管理 ( IAM ) 策略
        - IAM 是 aws 另外一個服務，用來做帳號權限管理的
        - 可以設置某個帳號只能訪問哪些 bucket 之類的
    - 這兩種互不衝突，可以同時設置資源可以被如何訪問以及誰可以訪問哪些 bucket

- 資源安全設置都可以在 `許可` 裡面編輯

#### Block public access

- 這東西有點像總開關

    ![block public access](https://jyunzn.github.io/bf/images/wiki/s3/4.block-public-access.png)

    - 不開放就改不了 Bucket
    - 即使之前 ACL 或 Bucket 已經開放，這裡禁用就沒效了

#### ACL

- 可以個別資源手動開放權限
- 這東西官方已經不太建議使用惹
- 流程

    1. 關閉禁用總開關
    2. 物件擁有權打開
        - 默認那個編輯按鈕會是不能點的，要先去上面那個物件擁有權的編輯把 ACL 啟用

            ![acl1](https://jyunzn.github.io/bf/images/wiki/s3/4.acl1.gif)

    3. `物件 > 你要公開的資源 > 動作 > 透過 ACL 公開 > 設為公有`


- 大概是醬:

    - 當前全部都沒開，將 `index.html` 設為公有

        ![acl set](https://jyunzn.github.io/bf/images/wiki/s3/4.acl-set-index.gif)

        - 成功之後就有文字惹
        - 只是 `index.html` 本來應該還有張圖片不見惹，因為那張圖片權限沒開

    - `1.png` 設為公有

        ![acl finish](https://jyunzn.github.io/bf/images/wiki/s3/4.acl-set-img.gif)

#### Bucket 策略

- 懶人包寫的那個規則就是這東西
- `arn`

    - Amazon Resource Name 的縮寫，是 amazon 資源的唯一標示符
    - 完整格式

        ```txt
        arn:partition:service:region:account-id:resource-id
        arn:partition:service:region:account-id:resource-type/resource-id
        arn:partition:service:region:account-id:resource-type:resource-id
        ```

    -  他會用 `:` 隔開每個參數

        - 參數根據不同類型的資源有些會沒有，但是 `:` 會留著 ( 這樣才有正確的參數 index )
        - 所以 S3 的 `arn:aws:s3:::example.com` 才有連續三個 `:`
            - `aws` 的 s3 服務下的 `example.com` bucket


- 規則說明

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "PublicReadForGetBucketObjects",
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::example.com/*"
            }
        ]
    }
    ```

    - `Effect`: 就兩種：同意 ( `Allow` ) 跟拒絕 ( `Deny` )
    - `Principal`: 主體, `*` 表示所有人
    - `Action` 描述你想幹嘛

        - `s3:GetObject` 獲取在 s3 get object 的權限

    - `Resource` 客體
    - 整個來說就是：同意所有人看 `arn:aws:s3:::example.com` 下的所有 object

- 除了直接寫 json 外，`action` 跟 `resource` 也能旁邊用點的，他會生成那些 node

    ![bucket click](https://jyunzn.github.io/bf/images/wiki/s3/4.bucket-click.gif)



### 加密 ( Default encryption )

- 流程：`屬性 > 預設加密`
- 在 s3 操作有兩種選擇

    - `SSE-S3`  什麼都不用幹，aws 生成一組來幫你加密

        ![s3 sse](https://jyunzn.github.io/bf/images/wiki/s3/4.s3-sse.gif)

    - `SSE-KMS`

        - `KMS ( Key Management Service )` 是 aws 另外一個服務，用來管理那些有的沒的 Key
        - kms 可以在 aws 手動生成密鑰，或者上傳自己生成的密鑰到 aws 保管

            ![kms key set](https://jyunzn.github.io/bf/images/wiki/s3/4.kms-key-set.gif)

            - 進階那邊就能選了
            - KMS 就是在 aws 手動生成密鑰來保管，下面那個就是自己傳自己的上去

        - kms 那邊弄好後，s3 這邊就有鑰匙可以選了

            ![s3 kms choose](https://jyunzn.github.io/bf/images/wiki/s3/4.s3-kms-choose.png)





### log

- 流程：`properties > Server access logging.edit > Enable > 寫要存儲的路徑 > save`

    ![log open](https://jyunzn.github.io/bf/images/wiki/s3/4.log-open.gif)

    - 只要訪問文件，他就會在指定路徑生成日誌

        ![log finish](https://jyunzn.github.io/bf/images/wiki/s3/4.log-finish.gif)





## 生命週期

- S3 的特點就是非常穩定，很適合大流量的場景，相對的成本也比較高

    - 影響 S3 定價因素

        - IO 請求數量
        - 文件大小
        - 數據傳輸量

    - 所以不經常訪問的文件就不適合放在 S3

        - 例如日誌文件、長期備份數據


- aws 提供了各種方案來適應各種需求

    - [storage classes](https://aws.amazon.com/tw/s3/storage-classes/)

        - 裡面有兩個詞 `durability`  跟  `availability`
            - `durability`  指的是數據會不會掛掉
                - aws 說一萬個 object 在一萬年只會丟一個
            - `availability`  指的是數據可以訪問的時間
                - aws 說一年頂多 7 - 10 個小時不能訪問

    - `S3`

        - `Standard`
        - `Standard_IA`

            - 存儲費用較低
            - 訪問費用較高
            - 適合不經常訪問的數據，`IA` 就是 `infrequent access` 的縮寫

        - `INTELLIGENT_TIERING`

            - 適合你不知道要把數據放哪裡的方案
            - 他會根據他的公式把經常訪問的放到 `Standard` ，不經常訪問的放到  `Standard_IA`
            - 但是會收取監控和自動化費用

        - `ONEZONE_IA`

            - 單區 IA 跟 IA 一樣，只是價格更便宜
            - 特點就是只保存在一個可用區，如果那個可用區出事，數據就有丟失風險

                - `Standard` 默認情況下會至少存到 3 個 ZONE

    - `S3 Glocier`

        - S3 的另一個分支服務
        - 特色是

            - 存儲很便宜，適合不經常訪問的數據
            - 單文件最大可以到 40T ( S3 只有 5T )
            - 文件默認就是加密的
            - 他存儲的數據是不能直接讀取的

                - 存入之後他會經過轉換後才存
                - 所以讀取時也要通過轉換回來後才能讀取
                - 這個轉換是要錢的

            - Glocier 的名稱不用全球唯一，他的名稱 scope 就是自己個 glocier

        - 缺點就是訪問的延遲時間很長

            - 因為需要轉換才能訪問
            - 不過付錢可以加速轉換

        - `Glocier` 各種分類特色都差不多，差別只在價格、訪問延遲之類的

            - `Instant Retrieval`
            - `Flexible Retrieval`
                - 轉換時間幾分鐘到幾小時
            - `Deep Archive`
                - **最便宜，但是讀取需要很久 ( 轉換時間幾個小時 )**
                - 適合不得不而需要長期存檔的數據


- 透過生命週期可以設定一些規則來自動移動數據

    - 大致上就分成「移動數據」跟「刪除數據」兩種操作
    - 操作概念就是當數據幾天後要怎麼處裡，例如 30 天後移到 `Glocier` ，一百天後刪掉數據之類的
    - 他還可以有篩選條件，符合哪些條件的 object 才會被操作
    - 流程：`管理 > 建立生命週期規則 > 寫個看得懂的名字 > 寫篩選條件或不篩選 > 勾出想要操作的對象 > 寫規則 > 建立規則`

        ![liftcycle](https://jyunzn.github.io/bf/images/wiki/s3/5.liftcycle.gif)


## CLI

**[ s3 ](https://docs.aws.amazon.com/cli/latest/reference/s3/index.html)**

- 就是一些簡單的 s3 增刪改查指令
- 指令本身蠻簡單的，就是 linux 那套，文檔寫的也蠻清楚的了，總之就是如果要操作 s3 的話，就加個 `s3://你的bucket/檔名` 就好了，所以這裡就簡單操作一下就好

    ```bash
    # ls: 看一堆有的沒的
    % aws s3 ls
    2022-01-12 17:05:58 test.backer-founder.com

    % aws s3 ls s3://test.backer-founder.com
    2022-01-12 17:21:40      82195 1.png
    2022-01-12 17:21:41        276 error.html
    2022-01-12 23:17:12        334 hello.html
    2022-01-12 17:21:42        322 index.html
    ```

    ```bash
    # cp
    % ls
    world.txt

    # 載 hello.html 到本地
    % aws s3 cp s3://test.backer-founder.com/hello.html  .
    download: s3://test.backer-founder.com/hello.html to ./hello.html

    % ls
    hello.html	world.txt

    # 傳 world.txt 上去
    % aws s3 cp world.txt s3://test.backer-founder.com/world.txt
    upload: ./world.txt to s3://test.backer-founder.com/world.txt

    % aws s3 ls s3://test.backer-founder.com/
    2022-01-12 17:21:40      82195 1.png
    2022-01-12 17:21:41        276 error.html
    2022-01-12 23:17:12        334 hello.html
    2022-01-12 17:21:42        322 index.html
    2022-01-13 11:01:35          6 world.txt  # <=
    ```

- 比較特別的指令
    - `presign` 他可以臨時的給訪問權限

        ```bash
        aws s3 presign s3://awsexamplebucket/test2.txt [--expires-in 秒數]?
        ```

        - 默認是一個小時
        - 他會生成一個 url，那個 url 就能臨時訪問惹
        - 大概是醬

            ![presign](https://jyunzn.github.io/bf/images/wiki/s3/6.presign.gif)

            - 本來 hello.html 不能訪問，不過可以拿去臨時生成一個 url 來訪問

    - `website` 用來設置靜態網站的

        ```bash
        website
        <S3Uri>
        [--index-document <value>]
        [--error-document <value>]
        ```

        ![website](https://jyunzn.github.io/bf/images/wiki/s3/6.website.gif)

- 補充：安裝 cli 與設置密鑰

    - [下載](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
    - 安裝

        ![cli install](https://jyunzn.github.io/bf/images/wiki/s3/6.cli-install.gif)

        ![cli install finish](https://jyunzn.github.io/bf/images/wiki/s3/6.cli-install-finish.png)

        - 第一次執行很慢是正常的，等他一下

    - 生成密鑰 ( 有了可以跳過，要操作 s3 的話，記得要開權限 )

        - 流程：`IAM > 使用者 > 新增使用者 > 想填的東西填一填 > 想開的權限記得給 > 建立 > 把金鑰存起來 ( 那個 csv 裡面就是惹 )`

            ![get key](https://jyunzn.github.io/bf/images/wiki/s3/6.get-secret-key.gif)

    - 設置金鑰

        ```bash
        aws configure
        ```

        ![set key](https://jyunzn.github.io/bf/images/wiki/s3/6.set-secret-key.gif)

        - 設置完就可以用惹

            ```bash
            % aws s3 ls
            2022-01-12 17:05:58 test.backer-founder.com
            ```


**[s3api](https://docs.aws.amazon.com/cli/latest/reference/s3api/index.html)**

- 提供各種比較細節的 api ，例如操作 bucket policy
- `bucket policy`

    - `put-bucket-policy` 寫 bucket-policy

        ```bash
        put-bucket-policy
        --bucket <value>
        [--content-md5 <value>]
        [--confirm-remove-self-bucket-access | --no-confirm-remove-self-bucket-access]
        --policy <value>
        [--expected-bucket-owner <value>]
        [--cli-input-json <value>]
        [--generate-cli-skeleton <value>]

        # --policy: 注意要給 file:// 協議
        ```

        - 大概是醬

            ```bash
            % cat bucket.policy.json
            {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Sid": "PublicReadForGetBucketObjects",
                        "Effect": "Allow",
                        "Principal": "*",
                        "Action": "s3:GetObject",
                        "Resource": "arn:aws:s3:::test.backer-founder.com/*"
                    }
                ]
            }

            % aws s3api put-bucket-policy --bucket test.backer-founder.com --policy file://bucket.policy.json
            ```

            ![put bucket policy](https://jyunzn.github.io/bf/images/wiki/s3/6.put-bucket-policy.gif)

    - `get-bucket-policy` 獲取當前 bucket 政策

        ```bash
        get-bucket-policy
        --bucket <value>
        [--expected-bucket-owner <value>]
        [--cli-input-json <value>]
        [--generate-cli-skeleton <value>]
        ```

        ```bash
        % aws s3api get-bucket-policy --bucket test.backer-founder.com
        {
            "Policy": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Sid\":\"PublicReadForGetBucketObjects\",\"Effect\":\"Allow\",\"Principal\":\"*\",\"Action\":\"s3:GetObject\",\"Resource\":\"arn:aws:s3:::test.backer-founder.com/*\"}]}"
        }
        ```
