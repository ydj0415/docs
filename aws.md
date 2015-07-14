## AWS
![AWS](https://lh3.googleusercontent.com/upQblh3bz-Num-jochnMcnC9OZ8HQi3ivw_wXtit-fo=w216-h288-no)

## AWS basic
* Region은 도쿄
* VPC 안에 생성 
* 서버를 생성할 때 **private key**를 등록해줘야 한다.
* **키는 다운로드를 할 때 주는데 잃어버리면 노답**
* 네트워크는 10.0.1.100은 private ip
* DHCP를 기본으로 하여 public ip를 사용할 수 있고 서버를 재부팅할 때마다 바뀐다.
* 고정 아이피는 따로 신청해야함
* 방화벽은 각서버마다 따로 할당

## Network Rule
* AWS의 서비스가 위치하고 있는 물리적 장소
* 도쿄를 제외한 모든 region 선택가능

## VPC 선택
![vpc](https://lh3.googleusercontent.com/F5kxbC1TxQTH1mRwlbWklyoEPCtIBuMkFeJkth2R7vM=w20-h20-no) Virtual Private Cloud

## Subnet
* 하나의 VPC 내에 생성 할 수 있는 IP 주소 범위 개체
```
rp-cloud-awsdemo-testsubnet-djyu
```

## AMI Selection
* 아마존 머신을 선택할 수 있다.
* AMI는 EC2 인스턴스를 생성하는데 사용되는 Template
* AMI는 종류에 따라 추가적으로 사용 금액이 발생할 수 있다.

## EC2 생성
* Cpu, Memory, Storage 및 Network 성능 한계를 결정
* 인스턴스 타입에 따라 사용 금액이 결정
* 필요한 만틈의 서능을 가진 인스턴스 타입을 선택

## 인스턴스 Detail
* IP 및 서브넷 등을 설정할 수 있다.

## Storage 추가
* root 볼륨과 추가 볼륨을 선택
* 필요한 사이즈 지정
* 타입은 General Perpose(GPSSD) 사용 권장

## Tag 인스턴스
* 인스턴스 속성 부여
* rp-cloud-awsdemo-svr01-djyu
* 사용 목적
* 사용 만료 날짜
* 사용 시간 명시

## Security Group
* 인스턴스에 대한 트래픽을 제어하는 가상 방화벽

## Key Pair
* 키를 기존에 있는것을 사용할 지 새로 만들지 정할 수 있다.
```
EBS 볼륨은 자동으로 Naming 되지 않으므로 EBS 콘솔에서 지정 필요
```

## EC2 관리
* Terminate를 사용하면 서비스 전체를 삭제할 수 있다. (시간은 5-10분 소요)

## URL
[아마존 AWS 설명](http://www.pyrasis.com/private/2014/09/30/publish-the-art-of-amazon-web-services-book)
