# BUG Clicker 원본 실행 안내

이 저장소에는 Windows, macOS, Linux 설치본에서 추출한 BUG Clicker 원본 앱과 에셋이 들어 있습니다. 운영체제별 `extracted/`는 앱 코드와 에셋이고, `resources/`에는 원본 `app.asar`와 해당 운영체제의 보조 파일이 있습니다. Electron 실행 환경은 포함되어 있지 않습니다.

| 운영체제 | 앱 버전 | 원본 설치본의 CPU |
| --- | --- | --- |
| Windows | 1.19.2 | x64 |
| macOS | 1.4.2 | Intel x64 및 Apple Silicon arm64 |
| Linux | 1.0.0 | x64 |

원본 앱과 같은 **Electron 44.2.0**을 사용합니다. Windows ARM64 및 Linux ARM64는 원본 설치본과 다른 구성으로 별도 검증이 필요합니다. macOS 원본의 최소 지원 버전은 13.0입니다.

## GitHub에 올리기

별도의 저장소를 새로 만들려면 이 `analysis` 폴더에서 아래 명령을 실행합니다. 주소는 본인 저장소 주소로 바꿉니다. 자동 줄바꿈 변환을 끈 뒤 처음부터 파일을 추가해야 원본 텍스트의 줄바꿈이 유지됩니다.

```sh
git init
git config core.autocrlf false
git add .
git commit -m "Add original BUG Clicker files"
git branch -M main
git remote add origin https://github.com/USER/REPOSITORY.git
git push -u origin main
```

이미 줄바꿈이 변환되어 커밋된 파일은 이 설정만으로 소급 복원되지 않습니다. 저장소 공개 전에 아래의 재배포 권한 안내를 확인하세요.

## 준비

- Git, Node.js **22.12 이상**과 npm이 필요합니다. 설치 및 복제에는 인터넷 연결이 필요합니다.
- 다음 복제 명령은 비공개 저장소에 대한 접근 권한이 있는 GitHub 계정으로 인증되어 있어야 합니다.
- 아래 명령은 새 작업 폴더에서 실행합니다. `analysis`와 `bug-clicker-electron`이 나란히 만들어지며, 원본 저장소 안에 Electron을 설치하지 않습니다.

```sh
git clone -c core.autocrlf=false https://github.com/Protocol314/BUG-Clicker.git analysis
npm install --prefix ./bug-clicker-electron --no-save electron@44.2.0
node ./bug-clicker-electron/node_modules/electron/install.js
```

Windows에서는 PowerShell, macOS와 Linux에서는 터미널에서 위 명령을 실행합니다. 이미 `analysis` 폴더가 있다면 `git clone`을 생략하고, 이 폴더의 **부모 폴더**에서 나머지 명령을 실행합니다. Electron 실행 파일 하나만이 아니라 설치된 런타임 폴더 전체가 필요합니다.

## Windows

PowerShell에서 위 준비 명령을 실행한 같은 폴더에서 원본 보조 파일을 Electron의 `resources` 폴더로 복사한 뒤 앱을 실행합니다.

```powershell
Copy-Item .\analysis\windows\resources\update.json .\bug-clicker-electron\node_modules\electron\dist\resources\
Copy-Item .\analysis\windows\resources\elevate.exe .\bug-clicker-electron\node_modules\electron\dist\resources\

& .\bug-clicker-electron\node_modules\electron\dist\electron.exe .\analysis\windows\extracted
```

## Linux

그래픽 데스크톱 세션과 배포판의 GTK, NSS/NSPR, X11 또는 Wayland, ALSA 등 Electron 시스템 라이브러리가 필요합니다. 누락된 라이브러리는 `ldd ./bug-clicker-electron/node_modules/electron/dist/electron | grep 'not found'`로 확인할 수 있습니다.

```sh
cp ./analysis/linux/resources/update.json ./bug-clicker-electron/node_modules/electron/dist/resources/

./bug-clicker-electron/node_modules/electron/dist/electron ./analysis/linux/extracted
```

## macOS

아래 명령은 원본 업데이트 구성 파일을 별도로 설치한 Electron 앱 번들에 복사하고 원본 앱을 실행합니다. 파일을 앱 번들에 추가하면 서명 상태가 달라질 수 있으며, Gatekeeper 및 실제 실행은 macOS에서 검증해야 합니다.

```sh
cp ./analysis/mac/resources/update.json ./bug-clicker-electron/node_modules/electron/dist/Electron.app/Contents/Resources/
cp ./analysis/mac/resources/app-update.yml ./bug-clicker-electron/node_modules/electron/dist/Electron.app/Contents/Resources/

./bug-clicker-electron/node_modules/electron/dist/Electron.app/Contents/MacOS/Electron ./analysis/mac/extracted
```

원본 타자 감지 기능을 사용하려면 Electron 앱을 종료하고 원본 보조 프로그램도 실행 파일 옆에 배치합니다. 이후 위 실행 명령을 다시 실행합니다.

```sh
cp ./analysis/mac/resources/bug-clicker-keys ./bug-clicker-electron/node_modules/electron/dist/Electron.app/Contents/MacOS/
chmod 755 ./bug-clicker-electron/node_modules/electron/dist/Electron.app/Contents/MacOS/bug-clicker-keys
```

이 보조 프로그램에도 앱 번들 서명 관련 제한이 적용될 수 있고, macOS 입력 모니터링 권한이 필요할 수 있습니다. 실제 Mac의 Gatekeeper·서명·권한 동작은 아직 검증하지 않았습니다.

## 실행 및 배포 시 참고

- 원본 앱은 사용자의 기존 BUG Clicker 설정이나 데이터 경로를 읽거나 수정할 수 있습니다. 실행 전에 필요한 데이터는 백업하세요.
- 원본 업데이트 및 공지 기능은 제작자의 GitHub 주소를 사용하며, 앱에는 제작자의 안내·문의 링크가 포함되어 있습니다. 원본 주소는 수정하지 않았습니다.
- 기존에 내려받은 원본 업데이트 패치를 적용하지 않고 추출된 버전으로 실행하려면 OS별 실행 명령 맨 끝에 `--no-patch`를 붙입니다. 이 옵션은 모든 네트워크 접속을 차단하지 않습니다.
- 개발용 Electron으로 실행할 때 설치본과 앱 서명, 아이콘, 파일 경로, 자동 업데이트 및 OS 통합이 다를 수 있습니다. Windows와 Linux의 기본 화면을 확인했지만, 모든 기능과 다른 PC 환경은 검증하지 않았습니다.
- 44.2.0은 원본 재현을 위한 버전이며 이후 Electron 보안 수정이 포함된다는 뜻은 아닙니다.
- 앱의 `package.json`에 표시된 라이선스만으로 캐릭터, 이미지, 음원 등 모든 에셋의 재배포 권한까지 확인되는 것은 아닙니다. 저장소 공개 전에 각 구성물의 권리와 라이선스를 확인하고 포함된 라이선스 고지를 보존하세요.
