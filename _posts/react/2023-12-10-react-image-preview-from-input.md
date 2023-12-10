---
title: React input 에서 선택한 이미지 미리보기 컴포넌트 만들기
author: sunshine@ptokos.com
categories: [react, tailwindcss, typescript]
---

## 만들어볼 컴포넌트

![react-image-preview-from-input.png](/assets/img/react/react-image-preview-from-input-1.png)

## 기능
- Upload 버튼을 누르면 파일 선택 창이 뜬다.
- 파일을 선택하면 미리보기가 나온다.
- 미리보기 영역 **우측 상단**에는 X 버튼이 있다.

## 구현
### Upload 컴포넌트 구현
여기서는 input type="file" 을 사용하여 파일을 선택하는 것까지만 만든다.

![react-image-preview-from-input.png](/assets/img/react/react-image-preview-from-input-2.png)

#### 코드
```tsx
import React from "react";
import {PhotoIcon} from "@heroicons/react/24/solid";

const SelectImageInput = () => {
    return (
        <div className="sm:grid sm:grid-cols-2 sm:items-start sm:gap-4 sm:py-6">
            <div className="mt-2 sm:col-span-2 sm:mt-0">
                <div
                    className="flex max-w-3xl justify-center rounded-lg border border-dashed border-gray-900/25 px-6 py-10">
                    <div className="text-center">
                        <PhotoIcon className="mx-auto h-12 w-12 text-gray-300" aria-hidden="true"/>
                        <div className="mt-4 flex text-sm leading-6 text-gray-600">
                            <label
                                htmlFor="file-upload"
                                className="relative cursor-pointer rounded-md bg-white font-semibold text-indigo-600 focus-within:outline-none focus-within:ring-2 focus-within:ring-indigo-600 focus-within:ring-offset-2 hover:text-indigo-500"
                            >
                                <span>Upload a file</span>
                            </label>
                            <input id="file-upload" name="file-upload" type="file" className="sr-only"
                            />
                        </div>
                        <p className="text-xs leading-5 text-gray-600">PNG, JPG, GIF up to 10MB</p>
                    </div>
                </div>
            </div>
        </div>
    )
}

export default SelectImageInput
```


### 미리보기 컴포넌트 구현
미리보기의 경우 input[type='file'] 의 onChange 이벤트로 전달 받을 것이기 때문에 FileList 타입으로 전달 받는다.

#### 코드

```tsx
import React, {useEffect, useState} from "react";
import {XMarkIcon} from "@heroicons/react/24/solid";

interface Props {
    onCancel?: () => void;
    fileList?: FileList | null;
}

const ImagePreviewAndCancelable = ({fileList, onCancel}: Props) => {
    const [imageUrl, setImageUrl] = useState<string>('');

    useEffect(() => {
        if (!fileList) {
            setImageUrl('');
            return;
        }

        const reader = new FileReader();
        const file = fileList[0];

        reader.readAsDataURL(file);
        reader.onload = () => {
            if (!reader.result || !(typeof reader.result === 'string')) {
                return;
            }

            setImageUrl(reader.result);
        }
    }, [fileList]);

    if (!imageUrl) {
        return;
    }

    return (
        <div className="relative my-4 border-2 rounded p-2 border-dashed">
            <div>
                <img src={imageUrl}/>
                <div
                    className="absolute right-0 z-10 flex items-center justify-center rounded-full p-2 bg-gray-900 text-white hover:bg-gray-700 cursor-pointer"
                    onClick={() => {
                        onCancel?.();
                    }}
                    style={{
                        width: 30,
                        height: 30,
                        borderRadius: 50,
                        top: -15,
                        right: -10,
                    }}
                >
                    <XMarkIcon className="h-full w-full"/>
                </div>
            </div>
        </div>
    )

}

export default ImagePreviewAndCancelable;
```

### 최종 코드
ImagePreviewAndCancelable 컴포넌트를 사용하며 이벤트 연결을 해주면 된다.

`inputRef.current.value = ''` 코드가 왜 존재하는지 궁금하다면 [React input 에 같은 파일 업로드 시 onChange 가 발생하지 않는 문제 해결](/react-input-same-file/) 글을 참고하면 된다.  

```tsx
import React, {useState} from "react";
import {PhotoIcon} from "@heroicons/react/24/solid";
import ImagePreviewAndCancelable from "@/components/ImagePreviewAndCancelable";

const SelectImageInput = () => {
    const [fileList, setFileList] = useState<FileList | null>();
    const inputRef = React.useRef<HTMLInputElement>(null);

    const onFileChange = (e: React.ChangeEvent<HTMLInputElement>) => {
        if (!e.target.files) {
            return;
        }

        setFileList(e.target.files);
    }

    return (
        <div className="sm:grid sm:grid-cols-2 sm:items-start sm:gap-4 sm:py-6">
            <div className="mt-2 sm:col-span-2 sm:mt-0">
                <ImagePreviewAndCancelable fileList={fileList} onCancel={() => {
                    setFileList(null)
                    if (inputRef.current?.files) {
                        inputRef.current.value = '';
                    }
                }}/>

                <div
                    className="flex max-w-3xl justify-center rounded-lg border border-dashed border-gray-900/25 px-6 py-10">
                    <div className="text-center">
                        <PhotoIcon className="mx-auto h-12 w-12 text-gray-300" aria-hidden="true"/>
                        <div className="mt-4 flex text-sm leading-6 text-gray-600">
                            <label
                                htmlFor="file-upload"
                                className="relative cursor-pointer rounded-md bg-white font-semibold text-indigo-600 focus-within:outline-none focus-within:ring-2 focus-within:ring-indigo-600 focus-within:ring-offset-2 hover:text-indigo-500"
                            >
                                <span>Upload a file</span>
                            </label>
                            <input id="file-upload" name="file-upload" type="file" className="sr-only"
                                   onChange={onFileChange}
                                   ref={inputRef}
                            />
                        </div>
                        <p className="text-xs leading-5 text-gray-600">PNG, JPG, GIF up to 10MB</p>
                    </div>
                </div>
            </div>
        </div>
    )
}

export default SelectImageInput
```

참고 사이트
- Tailwind UI [Form Layouts](https://tailwindui.com/components/application-ui/forms/form-layouts)
