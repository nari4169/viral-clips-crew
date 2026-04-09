<a href="https://x.com/alxfazio" target="_blank">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="images/vcc-github-banner.png">
    <img alt="OpenAI Cookbook Logo" src="images/vcc-github-banner.png" width="400px" style="max-width: 100%; margin-bottom: 20px;">
  </picture>
</a>

Your [CrewAI](https://github.com/joaomdmoura/crewAI) Powered Video Editing Assistant

당신의 [CrewAI](https://github.com/joaomdmoura/crewAI) 기반 영상 편집 어시스턴트

Are you a social media content curator? Skip the tedious editing process and get polished video highlights in minutes. `viral-clips-crew` watches and listens to long-form content, extracting the most striking and potentially viral segments, ready for publication on social media.

소셜 미디어 콘텐츠 큐레이터이신가요? 지루한 편집 과정을 건너뛰고 몇 분 만에 다듬어진 하이라이트 영상을 받아보세요. `viral-clips-crew`는 롱폼 콘텐츠를 시청하고 분석하여, 소셜 미디어 게시에 바로 활용할 수 있는 인상적이고 바이럴 가능성이 높은 구간을 추출합니다.

> [!note]
> This project was 99% vibe coded as a fun Saturday hack. I'm not going to support it in any way, it's provided here as is for other people's inspiration and I don't intend to improve it. Code is ephemeral now and libraries are over, ask your LLM to change it in whatever way you like.
>
> 이 프로젝트는 토요일에 재미로 만든 해킹 프로젝트에 가깝고, 거의 감각적으로(99%) 작성되었습니다. 별도의 지원은 제공하지 않으며, 다른 사람들에게 영감을 주기 위해 있는 그대로 공개합니다. 지속적인 개선 계획은 없으니, 원하는 방식으로 LLM에게 수정 요청해서 사용하세요.

## Content Repurposing Made Easy

콘텐츠 재활용을 더 쉽게

<div align="center">
  <img src="https://github.com/alexfazio/viral-clips-crew/assets/34505954/c69da629-06eb-4279-a5cb-0d8d7fc1dfee" width="600px" height="auto">
</div>

`viral-clips-crew` helps you repackage your valuable content in new and engaging ways to capture attention on social media and drive traffic back to the original long-form piece. Whether you're looking to refresh your own content or recycle content from other creators, this tool streamlines the process, making content repurposing effortless and efficient.

`viral-clips-crew`는 소중한 콘텐츠를 새롭고 매력적인 방식으로 재가공하여 소셜 미디어에서 주목을 얻고, 원본 롱폼 콘텐츠로 트래픽을 다시 유도할 수 있도록 도와줍니다. 내 콘텐츠를 새롭게 활용하든, 다른 크리에이터의 콘텐츠를 재활용하든 이 도구는 과정을 단순화해 콘텐츠 재활용을 쉽고 효율적으로 만들어줍니다.

## Requirements

요구 사항

This project requires:

이 프로젝트를 실행하려면 다음이 필요합니다:

- Python 3.7+
  - Python 3.7 이상
- CrewAI
  - CrewAI
- OpenAI API key and Google Gemini API key
  - OpenAI API 키와 Google Gemini API 키

All required Python libraries are listed in `pyproject.toml`.

필요한 모든 Python 라이브러리는 `pyproject.toml`에 정리되어 있습니다.

## Installation

설치

1. Clone this repository to your local machine:

   이 저장소를 로컬 환경에 클론합니다:

    ```shell
    git clone https://github.com/alexfazio/viral-clips-crew.git
    ```

2. Install Poetry to automatically manage project dependencies:

   의존성 관리를 위해 Poetry를 설치합니다:

    ```shell
    pip install poetry
    ```

3. Install the required Python packages using Poetry:

   Poetry로 필요한 Python 패키지를 설치합니다:

    ```shell
    poetry install
    ```

4. Update Pydantic:

   Pydantic을 업데이트합니다:

    ```shell
    poetry update pydantic
    ```

5. Open `.env` and insert your OpenAI API key and Google Gemini API key.

   `.env` 파일을 열고 OpenAI API 키와 Google Gemini API 키를 입력합니다.

    ```shell
   echo -e "OPENAI_API_KEY=<your-api-key>\nGEMINI_API_KEY=<your-api-key>" > .env
    ```

## Usage

사용 방법

After setting up, drag your desired clip into the `input_files` directory.

설정을 마친 뒤 원하는 영상을 `input_files` 디렉터리에 넣어주세요.

**Gemini can process videos up to 1 hour in length. If you are using the OpenAI API, please ensure that the clip is less than 15 minutes in length. The current LLM context windows are approximately 15 minutes.**

**Gemini는 최대 1시간 길이의 영상을 처리할 수 있습니다. OpenAI API를 사용할 경우 영상 길이가 15분 미만인지 확인해 주세요. 현재 LLM의 컨텍스트 윈도우는 약 15분입니다.**

Run `viral-clips-crew` using Poetry with the following command:

아래 명령으로 Poetry 환경에서 `viral-clips-crew`를 실행합니다:

    ```shell
    poetry run python app.py
    ```

This will kickstart the process from beginning to completion.

이 명령을 실행하면 전체 파이프라인이 처음부터 끝까지 자동으로 진행됩니다.

Final output will be in the `subtitler_output` directory.

최종 결과물은 `subtitler_output` 디렉터리에 생성됩니다.

## Support

후원

If you like this project and want to support it, please consider leaving a star. Every contribution helps keep the project running. Thank you!

이 프로젝트가 마음에 들고 응원하고 싶다면 스타를 눌러주세요. 작은 기여도 프로젝트에 큰 도움이 됩니다. 감사합니다!

## Troubleshooting

문제 해결

If you encounter a `TypeError: 'NoneType' object is not iterable`, please check the following:

`TypeError: 'NoneType' object is not iterable` 오류가 발생하면 아래 항목을 확인해 주세요:
- Ensure your API keys are correctly set in the `.env` file.
  - `.env` 파일에 API 키가 정확히 설정되어 있는지 확인하세요.
- Verify that you have enough pay-as-you-go credits in your OpenAI account and Google Cloud account.
  - OpenAI 계정과 Google Cloud 계정에 사용량 기반 과금 크레딧이 충분한지 확인하세요.

## Note

참고

The code for `viral-clips-crew` is intended for demonstrative purposes and is not meant for production use. The API keys are hardcoded and need to be replaced with your own. Always ensure your keys are kept secure.

`viral-clips-crew` 코드는 데모 목적이며, 프로덕션 환경 사용을 전제로 하지 않습니다. API 키는 하드코딩되어 있을 수 있으므로 반드시 본인 키로 교체하고 안전하게 관리하세요.

## Credits

크레딧

Thank you to [Rip&Tear](https://x.com/Cyb3rCh1ck3n) for his ongoing assistance in improving this tool.

이 도구 개선에 꾸준히 도움을 준 [Rip&Tear](https://x.com/Cyb3rCh1ck3n)에게 감사드립니다.

## License

라이선스

[MIT](https://opensource.org/licenses/MIT)

Copyright (c) 2024-present, Alex Fazio

Copyright (c) 2024-present, Alex Fazio

---

[![Watch the video](https://i.imgur.com/TBD2bvj.png)](https://x.com/alxfazio/status/1791863931931078719)
