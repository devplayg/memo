
## yt-dlp

- https://github.com/yt-dlp/yt-dlp
- https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp

```
docker run -it --rm -v /Users/won/volumns/python:/data python:3 bash
./yt-dlp -x --audio-format mp3 --add-metadata --embed-thumbnail https://sample.mp3
```

## rename

```python
#!/bin/zsh

# --- 설정 ---
# true로 바꾸기 전까지는 실제 파일명을 변경하지 않고, 어떻게 바뀔지만 보여줍니다.
DRY_RUN=false

# --- 스크립트 시작 ---
echo "🎵 MP3 파일명 정리를 시작합니다..."
if [ "$DRY_RUN" = true ]; then
    echo "⚠️ 미리보기 모드로 실행합니다. (실제 파일은 변경되지 않습니다)"
else
    echo "🔥 실제 파일명 변경 모드로 실행합니다."
fi
echo "--------------------------------------------------"

# 현재 디렉터리의 모든 .mp3 파일을 순회
for file in *.mp3; do
    # 원본 파일명 보존 (로그 출력용)
    original_filename="$file"
    
    echo "--- 📂 처리 중: $original_filename"

    # exiftool을 사용해 아티스트와 제목 정보 추출
    # -s3 옵션은 "Artist: " 같은 태그 이름 없이 순수한 값만 가져오게 함
    artist=$(exiftool -s3 -Artist "$file")
    title=$(exiftool -s3 -Title "$file")

    # 아티스트나 제목 정보가 파일에 없는 경우 건너뛰기
    if [[ -z "$artist" || -z "$title" ]]; then
        echo "  -> ❗️ 건너뛰기: 파일에 아티스트 또는 제목 태그가 없습니다."
        continue
    fi

    # 파일명으로 사용할 수 없거나 보기 싫은 문자 제거/변경 (예: 슬래시(/)를 언더스코어(_)로)
    sanitized_artist=$(echo "$artist" | tr '/' '_')
    sanitized_title=$(echo "$title" | tr '/' '_')

    # 새로운 파일명 생성: "아티스트 - 노래제목.mp3"
    new_filename="$sanitized_artist - $sanitized_title.mp3"

    # 현재 파일명과 새로운 파일명이 다를 경우에만 처리
    if [[ "$original_filename" != "$new_filename" ]]; then
        echo "  -> 🎤 아티스트: $artist"
        echo "  -> 🎶 제    목: $title"
        echo "  -> ✨ 변경될 이름: $new_filename"

        if [ "$DRY_RUN" = false ]; then
            # 파일명 충돌을 방지하는 -n 옵션과 함께 파일명 변경 실행
            mv -n "$file" "$new_filename"
            echo "  -> ✅ 파일명 변경 완료!"
        fi
    else
        echo "  -> 👍 이미 올바른 이름입니다. 건너뜁니다."
    fi
done

echo "--------------------------------------------------"
echo "✅ 모든 작업이 완료되었습니다."w
```

## download

```go
package main

import (
	"archive/zip"
	"fmt"
	"html/template"
	"io"
	"log"
	"net"
	"net/http"
	"os"
	"path/filepath"
	"strings"
)

// !!! 중요 !!! MP3 파일이 있는 실제 폴더 경로를 여기에 적어줘.
const musicDir = "/Users/won/volumns/python/mp3/" // 예시 경로, 실제 경로로 수정 필요

// 웹페이지를 위한 HTML 템플릿
const listTemplate = `
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MP3 다운로드 서버</title>
    <style>
        body { font-family: -apple-system, BlinkMacSystemFont, sans-serif; line-height: 1.6; padding: 20px; background-color: #f8f9fa; color: #343a40; }
        .container { max-width: 800px; margin: auto; background-color: #fff; padding: 20px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        h1, h2 { border-bottom: 2px solid #dee2e6; padding-bottom: 10px; }
        ul { list-style-type: none; padding: 0; }
        li { padding: 10px; border-bottom: 1px solid #eee; }
        li:last-child { border-bottom: none; }
        a { text-decoration: none; color: #007bff; }
        a:hover { text-decoration: underline; }
        .download-all a { display: inline-block; background-color: #28a745; color: white; padding: 15px 25px; border-radius: 5px; font-weight: bold; margin-top: 20px; }
    </style>
</head>
<body>
    <div class="container">
        <h1>🎵 내 MP3 목록</h1>
        <div class="download-all">
            <a href="/download-all">모두 다운로드 (ZIP)</a>
        </div>
        <h2>개별 파일</h2>
        <ul>
            {{range .Files}}
            <li><a href="/download?file={{.}}">{{.}}</a></li>
            {{end}}
        </ul>
    </div>
</body>
</html>
`

func main() {
	// 1. HTTP 핸들러 등록
	http.HandleFunc("/", listHandler)                 // 파일 목록 보여주기
	http.HandleFunc("/download", downloadHandler)     // 개별 파일 다운로드
	http.HandleFunc("/download-all", zipHandler)      // 전체 파일 ZIP으로 다운로드

	// 2. 서버 실행을 위한 IP 주소와 포트 설정
	port := "8080"
	ip, err := getLocalIP()
	if err != nil {
		log.Fatalf("로컬 IP 주소를 찾을 수 없습니다: %v", err)
	}

	serverURL := fmt.Sprintf("http://%s:%s", ip, port)
	log.Printf("✅ 서버가 시작되었습니다. 아래 주소로 접속하세요:")
	log.Printf(">>> %s <<<", serverURL)

	// 3. 웹서버 시작
	if err := http.ListenAndServe(":"+port, nil); err != nil {
		log.Fatalf("❌ 서버 시작 실패: %v", err)
	}
}

// 파일 목록을 보여주는 핸들러
func listHandler(w http.ResponseWriter, r *http.Request) {
	entries, err := os.ReadDir(musicDir)
	if err != nil {
		http.Error(w, "폴더를 읽을 수 없습니다.", http.StatusInternalServerError)
		return
	}

	var files []string
	for _, entry := range entries {
		if !entry.IsDir() && strings.HasSuffix(strings.ToLower(entry.Name()), ".mp3") {
			files = append(files, entry.Name())
		}
	}

	tmpl, err := template.New("list").Parse(listTemplate)
	if err != nil {
		http.Error(w, "템플릿 에러", http.StatusInternalServerError)
		return
	}
	
	data := struct {
		Files []string
	}{
		Files: files,
	}
	tmpl.Execute(w, data)
}

// 개별 파일을 다운로드하는 핸들러
func downloadHandler(w http.ResponseWriter, r *http.Request) {
	fileName := r.URL.Query().Get("file")
	if fileName == "" {
		http.Error(w, "파일이 지정되지 않았습니다.", http.StatusBadRequest)
		return
	}

	// 보안: 다른 디렉터리로 접근하는 것을 막음 (예: ../../etc/passwd)
	safeFileName := filepath.Base(fileName)
	filePath := filepath.Join(musicDir, safeFileName)

	// 파일이 존재하는지 다시 확인
	if _, err := os.Stat(filePath); os.IsNotExist(err) {
		http.NotFound(w, r)
		return
	}
	
	log.Printf("다운로드 요청: %s", safeFileName)
	w.Header().Set("Content-Disposition", "attachment; filename="+safeFileName)
	http.ServeFile(w, r, filePath)
}

// 모든 파일을 ZIP으로 묶어 다운로드하는 핸들러
func zipHandler(w http.ResponseWriter, r *http.Request) {
	log.Println("ZIP 파일 생성 및 다운로드 요청 시작...")
	
	w.Header().Set("Content-Type", "application/zip")
	w.Header().Set("Content-Disposition", "attachment; filename=\"all_music.zip\"")

	zipWriter := zip.NewWriter(w)
	defer zipWriter.Close()

	entries, err := os.ReadDir(musicDir)
	if err != nil {
		http.Error(w, "폴더를 읽을 수 없습니다.", http.StatusInternalServerError)
		return
	}

	for _, entry := range entries {
		if !entry.IsDir() && strings.HasSuffix(strings.ToLower(entry.Name()), ".mp3") {
			filePath := filepath.Join(musicDir, entry.Name())
			fileToZip, err := os.Open(filePath)
			if err != nil {
				log.Printf("파일 열기 실패 %s: %v", entry.Name(), err)
				continue
			}
			defer fileToZip.Close()

			writer, err := zipWriter.Create(entry.Name())
			if err != nil {
				log.Printf("ZIP에 파일 추가 실패 %s: %v", entry.Name(), err)
				continue
			}
			if _, err := io.Copy(writer, fileToZip); err != nil {
				log.Printf("파일 내용 복사 실패 %s: %v", entry.Name(), err)
				continue
			}
		}
	}
	log.Println("ZIP 파일 전송 완료.")
}


// 로컬 IP 주소를 찾아주는 헬퍼 함수
func getLocalIP() (string, error) {
	addrs, err := net.InterfaceAddrs()
	if err != nil {
		return "", err
	}
	for _, address := range addrs {
		if ipnet, ok := address.(*net.IPNet); ok && !ipnet.IP.IsLoopback() {
			if ipnet.IP.To4() != nil {
				return ipnet.IP.String(), nil
			}
		}
	}
	return "127.0.0.1", fmt.Errorf("유효한 로컬 IP를 찾을 수 없습니다")
}

```
