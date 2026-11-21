# 미팅메이트 (Meeting Mate)

> 비즈니스 미팅은 그대로, **통역·녹음·요약·표현 스터디**는 여기서.

구글 실시간 번역 기반의 미팅 코파일럿 PWA입니다. 상대의 영어와 내 한국어를 실시간 자막으로 바꾸고, 미팅을 자동 녹음해 종료와 동시에 요약·액션 아이템·비즈니스 표현을 뽑아 스터디 카드로 남깁니다.

- 앱: https://danielsonghoon-sketch.github.io/meeting-mate/
- 저장소: https://github.com/danielsonghoon-sketch/meeting-mate

## 기능

### 1. 실시간 자막 · 번역
- Web Speech API(`SpeechRecognition`, `continuous`, `interimResults`)로 실시간 음성 인식.
- **상대(English) / 나(한국어)** 버튼 1탭 전환(키보드 `Tab`)으로 양방향 통역. 언어 방향은 설정에서 변경.
- 번역 엔진: **Google Translate** 공개 엔드포인트(clients5 → translate.googleapis.com 순차 재시도, CORS/네트워크 실패 시 문장별 재시도 버튼) 기본. OpenAI 키가 있으면 **GPT 통역 모드** 선택 가능.
- 자막마다 ★(스터디 저장)·다시 듣기(TTS). 실패한 문장은 탭해서 재번역.

### 2. 자동 녹음
- 미팅 시작과 동시에 `MediaRecorder`로 녹음(기본 ON, 토글 가능), 일시정지/재개, 경과 타이머, 실시간 레벨 미터.
- 마이크 + (선택) **시스템 오디오**를 WebAudio로 믹스해 한 파일로 녹음. PC 온라인 미팅에서 상대 목소리까지 담을 수 있습니다.
- 종료 후 앱 안에서 바로 재생(커스텀 플레이어)·다운로드. 오디오는 **IndexedDB**에 저장해 localStorage 용량 초과를 피합니다.

### 3. 자동 요약 · 액션 아이템
- 종료 시 자동 요약: 한 줄 핵심 · 주요 안건 · 결정 사항 · Action Items(담당/기한) · 고급 비즈니스 표현 5~10개.
- **OpenAI 키가 없어도** 규칙/키워드 기반 요약 + 자막·요약 프롬프트 원탭 복사로 어디서나 사용 가능.
- OpenAI 키(`gpt-4o-mini`)가 있으면 문맥 기반 요약·표현 추출. 키는 이 기기 localStorage에만 저장되고 백업 JSON에는 포함되지 않습니다.

### 4. 표현 스터디
- 회의 중 ★ 즉시 저장, 요약 화면에서 표현별/전체 1탭 추가, 직접 추가.
- 출처 회의·날짜, 검색·필터(전체/미학습/학습 완료), **발음 듣기 1.0x / 0.8x**, **암기 카드**(뒤집기·발음·말해 보기 채점), 학습 완료 체크.
- JSON 백업/가져오기(설정에서도 가능).

## 사용 환경 · 제약 (솔직한 안내)

- **Chrome / Edge(Android·PC)** 권장. 음성 인식은 HTTPS에서만 동작하며 iOS Safari는 지원이 제한적입니다.
- 음성 인식(Web Speech API)은 **기본 마이크 입력만** 읽습니다. 줌·팀즈의 탭/시스템 오디오를 자막으로 직접 바꿀 수는 없습니다. 그래서 PC 온라인 미팅은 **스피커로 상대 목소리를 들려주고 마이크로 함께 수음**하는 방식을 안내합니다. 시스템 오디오 공유는 **녹음 파일**에 상대 음성을 더하는 용도입니다.
- 구글 번역 공개 엔드포인트는 비공식 경로라 네트워크·정책에 따라 일시적으로 실패할 수 있습니다. 실패 시 문장별 재시도와 안내를 제공합니다.
- 기록·표현·녹음은 서버로 전송되지 않고 이 브라우저(로컬 저장소·IndexedDB)에만 남습니다. 사이트를 지우면 함께 사라지므로 백업을 권장합니다.

## 구조

```
meeting-mate/
  build.py                 # 검증 + 단일 index.html/PWA 자산 빌드 (build/)
  src/
    index.template.html    # 셸(탑바·탭바·토스트)
    styles.css             # 다크 슬레이트 + 인디igo/에메랄드 테마
    app.js                 # 자막·번역·녹음·요약·스터디·복습 전체 로직
    sw.template.js         # 빌드 해시 캐시 서비스워커
  tools/make_icons.py      # PWA 아이콘(Pillow)
  test/e2e.py              # Playwright 실브라우저 E2E (로컬/라이브)
  test/smoke.py            # 빠른 로컬 스모크
  build/                   # GitHub Pages 배포 산출물 (git 저장소)
```

## 명령

```bash
python3 build.py
LD_LIBRARY_PATH=/workspace/tmp/libs/usr/lib/x86_64-linux-gnu python3 test/e2e.py
LD_LIBRARY_PATH=/workspace/tmp/libs/usr/lib/x86_64-linux-gnu python3 test/e2e.py https://danielsonghoon-sketch.github.io/meeting-mate/ --shots
```

## 배포

```bash
cd build && git add -A \
  && git -c user.name=danielsonghoon-sketch \
        -c user.email=228190639+danielsonghoon-sketch@users.noreply.github.com \
        commit -m "..." && git push origin main
```

- 저장 키: `meetingmate.settings.v1` · `meetingmate.meetings.v1` · `meetingmate.study.v1` · `meetingmate.draft.v1`, 녹음은 IndexedDB `meetingmate/audio`.
