# Python 스택 게이트

`SKILL.md` 1절의 네 칸을 Python 프로젝트에서 채우는 기본값.
프로젝트가 이걸 `CLAUDE.md` 에 네 줄로 옮겨 적는다.

## 기본값

```markdown
## 게이트
lint   uv run ruff check . && uv run ruff format --check .
test   uv run pytest -q
build  없음 — 라이브러리·스크립트라 산출물 없음
deploy 없음 — 배포 대상 없음
```

`uv` 가 없으면 `python -m` 으로 바꾼다 (`python -m ruff`, `python -m pytest`).

`python` 이 빈 버전을 반환하거나 venv 경로가 OS 마다 다르면(`Scripts/` vs `bin/`)
해석기를 찾는 한 줄짜리 실행기로 묶는다 — venv 활성화에 의존하지 않는다.

```bash
PY=.venv/Scripts/python.exe; [ -x "$PY" ] || PY=.venv/bin/python
```

## ruff 스캔 범위 — `.claude` 를 제외한다

ruff 는 `.gitignore` 를 읽어 `.venv/` 를 제외하지만 **`.claude/` 는 제외하지 않는다.**
정책 서브모듈의 마크다운까지 스캔 대상에 넣으므로, 남이 쓴 문서가 우리 lint 를
깨뜨릴 수 있다.

```toml
[tool.ruff]
extend-exclude = [".claude"]
```

실측: 추가 전 15 파일(정책 서브모듈 `.md` 14 + `CLAUDE.md`), 추가 후 1 파일.

`exclude` 가 아니라 `extend-exclude` 다 — `exclude` 로 쓰면 ruff 의 기본 제외
목록(`.venv` `.git` 등)이 사라진다.

## 도구 선택

| 자리 | 쓰는 것 | 왜 |
|---|---|---|
| lint + format | **ruff** | flake8·isort·black 을 한 도구가 대체한다. 설정 하나, 프로세스 하나 |
| test | **pytest** | 표준이고 `assert` 를 그대로 쓴다 |
| 패키지·venv | **uv** | 설치가 빠르고 `uv.lock` 이 재현을 보장한다. 없어도 `pip` + `requirements.txt` 로 동작한다 |
| 타입 | **안 넣는다** | mypy·pyright 는 코드가 자라고 타입 오류를 실제로 겪은 뒤에 넣는다 |

타입 체커를 처음부터 넣지 않는 이유: 설정과 무시 주석이 코드보다 먼저 쌓인다.
필요해지면 lint 자리에 한 줄 더하면 된다.

## 버전 고정 (`/task-policy` 4절)

| 파일 | 무엇 | 커밋 |
|---|---|---|
| `.python-version` | 인터프리터 버전 한 줄 | 한다 |
| `uv.lock` (또는 `requirements.txt`) | 의존성 정확한 버전 | 한다 |
| `pyproject.toml` | 의존성 범위·ruff·pytest 설정 | 한다 |

`.venv/` 는 커밋하지 않는다 — 머신에 묶인다 (`/task-policy` 4절 경로 규칙).

## build 가 "없음" 이 아닌 경우

| 산출물 | build 명령 |
|---|---|
| 배포 가능한 패키지 | `uv build` → `dist/` |
| 단일 실행 파일 | `pyinstaller` — 플랫폼마다 따로 만들어야 한다 |
| 컨테이너 | `docker build` — 태그는 `/version-policy` 의 버전과 맞춘다 |

## 흔한 함정

- **`pytest` 가 아무것도 못 찾고 exit 5 로 끝난다** — 테스트 0 개는 게이트 통과가
  아니다. `--strict-markers` 와 함께 테스트가 실제로 수집됐는지 개수를 본다
- **`ruff format --check` 를 빼먹는다** — `ruff check` 만 돌리면 포맷은 안 본다
- **Windows 줄바꿈** — `.gitattributes` 의 `eol=lf` 가 있어야 다른 PC 에서 diff 가 안 뜬다
- **`.claude` 를 제외하지 않는다** — 정책 서브모듈 문서가 lint 대상에 들어간다 (위 참조)
