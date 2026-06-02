# Bonsai Image Demo

<p align="center">
  <img src="./assets/bonsai-logo.svg" width="280" alt="Bonsai Image">
</p>

<p align="center">
  <a href="https://prismml.com"><b>Website</b></a> &nbsp;|&nbsp;
  <a href="https://github.com/PrismML-Eng/Bonsai-Image-Demo"><b>GitHub</b></a> &nbsp;|&nbsp;
  <a href="https://discord.gg/prismml"><b>Discord</b></a>
</p>

<p align="center">
  <b>HF Collections:</b>
  <a href="https://huggingface.co/collections/prism-ml/bonsai-image">Bonsai-Image</a>
  &nbsp;|&nbsp;
  <b>Whitepapers:</b>
  <a href="bonsai-image-4b-whitepaper.pdf">Bonsai-Image 4B</a>
</p>

<p align="center">
  <b>Other Demos:</b>
  <a href="https://huggingface.co/spaces/prism-ml/Bonsai-image-demo">HuggingFace Space</a> ·
  <a href="https://colab.research.google.com/github/PrismML-Eng/Bonsai-image-demo/blob/main/notebooks/bonsai_image_colab.ipynb">Google Colab</a>
</p>

---

Apple Silicon (macOS, [mflux](https://github.com/filipstrand/mflux) + MLX), NVIDIA GPU (Linux, [gemlite](https://github.com/dropbox/gemlite) + [HQQ](https://github.com/dropbox/hqq) 커널 `backend_gpu` 사용), 또는 NVIDIA GPU **Windows 기본 환경** (동일한 gemlite/HQQ 스택, [triton-windows](https://github.com/triton-lang/triton-windows) 사용, WSL2 불필요)에서 Bonsai로 이미지를 생성하세요.

## 빠른 시작 (Quick start)

**macOS / Linux:**

```bash
./setup.sh
./scripts/generate.sh --prompt "An icy Bonsai tree, in a rainy forest with a snowy mountains in the background, photo realistic."
```

**Windows (PowerShell):**

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned   # 1회성 설정; PowerShell은 기본적으로 .ps1 파일 실행을 차단합니다.
.\setup.ps1
.\scripts\generate.ps1 -p "An icy Bonsai tree, in a rainy forest with a snowy mountains in the background, photo realistic."
```

Windows에서 작동하지 않는 경우, [scripts/windows.md](scripts/windows.md)에서 사전 요구 사항 및 알려진 실패 원인(실행 정책, git 누락, 오래된 NVIDIA 드라이버, vcredist, 사용 중인 포트, 1024x1024에서의 OOM 등)에 대한 FAQ를 확인하세요.

## 모델 다운로드 (Download models)

```bash
./scripts/download_model.sh                # ternary (기본값), macOS에서는 mlx, Linux에서는 gemlite 선택
./scripts/download_model.sh ternary        # 위와 동일 — 명시적 지정
./scripts/download_model.sh binary         # binary 1-bit, 플랫폼 인식
./scripts/download_model.sh --model binary-gemlite   # 전체 형식, 백엔드 선택 재정의
```

Ternary (1.58-bit)는 권장되는 데모 변형으로, 약간의 크기 증가로 더 나은 품질을 제공합니다. Binary (1-bit)는 더 작고 가벼운 버전입니다.

## 구성 (Configuration)

| 변수 (Variable) | 값 (Values) | 기본값 (Default) |
|----------|--------|---------|
| `BONSAI_VARIANT` | `ternary` (1.58-bit) / `binary` (1-bit) | `ternary` |
| `BONSAI_PACKAGE_MIN_AGE_DAYS` | int | `7` |

`BONSAI_VARIANT`는 `setup.sh` (다운로드할 가중치 선택) 및 `serve.sh` (스튜디오에서 로드할 모델 선택) 모두에서 적용됩니다. 세션당 한 번 설정하세요: `BONSAI_VARIANT=binary ./setup.sh` 실행 후 `BONSAI_VARIANT=binary ./scripts/serve.sh` 실행.

기본적으로 `setup.sh` 및 `serve.sh`는 `uv sync --exclude-newer` 및 `npm install --before`를 통해 게시된 지 `BONSAI_PACKAGE_MIN_AGE_DAYS=7`일 미만인 Python 또는 npm 패키지 버전의 설치를 거부합니다. 이는 `setup.sh`를 실행할 때까지 발견되거나 수정되지 않은 최신 공급망 손상에 대한 방어책입니다. 비활성화하려면 `BONSAI_PACKAGE_MIN_AGE_DAYS=0`으로 설정하세요.

## 전체 스튜디오 실행 (백엔드 + 프론트엔드)

<p align="center">
  <img src="./assets/app_screenshot.png" width="720" alt="Bonsai Image Studio screenshot">
</p>

`scripts/serve.sh`는 prism-image-studio의 FastAPI 백엔드를 `:8000` (이 리포지토리의 `models/`를 가리킴)에서 열고, Next.js 프론트엔드를 `:3000`에서 엽니다 (둘 다 포그라운드에서 실행). Ctrl+C를 누르면 함께 종료됩니다.

```bash
# 기본값: ternary, 포트 8000/3000
./scripts/serve.sh                       

# 사용자 지정
BACKEND_PORT=8800 FRONTEND_PORT=3100 ./scripts/serve.sh
```

프론트엔드는 `vendor/image-studio`에 있습니다 (`setup.sh`에 의해 클론됨; `STUDIO_DIR=...`를 통해 경로 재정의 가능). 처음 실행 시 `frontend/node_modules`가 자동으로 설치됩니다.

## 이미지 생성 (CLI)

> **권장 방법**: `./scripts/serve.sh`를 한 번 실행하고 브라우저에서 스튜디오 UI를 사용하거나 터미널에서 `./scripts/send_request.sh -p "…"`를 사용하세요. 둘 다 실행 중인 백엔드와 통신합니다.
>
> `./scripts/generate.sh`는 실행 중인 serve.sh 없이 1회성 실행이 꼭 필요할 때를 위한 대안입니다. 실행할 때마다 콜드 스타트 비용(가져오기 + 모델 로드 + 첫 번째 형태 JIT)이 발생합니다.

### `send_request.sh` - 실행 중인 스튜디오와 통신하는 HTTP 클라이언트

`generate.sh`와 동일한 플래그를 사용하지만, 각 호출은 스튜디오의 `/generate`에 POST 요청을 보내므로 렌더링 간에 가중치와 커널이 웜(warm) 상태를 유지합니다:

```bash
./scripts/send_request.sh -p "An icy bonsai tree, in a rainy forest with a snowy mountain in the background, photo realistic." --size 1248x832 --seed 9909
BACKEND_PORT=8800 ./scripts/send_request.sh -p "..."     # 사용자 지정 서버
```

### `generate.sh` — 1회성, 서버 없음

`scripts/generate.py`를 감싸는 인프로세스 래퍼로, 로컬 `models/` 트리에 대해 [`prism-image-studio`](vendor/image-studio)의 `FluxPipeline`을 구동합니다:

```bash
./scripts/generate.sh -p "An icy bonsai tree, in a rainy forest with a snowy mountain in the background, photo realistic." --size 1248x832 --seed 9909 --output outputs/icy_bonsai.png --open
```

기본 크기는 512×512(빠른 미리보기)입니다. 크기는 32의 배수여야 합니다. 권장 크기는 다음과 같습니다:

| 비율 (Aspect) | 빠름 (~0.25MP) | 고품질 (~1MP) |
|-------------------|----------------|----------------|
| 정사각형 (1:1)    | 512×512        | 1024×1024      |
| 가로형 (3:2)      | 624×416        | 1248×832       |
| 세로형 (2:3)      | 416×624        | 832×1248       |
| 넓은 가로 (2:1)   | 704×352        | 1408×704       |
| 긴 세로 (1:2)     | 352×704        | 704×1408       |


## 설정 후 폴더 구조 (Folder structure after setup)

```
Bonsai-image-demo/
  .venv/
  vendor/                              # setup.sh/.ps1에 의해 클론된 비공개 종속성
    image-studio/                      # FastAPI 백엔드 + Next.js 프론트엔드
    mflux-prism/                       # prism 패치가 적용된 mflux 포크
  models/
    bonsai-image-4B-ternary-mlx/       # macOS 전용 (기본값)
    bonsai-image-4B-ternary-gemlite/   # Linux / Windows (기본값)
    bonsai-image-4B-binary-mlx/        # macOS 전용 (선택 사항, 더 작은 1-bit)
    bonsai-image-4B-binary-gemlite/    # Linux / Windows (선택 사항, 더 작은 1-bit)
  outputs/
  .serve-logs/                         # serve.sh / serve.ps1의 로그
  setup.sh                             # macOS + Linux 진입점
  setup.ps1                            # Windows 진입점 (setup.sh와 동일한 기능)
  scripts/
    common.sh                          # 공유 bash 헬퍼
    common.ps1                         # 공유 PowerShell 헬퍼
    download_model.sh / .ps1
    serve.sh        / .ps1               # 기본: 백엔드 + 프론트엔드 스튜디오
    send_request.sh / .ps1               # 실행 중인 serve에 대한 HTTP 클라이언트
    generate.sh     / .ps1               # 1회성 CLI (매 호출마다 콜드 스타트)
    generate.py                          # generate.{sh,ps1} 뒤의 Python 진입점
```
