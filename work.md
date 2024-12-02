# work

---
## 유플러스
* 사용자 이름, 전화번호쪽 코드 유플러스쪽 엑셀파일이 암호화되서 저장되는데 이번에 우리서버 복호화서버 연결해뒀다고는함 그래서 테스트한번 해봐야함
* stt쪽에서 화자들이 전화번호 말했을때 그것을 파이썬 쪽에서 마스킹해서주는데 그걸 자바단에서도 좀더 후처리해야하는코드 있음 이 형식 바뀐거 맞춰서 바꿔야함 `maskPersonalData`

---

## LG POC

poc 깃 주소 : http://15.165.113.182:8089/lg_cns/lg_poc.git

help 개발서버 주소 : http://121.125.63.254:8088/

ini안쪽에 lg가 poc쪽 설정파일 아닌게 헬프트라이얼

PW : 9RHDB0EuYvBVo
root PW : f5jHLWuaeid8t
Host 서초IDC(GPU_O)
 HostName 121.125.63.254
 port 31622
 User furence

위치 : /home/furence/meeting_minutes/web/

배포방법
2.agentec_shutdown.sh
기존 jar파일 뒤에 .bk년도월날짜0n붙여서 백업
내가 말은 jar파일 복붙. ls해서 복붙 확인후에./1.agentec_start.sh

---

## IPCR

svn 아이디 비번
worker
fcpass604**

ipcr3.5클론 : svn checkout --username worker --password fcpass604** https://121.170.218.142:8443/svn/Engine/java/src/IPCR3.5

---

## AICC

svn주소
https://121.170.218.142:8443/svn/Engine/java/src/LG_AICC_INTL/furence-aicc-admin
https://121.170.218.142:8443/svn/Engine/java/src/LG_AICC_INTL/furence-aicc-chat
