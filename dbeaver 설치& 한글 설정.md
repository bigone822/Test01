

## 1. 예전 등록 과정에서 꼬였을 수 있는 리스트 파일을 먼저 지웁니다.
sudo rm -f /etc/apt/sources.list.d/dbeaver.list

## 2. DBeaver 공식 GPG 키를 다시 안전하게 다운로드합니다.
sudo wget -q -O - https://dbeaver.io/debs/dbeaver.gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/dbeaver.gpg.key

## 3. 정확한 공식 APT 저장소 경로를 등록합니다.
echo "deb [signed-by=/usr/share/keyrings/dbeaver.gpg.key] https://dbeaver.io/debs/dbeaver-ce /" | sudo tee /etc/apt/sources.list.d/dbeaver.list

## 4. 패키지 목록을 갱신합니다. (이제 에러 없이 dbeaver를 찾아냅니다)
sudo apt-get update

## 5. DBeaver를 설치합니다.

'''
sudo apt-get install dbeaver-ce
'''

---
# 시스템 한글 폰트 설치 및 리프레시

## 1. 리눅스 표준 한글 폰트 패키지들을 설치합니다.
가장 무난하고 깔끔한 네이버 나눔폰트와 구글 노토(Noto) 한글 폰트 패키지입니다. 터미널에 아래 명령어를 복사해 넣으세요.

sudo apt update
sudo apt install -y fonts-nanum fonts-nanum-coding fonts-noto-cjk


## 2. 설치 후 시스템의 폰트 캐시를 완전히 갱신합니다.
새로 설치한 폰트를 시스템과 DBeaver가 즉시 인식할 수 있도록 폰트 인덱스를 강제로 재빌드합니다.

sudo fc-cache -f -v

## 확인 및 조치 방법

위 과정을 마치고 DBeaver를 완전히 종료했다가 다시 실행

이제 상단 메뉴 Window → Preferences → User Interface → Regional Settings로 이동했을 때, 사각형 박스 대신 깔끔하게 'Korean (한국어)' 혹은 '한국어'라고 제대로 텍스트가 보이실 겁니다.

만약 메뉴는 한글로 바뀌었는데 쿼리를 작성하는 편집창(SQL Editor)의 폰트가 여전히 깨져 보인다면 다음과 같이 폰트를 수동으로 지정해 주시면 됩니다.

Preferences (환경 설정) 창을 엽니다.

User Interface → Appearance → Colors and Fonts로 이동합니다.

우측 목록에서 DBeaver Fonts → Monospace font (또는 Main font)를 선택하고 Edit(편집) 버튼을 누릅니다.

방금 설치한 NanumGothicCoding 또는 NanumGothic, Noto Sans CJK KR을 선택하고 적용합니다.
