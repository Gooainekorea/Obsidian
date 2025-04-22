---
tags:
  - Obsidian
---

Ubuntu 24.04의 데스트탑환경에서 기본적으로 마운팅 가능하게되있는 구글 드라이브 경로로 연동시 폴더명이 이상한 숫자와 문자의 문자열로 출력됨
- ## 원인 분석
    ### 1. Google Drive의 가상 파일 시스템 특성   
    - **Google 드라이브의 FUSE 마운트**
        - Ubuntu에서 Google Drive를 연동시, 일반적으로 `rclone mount` 또는 GNOME Online Accounts(nautilus integration) 등을 사용.
        - Google Drive는 로컬 파일 시스템과 다르게, 내부적으로 각 파일과 폴더를 
         **고유한 ID(문자+숫자 조합)**로 관리.
    - **폴더명/경로 변환 문제**
        일부 마운트 방식(특히 GNOME Online Accounts, `gvfs` 등)은 Google Drive의 폴더를 실제 이름이 아닌, Google Drive의 **내부 ID(예: `1a2b3c4d5e...`)로 마운트 경로를 생성하는 경우가 있다.**	
    
    ### 2. Obsidian의 볼트 경로 인식 방식
    - Obsidian에서 볼트를 새로 만들 때, **실제 파일 시스템상의 경로**를 기록.
    - 만약 Google Drive가 폴더 이름 대신 **ID 기반 디렉토리**로 경로를 노출한다면, Obsidian은 그 경로(예: `/run/user/1000/gvfs/google-drive:host=.../1a2b3c4d5e.../볼트명`)를 그대로 기록합니다.
    - 이로 인해, Obsidian 볼트의 경로가 사람이 읽을 수 없는 **이상한 숫자와 문자의 조합**으로 보이게 됩니다.
    
    ### 3. Windows와 Ubuntu의 차이
    - **Windows용 Google Drive**는 로컬 드라이브(예: G:\My Drive)처럼 동작하며, 폴더명이 그대로 보임.
    - **Ubuntu의 Google Drive 연동**은 내부적으로 ID를 경로로 쓸 수 있으므로, 동일한 Google Drive라도 경로가 다르게 보일 수 있습니다.
    
    ## 결론
	구글 드라이브의 내부 ID노출로 옵시디언이 폴더 경로가 아니라 ID로 기록해서 볼트명이 이상하게 보였다.
    

|구분|경로 표시 방식|원인 요약|
|---|---|---|
|Windows|G:\My Drive\폴더명|폴더명이 그대로 노출|
|Ubuntu (rclone, nautilus)|/run/user/1000/gvfs/google-drive:host=.../1a2b3c4d5e...|Google Drive의 내부 ID가 경로에 노출됨|

- **Ubuntu에서 Google Drive 폴더가 “이상한 문자+숫자”로 보이는 것은, Google Drive가 내부적으로 폴더를 ID로 관리하고, 이를 경로에 노출하기 때문.**
- 이는 Obsidian이나 Ubuntu의 문제가 아니라, Google Drive 연동 방식의 특성.
    
## 해결 방법 및 권장 사항

> [!rclone mount 파일설정]
> [Unit] 
> Description=Mount Google Drive using rclone 
> Wants=network-online.target 
> After=network-online.target 
> 
> 
> [Service] 
> Type=simple 
> User=ain  
> Group=ain  
> ExecStart=/usr/bin/rclone mount gdrive: /home/ain/MyDrive \  
> --config=/home/ain/.config/rclone/rclone.conf \
> --vfs-cache-mode writes \ 
> --allow-other \  
> --log-file=/home/ain/rclone.log \ 
> --log-level=INFO  
> ExecStop=/bin/fusermount -uz /home/ain/MyDrive 
> Restart=always RestartSec=10 
> 
> [Install] 
> WantedBy=multi-user.target 

[!rclone mount 파일설정]
> [Unit] 
> Description=Mount Google Drive using rclone<span style="color:rgb(142, 182, 210)"> ## 서비스 설명 : <span style="color:rgb(0, 112, 192)">rclone을 사용하여 Google Drive를 마운트</span></span>
> Wants=network-online.target                            <span style="color:rgb(142, 182, 210)">## 이 서비스 실행시 `network-online.target`이 활성화되어 있기를 원함</span>
> After=network-online.target                              <span style="color:rgb(142, 182, 210)">## `network-online.target`이 활성화된 이후에 이 서비스가 실행되도록 순서를 지정</span>
> 
> 
> [Service] 
> Type=simple                                                        <span style="color:rgb(142, 182, 210)">## 서비스 타입 지정. ExecStart에서 지정한 명령이 곧바로 메인 프로세스가 됨/ 별도의 준비 신호 없음</span>
> User=ain                                                              <span style="color:rgb(142, 182, 210)">## 실행할 사용자 계정 이름</span>
> Group=ain                                                           <span style="color:rgb(142, 182, 210)">## 실행시 사용할 그룹</span>
> ExecStart=/usr/bin/rclone mount gdrive: /home/ain/MyDrive \  <span style="color:rgb(142, 182, 210)">## 서비스에서 실행할 명령어</span>
>                                       <span style="color:rgb(142, 182, 210)"> ##`rclone mount` 명령을 사용하여 `gdrive:`라는 rclone remote<span style="color:rgb(0, 112, 192)">(구글 드라이브)을 `/home/ain/MyDrive` 디렉토리에 마운트</span></span>
> --config=/home/ain/.config/rclone/rclone.conf \ <span style="color:rgb(142, 182, 210)"> ## rclone 설정 파일의 경로를 지정</span>
> --vfs-cache-mode writes \                                   <span style="color:rgb(142, 182, 210)">## VFS 캐시 모드를 "writes"로 설정하여, 파일 쓰기 작업을 캐시에 저장한 뒤 업로드 </span>
> --allow-other \                                                     <span style="color:rgb(142, 182, 210)">## 시스템의 다른 사용자도 마운트된 파일 시스템에 접근할 수 있도록 허용</span>
> --log-file=/home/ain/rclone.log \                       <span style="color:rgb(142, 182, 210)">## 로그 파일 경로를 지정</span>
> --log-level=INFO                                                 <span style="color:rgb(142, 182, 210)">## 로그 레벨을 INFO로 설정하여, 정보 수준의 로그를 기록</span>
> ExecStop=/bin/fusermount -uz /home/ain/MyDrive       <span style="color:rgb(142, 182, 210)">## 서비스가 중지될 때 실행할 명령어</span>
> 									  <span style="color:rgb(142, 182, 210)">## `fusermount -uz` 명령으로 마운트 해제를 수행. (`-u`는 언마운트, `-z`는 강제 언마운트 옵션)</span>
> Restart=always RestartSec=10                            <span style="color:rgb(142, 182, 210)">## 서비스가 비정상적으로 종료되었을 때 항상 재시작하도록 지정</span>
> 
> [Install] 
> WantedBy=multi-user.target                              <span style="color:rgb(142, 182, 210)">## 서비스가 종료된 후 재시작하기 전까지 10초간 대기</span>

>[!파일 적용]
>sudo systemctl daemon-reload
sudo systemctl enable rclone
sudo systemctl start rclone

>[!부팅후 로그확인] 
>- 에러 메시지가 있다면 mount 실패 원인을 직접적으로 알 수 있다.
>sudo systemctl status rclone sudo journalctl -u rclone

## 참고

- [rclone 공식 문서](https://rclone.org/drive/)
- [구글 드라이브의 내부 폴더 ID 구조](https://developers.google.com/drive/api/guides/folder)

