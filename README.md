# Linux_PAM 적용🎛

Linux에 PAM(Pluggable Authentication Modules)을 적용하여 비밀번호 입력 시 규제가 걸리게 끔 구성 요소를 설정해주는 작업을 진행해보고자 한다.

### **PAM 이란?**

Pluggable Authentication Modules의 약자로 인증 모듈을 뜻하며, 리눅스 환경에서 응용 프로그램에 대한 사용자의 사용 권한을 제어하는 모듈이다.

### **PAM을 사용하는 목적?**

권한을 부여하는 소프트웨어의 개발과 안전하고 적정한 인증의 개발을 분리하려는 데에 있다.

이에 앞써 test에 사용될 서버를 먼저 생성해 준다.

- 서버 생성

<div align="center">
  <img src="https://github.com/user-attachments/assets/555320e2-a394-4d9e-b215-9b34e327beea" width="80%" />
</div>

</br>

- 고정 IP 설정 (`00-installer-config.yaml 수정` )
```powershell
**sudo nano /etc/netplan/00-installer-config.yaml

=================================================**

network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      addresses:
        - 10.0.2.19/24  # 고정 IP 주소
      routes:
        - to: default
          via: 10.0.2.1  # 게이트웨이
      nameservers:
        addresses:
          - 8.8.8.8
      dhcp4: false
```

- 설정 적용

```powershell
sudo netplan apply
```

- 변경된 IP 확인

<div align="center">
  <img src="https://github.com/user-attachments/assets/a08bbaf6-9b3f-4722-8651-4547b0bedc05" width="70%" />
</div>


- 새롭게 추가된 IP 및 포트 포워딩 진행

<div align="center">
  <img src="https://github.com/user-attachments/assets/aab86247-f77f-4ca7-9bfb-4d8d8c900387" width="70%" />
</div>

### PAM 설정 🔗

그렇다면 비밀번호를 생성하거나 변경할 때 **암호를 8자리 이상으로 규제** 하기 위한 Configuration을 진행해보자.
</br></br>

- PAM 설정 파일 수정

`/etc/pam.d` 경로에서 PAM 라이브러리들을 확인할 수 있다(혹은 `/lib/security`). `common-password`를 편집해준다.

<div align="center">
  <img src="https://github.com/user-attachments/assets/ec028891-f127-462f-bea8-25ddd78b6235" width="70%" />
</div>

```bash
sudo nano /etc/pam.d/common-password
```
</br>
※ PAM 확인 
PAM은 리눅스 시스템에 기본적으로 설치되어 있는 모듈이며, `rpm -qi pam` 명령어로 확인 가능하다.

`pam_pwquality.so` 모듈에 `minlen=8 (암호의 최소 길이)` 규제를 추가해준다. 

<div align="center">
  <img src="https://github.com/user-attachments/assets/2f6da3cc-c2d8-42f1-8265-a566e5bc20ce" width="70%" />
</div>

`retry=3`의 경우 암호를 입력할 때 3번의 시도만 허용한다는 의미

- 규제 추가 후 **테스트** 결과

<div align="center">
  <img src="https://github.com/user-attachments/assets/293e930a-a340-4863-8532-07c3b822cdad" width="50%" /> &nbsp;&nbsp;&nbsp;
  <img src="https://github.com/user-attachments/assets/cbfdeebf-d856-41c3-b16e-d61f7efba179" width="30%" />
</div>
</br>

`ubuntu1`로 변경하였을때 조건을 충족하지 못해 <b>fail</b>이 뜨지만 `thseogus97`로 변경하였을 때는 조건 충족하여 <b>sucess</b>

</br></br>
**※ minlen 외에도 가능한 옵션**

```bash
password    requisite     pam_pwquality.so   retry=3 minlen=8
                                             dcredit=-1 # 숫자 최소 1개
                                             ucredit=-1 # 소문자 최소 1개
                                             lcredit=-1 # 대문자 최소 1개
                                             ocredit=-1 # 특수문자 최소 1개
```

이 과정을 통해 리눅스 시스템에서 비밀번호 정책을 강화하고, 사용자들이 8자리 이상의 비밀번호를 사용하도록 할 수 있다.
