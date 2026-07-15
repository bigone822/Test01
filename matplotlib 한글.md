## OS 별 한글 설정

''' python
import matplotlib.pyplot as plt
import platform

# OS에 따라 한글 폰트 설정
if platform.system() == 'Windows':
    plt.rc('font', family='Malgun Gothic') # 맑은 고딕
elif platform.system() == 'Darwin': # macOS
    plt.rc('font', family='AppleGothic')   # 애플고딕
else: # Linux (Ubuntu 등)
    plt.rc('font', family='NanumGothic')   # 나눔고딕 (설치 필요)

# 마이너스 기호(-)가 깨지는 현상 방지
plt.rcParams['axes.unicode_minus'] = False
'''
