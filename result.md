# viral-clips-crew 분석 보고서

## 1) 작업 범위
- 대상: `app.py`, `ytdl.py`, `local_transcribe.py`, `extracts.py`, `crew.py`, `clipper.py`, `subtitler.py`, `utils.py`, `reboot.py`, `pyproject.toml`, `README.md`
- 목적: 현재 코드베이스의 실행 흐름/구조를 파악하고, 실제 운영 가능한 실행 방향을 제시

## 2) 프로젝트 한 줄 요약
`viral-clips-crew`는 긴 영상(유튜브 URL 또는 로컬 mp4)을 입력받아, 전사 -> 클립 후보 추출 -> 구간 매칭 SRT 생성 -> 영상 트리밍 -> 자막 번인까지 자동화하는 파이프라인입니다.

## 3) 아키텍처 및 모듈 역할
| 모듈 | 역할 | 주요 입출력 |
|---|---|---|
| `app.py` | 전체 오케스트레이션(진입점) | 입력: 사용자 선택/영상 파일, 출력: `subtitler_output/*.mp4` |
| `ytdl.py` | 유튜브 다운로드 + 자막/텍스트 생성 | 입력: URL, 출력: `input_files/*.mp4`, `whisper_output/subtitles.srt`, `whisper_output/transcript.txt` |
| `local_transcribe.py` | Whisper 로컬 전사 | 입력: `input_files/*.mp4`, 출력: `whisper_output/*.srt`, `whisper_output/*.txt` |
| `extracts.py` | OpenAI로 바이럴 후보 텍스트 추출 | 입력: 전사 텍스트, 출력: `crew_output/api_response.json`, clips 텍스트 리스트 |
| `crew.py` | CrewAI/Gemini로 clips 텍스트를 SRT 구간에 매칭 | 입력: clips + 원본 SRT, 출력: `crew_output/new_file_return_subtitles_*.srt` |
| `clipper.py` | SRT 시간 기준 영상 트리밍(옵션: 정사각형 크롭) | 입력: mp4 + srt, 출력: `clipper_output/*_trimmed.mp4` |
| `subtitler.py` | SRT 시간 0초 기준 재정렬 + 번인 | 입력: trimmed mp4 + srt, 출력: `subtitler_output/*_subtitled.mp4` |
| `reboot.py` | 중간/출력 파일 정리 | 입력: 각 output 디렉터리, 출력: 파일 삭제(휴지통 이동) |

## 4) 실제 실행 흐름(E2E)
1. `app.py`에서 실행 모드 선택
   - 옵션 1: `ytdl.main(...)`로 유튜브 다운로드 및 transcript/srt 생성
   - 옵션 2: `local_whisper_process(...)`로 로컬 전사
2. `extracts.main()`이 전사 텍스트 기반으로 클립 후보 텍스트(현재 3개) 생성
3. `crew.main(extracts_data)`가 후보 텍스트를 SRT 구간과 매칭하여 `crew_output/*.srt` 생성
4. `clipper.main(...)`이 SRT 시간 범위로 영상 트리밍
5. `subtitler.process_video_and_subtitles(...)`가 자막을 영상에 번인

## 5) 확인된 상태
- 문법 수준 점검 결과(컴파일): 정상
  - 실행: `python -m compileall app.py clipper.py crew.py extracts.py local_transcribe.py subtitler.py utils.py ytdl.py reboot.py`
  - 결과: 전 파일 compile 성공

## 6) 핵심 리스크 및 개선 포인트

### [상] 배포/실행 진입점 불일치
- `pyproject.toml`의 스크립트가 `viral-clips-crew = "main:main"`으로 설정되어 있으나 `main.py` 파일이 없습니다.
- 영향: `poetry run viral-clips-crew` 실행 실패 가능성이 큼.
- 권장: `viral-clips-crew = "app:main"`으로 수정.

### [상] 영상-자막 매칭 로직이 N x M 조합으로 동작
- `app.py`에서 `input_files/*.mp4`와 `crew_output/*.srt`를 이중 루프로 모두 조합합니다.
- 영향: 영상과 무관한 SRT로 잘못 트리밍될 수 있음.
- 권장: 파일명 기반 1:1 매칭 규칙으로 변경.

### [상] `extracts.py` 프롬프트/검증 로직 불일치
- 프롬프트 상단은 4개 클립 지시, 하단 reminder는 정확히 3개 클립 지시.
- 코드 주석/로그도 "exactly four clips" 문구와 `len(...)=3` 검증이 섞여 있음.
- 영향: 모델 응답 일관성 저하, 후속 인덱싱/품질 저하 가능.
- 권장: 클립 개수 정책(3 또는 4) 단일화 및 프롬프트/검증/로그 동기화.

### [중] `crew.py` 입력 가정이 고정(3개 clips)
- `extracts[0]`, `extracts[1]`, `extracts[2]`를 직접 참조합니다.
- 영향: clips 개수 변동 시 즉시 예외(IndexError) 가능.
- 권장: clips 길이에 따라 Task를 동적으로 생성.

### [중] OS 의존 명령/문서 불일치
- `README.md`의 `.env` 생성 예시(`echo -e ...`)는 PowerShell에서 그대로 동작하지 않을 수 있음.
- 영향: 환경 변수 설정 실패로 초기 실행 중단.
- 권장: Windows PowerShell용/Unix용 명령을 분리 기재.

### [중] 자막 인코딩 변환 가정이 단일(iso-8859-1)
- `subtitler.py`가 무조건 `iso-8859-1`로 읽은 뒤 UTF-8로 저장.
- 영향: UTF-8 원본 자막에서 글자 깨짐 가능.
- 권장: UTF-8 우선 읽기 -> 실패 시 대체 인코딩 fallback 방식.

### [중] YouTube ID 검증 부족
- `ytdl.py`에서 URL 파싱 실패(`None`) 시 예외 처리 없이 transcript 조회 진행.
- 영향: 런타임 오류 발생.
- 권장: ID 미추출 시 즉시 사용자 에러 메시지 후 종료.

## 7) 실행 방향(권장 로드맵)

### 단계 A. "실행 안정화"(즉시)
1. `pyproject.toml` 스크립트 엔트리 수정 (`app:main`)
2. `.env` 설정 가이드 OS별 분리
3. 파일 매칭 로직을 1:1로 고정(영상명 기준)
4. clips 개수 정책을 3 또는 4로 확정 후 전체 동기화

### 단계 B. "품질 보강"(단기)
1. `crew.py` Task 동적 생성
2. `subtitler.py` 인코딩 fallback
3. `ytdl.py` URL/ID 유효성 검증 추가
4. 핵심 단계별 실패 메시지/리트라이 정책 추가

### 단계 C. "운영화"(중기)
1. 실행 옵션을 CLI 인자화(비대화형 실행 지원)
2. 샘플 입력 영상으로 E2E 스모크 테스트 스크립트 추가
3. 로그/결과물 메타데이터(처리시간, 실패원인, 클립 선택 근거) 저장

## 8) 현재 기준 실행 절차(권장)
아래는 "현 코드 구조" 기준 최소 실행 순서입니다.

1. API 키 준비 (`OPENAI_API_KEY`, `GEMINI_API_KEY`)
2. 입력 영상 준비 (`input_files`에 mp4 넣기) 또는 유튜브 URL 모드 선택
3. 실행: `poetry run python app.py`
4. 결과 확인: `subtitler_output`

## 9) 결론
- 이 프로젝트는 "아이디어 검증용 자동 편집 파이프라인"으로 구조는 명확합니다.
- 다만 실제 안정적 운영을 위해서는 **진입점 수정**, **1:1 파일 매칭**, **clips 개수 정책 정합성** 3가지를 우선 정리해야 합니다.
- 위 3가지만 우선 보완해도 실행 성공률과 결과 일관성이 크게 개선될 가능성이 높습니다.

