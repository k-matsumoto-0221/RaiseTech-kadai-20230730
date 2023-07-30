## VPC
---
*構成要素*
- VPC
- サブネット
- ルートテーブル
- IGW

VPC
![VPC作成1](img/第4回目/Vpc.png)

* パブリックサブネット
![VPC作成1](img/第4回目/Subnet-Public.png)

  * ルートテーブル
![VPC作成1](img/第4回目/RT-Public.png)

* プライベートサブネット
![VPC作成1](img/第4回目/Subnet-Private.png)

  * ルートテーブル
![VPC作成1](img/第4回目/RT-Private.png)

* IGW
![VPC作成1](img/第4回目/Igw.png)  

## EC2
---
*構成要素*
  - EC2
  - セキュリティグループ

EC2
![VPC作成1](img/第4回目/Ec2.png)

* セキュリティグループ(インバウンド)
![VPC作成1](img/第4回目/Sg-Ec2-%E3%82%A4%E3%83%B3%E3%83%90%E3%82%A6%E3%83%B3%E3%83%89.png)

* セキュリティグループ(アウトバウンド)
![VPC作成1](img/第4回目/Sg-Ec2-%E3%82%A2%E3%82%A6%E3%83%88%E3%83%90%E3%82%A6%E3%83%B3%E3%83%89.png)

## RDS
---
*構成要素*
  - RDS
  - セキュリティグループ

RDS
![VPC作成1](img/第4回目/Rds.png)

* セキュリティグループ(インバウンド)
![VPC作成1](img/第4回目/Sg-Rds-%E3%82%A4%E3%83%B3%E3%83%90%E3%82%A6%E3%83%B3%E3%83%89.png)

* セキュリティグループ(アウトバウンド)  
  設定なし

## ローカルPCからEC2に接続
---
![VPC作成1](img/第4回目/Ec2%E3%81%AB%E6%8E%A5%E7%B6%9A.png)

## EC2からRDSに接続
---
![VPC作成1](img/第4回目/Ec2%E3%81%8B%E3%82%89Rds%E3%81%AB%E6%8E%A5%E7%B6%9A.png)
