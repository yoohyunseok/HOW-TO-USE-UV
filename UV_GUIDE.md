# uv 사용 가이드

uv는 Rust로 만든 파이썬 패키지·프로젝트 관리 도구예요. pip, venv, pyenv가 하던 일을 하나로 합쳤고, 설치 속도는 pip보다 10~100배 빨라요.

## 목차

- [설치](#설치)
- [두 가지 사용 방식](#두-가지-사용-방식)
- [pip 대체 방식: uv pip](#pip-대체-방식-uv-pip)
- [프로젝트 방식: uv init · uv add](#프로젝트-방식-uv-init--uv-add)
- [Python 버전 관리](#python-버전-관리)
- [도구 실행: uvx](#도구-실행-uvx)
- [기존 requirements.txt 프로젝트를 uv로 전환하기](#기존-requirementstxt-프로젝트를-uv로-전환하기)
- [pip ↔ uv 명령어 요약](#pip--uv-명령어-요약)
- [자주 묻는 질문](#자주-묻는-질문)

## 설치

컴퓨터에 한 번만 설치하면 돼요. 설치 후에는 터미널을 새로 열어야 `uv` 명령을 찾을 수 있어요.

**Windows (PowerShell)**

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

**macOS · Linux**

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**확인 및 업데이트**

```bash
uv --version
uv self update
```

이미 파이썬이 있다면 `pip install uv`로 설치해도 돼요. 이 경우 업데이트도 pip로 해야 해요.

## 두 가지 사용 방식

uv는 쓰는 방법이 두 가지예요. 기존 습관 그대로 빠르게 쓰고 싶으면 pip 대체 방식, 의존성 관리를 자동으로 맡기고 싶으면 프로젝트 방식을 고르면 돼요.

| 방식 | 명령 | 특징 |
| --- | --- | --- |
| pip 대체 방식 | `uv pip ...` | pip 명령 앞에 `uv`만 붙여요. requirements.txt를 그대로 쓰고, 가상환경 활성화도 지금처럼 직접 해요. |
| 프로젝트 방식 | `uv init` · `uv add` · `uv run` | pyproject.toml과 uv.lock으로 의존성을 관리해요. 패키지를 추가하면 기록, 설치, 버전 고정이 한 번에 처리돼요. |

## pip 대체 방식: uv pip

이미 활성화된 `.venv`가 있으면 uv가 자동으로 찾아서 그 안에 설치해요. 활성화하지 않아도 현재 폴더의 `.venv`를 찾아요.

```bash
# 가상환경 만들기 (python -m venv .venv 대신)
uv venv
.venv\Scripts\activate          # macOS/Linux: source .venv/bin/activate

# requirements.txt로 설치
uv pip install -r requirements.txt

# 패키지 추가 · 제거 · 확인
uv pip install pandas
uv pip uninstall pandas
uv pip freeze > requirements.txt
```

> [!WARNING]
> 이 방식에서는 패키지를 설치해도 requirements.txt가 자동으로 바뀌지 않아요. 설치 후 `uv pip freeze` 결과를 보고 직접 반영해야 해요.

## 프로젝트 방식: uv init · uv add

npm처럼 uv가 의존성 목록, 가상환경, 버전 고정을 모두 관리해요. 가상환경을 직접 활성화할 필요도 없어요.

### 새 프로젝트 시작

```bash
uv init my_app        # my_app 폴더를 만들고 프로젝트 파일 생성
cd my_app
uv add streamlit openai python-dotenv
uv run streamlit run app.py
```

`uv init`이 만드는 파일은 아래와 같아요. 이미 있는 폴더에서 실행하려면 폴더 이름 없이 `uv init`만 쓰면 돼요.

| 파일 | 역할 | Git 커밋 |
| --- | --- | --- |
| `pyproject.toml` | 프로젝트 정보와 의존성 목록. requirements.txt 역할 | O |
| `uv.lock` | 하위 패키지까지 정확한 버전을 전부 기록한 잠금 파일. 첫 `uv add`나 `uv sync` 때 생겨요 | O |
| `.python-version` | 이 프로젝트에서 쓸 Python 버전 | O |
| `.venv/` | 가상환경. uv가 자동으로 만들고 맞춰줘요 | X |
| `main.py` | 예시 코드. 필요 없으면 지워도 돼요 | 선택 |

### 패키지 관리

```bash
uv add requests                 # 최신 버전 추가
uv add "streamlit==1.66.1"       # 버전 지정
uv add --dev pytest              # 개발할 때만 쓰는 패키지
uv remove requests              # 제거 (기록에서도 삭제)
uv sync                         # uv.lock 그대로 환경 맞추기
uv lock --upgrade               # 허용 범위 안에서 최신 버전으로 갱신
uv tree                         # 의존성 트리 보기
```

### 실행은 uv run

`uv run`은 실행 전에 환경이 uv.lock과 맞는지 확인하고, 다르면 먼저 맞춘 다음 실행해요. 그래서 가상환경을 활성화하지 않아도 항상 올바른 환경에서 돌아가요.

```bash
uv run python main.py
uv run streamlit run app.py
uv run pytest
```

## Python 버전 관리

uv는 Python 자체도 설치해 줘요. 프로젝트마다 다른 버전을 써야 할 때 pyenv 없이 해결돼요.

```bash
uv python list              # 설치됐거나 설치 가능한 버전 보기
uv python install 3.12      # 특정 버전 설치
uv python pin 3.12          # 이 프로젝트를 3.12로 고정 (.python-version 수정)
uv venv --python 3.12       # 3.12로 가상환경 만들기
```

## 도구 실행: uvx

코드 포매터처럼 프로젝트 의존성은 아니지만 가끔 쓰는 도구는 `uvx`로 설치 없이 바로 실행해요. 자주 쓰는 도구는 `uv tool install`로 전역 설치할 수 있어요.

```bash
uvx ruff check .            # 임시 환경에서 ruff 실행
uv tool install ruff        # ruff를 어디서나 쓸 수 있게 설치
uv tool list
```

## 기존 requirements.txt 프로젝트를 uv로 전환하기

이 저장소(환율 챗봇)처럼 requirements.txt로 관리하던 프로젝트를 프로젝트 방식으로 옮기는 순서예요.

1. 프로젝트 폴더에서 pyproject.toml만 만들어요. `--bare`를 붙이면 main.py 같은 예시 파일을 만들지 않아요.

   ```bash
   uv init --bare
   ```

2. requirements.txt에 있는 패키지를 한 번에 가져와요. pyproject.toml과 uv.lock이 채워지고 `.venv`도 맞춰져요.

   ```bash
   uv add -r requirements.txt
   ```

3. 앱이 잘 실행되는지 확인해요.

   ```bash
   uv run streamlit run app.py
   ```

4. requirements.txt도 계속 필요하면 uv.lock에서 뽑아내요. 패키지를 바꿀 때마다 다시 실행하면 돼요.

   ```bash
   uv export --format requirements-txt --no-hashes -o requirements.txt
   ```

5. `pyproject.toml`, `uv.lock`, `.python-version`을 커밋해요. `.venv`는 .gitignore에 있는지 확인해요.

## pip ↔ uv 명령어 요약

| 하는 일 | pip 방식 | uv 프로젝트 방식 |
| --- | --- | --- |
| 가상환경 만들기 | `python -m venv .venv` | `uv venv` 또는 자동 |
| 가상환경 활성화 | `.venv\Scripts\activate` | 필요 없음 (`uv run`) |
| 패키지 추가 | `pip install X` + 파일 수정 | `uv add X` |
| 패키지 제거 | `pip uninstall X` + 파일 수정 | `uv remove X` |
| 다른 PC에서 똑같이 설치 | `pip install -r requirements.txt` | `uv sync` |
| 설치 목록 보기 | `pip freeze` | `uv tree` |
| 스크립트 실행 | `python app.py` | `uv run app.py` |
| Python 버전 바꾸기 | pyenv 등 별도 도구 | `uv python pin 3.12` |

## 자주 묻는 질문

<details>
<summary><b>uv.lock은 커밋해야 하나요?</b></summary>

네. uv.lock이 있어야 다른 사람이 `uv sync`로 완전히 같은 버전을 설치할 수 있어요. 직접 수정하지 말고 uv 명령으로만 바꿔요.

</details>

<details>
<summary><b>pyproject.toml과 uv.lock은 뭐가 달라요?</b></summary>

pyproject.toml은 "streamlit이 필요하다"처럼 직접 고른 패키지와 허용 범위를 적은 목록이에요. uv.lock은 그 결과로 실제 설치된 모든 패키지(하위 패키지 포함)의 정확한 버전이에요.

</details>

<details>
<summary><b>uv run 대신 가상환경을 활성화해서 써도 되나요?</b></summary>

네. `.venv\Scripts\activate` 후에 평소처럼 `streamlit run app.py`를 써도 돼요. 다만 패키지 추가는 `pip install` 말고 `uv add`로 해야 기록이 남아요.

</details>

<details>
<summary><b>VS Code가 패키지를 못 찾는다고 빨간 줄이 떠요.</b></summary>

VS Code의 Python 인터프리터를 프로젝트의 `.venv`로 바꿔요. `Ctrl+Shift+P` → "Python: Select Interpreter" → `.venv`를 고르면 돼요.

</details>

<details>
<summary><b>uv 캐시가 디스크를 많이 차지해요.</b></summary>

uv는 내려받은 패키지를 캐시에 모아 두고 재사용해서 빨라요. 정리하려면 `uv cache clean`을 실행해요.

</details>

---

공식 문서: [docs.astral.sh/uv](https://docs.astral.sh/uv/)
