---
title: Shell Script로 파일 찾고 파일 복사하기 
author: sunshine@ptokos.com
categories: [ShellScript]
---

기존에는 파일을 다루어야 하는 상황이 발생하면 python, js, java 의 언어 중 하나로 개발을 하고는 했다.
그런데 [Ch13인 셀 가지고 놀기](/실용주의프로그래머-17/)가 떠올라 셀로 작업을 진행해보았다.

## 구현하고자하는 상황
특정 디렉토리 하위로 부터 이미지 파일 목록을 불러와 해당 파일의 이름에 L, M, S 가 붙는 이미지 파일이 추가되어야한다.

### 기존 디렉토리 구조
```
- foo
    - a
        - img1.jpg
    - b
        - img2.jpg
    - ima3.png   
```

### 기대하는 디렉토리 구조
```
- foo
    - a
        - img1.jpg
        - L_img1.jpg
        - M_img1.jpg
        - S_img1.jpg
    - b
        - img2.jpg
        - L_img2.jpg
        - M_img2.jpg
        - S_img2.jpg
    - ima3.png  
    - L_img3.jpg
    - M_img3.jpg
    - S_img3.jpg 
```

## Shell Script
```shell
#!/bin/bash

rename_ext() {
    local ext="$1"

    while IFS= read -r -d '' fullpath ; do
        dirname="$(dirname "$fullpath")"
        basename="$(basename "$fullpath")"

        echo ""
        echo fullpath: "$fullpath"
        echo dirname: "$dirname"
        echo basename: "$basename"
        echo ""

        cp "$fullpath" "$dirname/L_$basename"
        cp "$fullpath" "$dirname/M_$basename"
        cp "$fullpath" "$dirname/S_$basename"
    done < <(find ./ -iname "*.$ext" -type f -print0)
}

for ext in apng avif bmp gif jpeg jpg png tif webp ; do
    rename_ext "$ext" "IMG"
done

# OUTPUT
# fullpath: .//foo/IMG_3928.jpg
# dirname: .//foo
# basename: IMG_3928.jpg


# fullpath: .//foo/a/1541045981105.jpg
# dirname: .//foo/a
# basename: 1541045981105.jpg


# fullpath: .//foo/b/Asset 2@4x.png
# dirname: .//foo/b
# basename: Asset 2@4x.png


```

## 참고 문서
[https://unix.stackexchange.com/questions/718587/how-to-make-a-script-to-rename-images-and-videos-with-the-date-of-modification](https://unix.stackexchange.com/questions/718587/how-to-make-a-script-to-rename-images-and-videos-with-the-date-of-modification)
