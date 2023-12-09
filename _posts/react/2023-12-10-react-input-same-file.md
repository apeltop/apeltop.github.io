---
title: React input 에 같은 파일 업로드 시 onChange 가 발생하지 않는 문제 해결
author: sunshine@ptokos.com
categories: [react, tailwindcss, typescript]
---

## 문제 상황
input type="file" 일 때 같은 파일을 선택하면 onChange 가 발생하지 않는 문제가 발생하는 것을 확인하였다.

![react-input-same-file-1.gif](/assets/img/react/react-input-same-file-1.gif)

위에 gif 를 보면 파일을 선택하고 다시 같은 파일을 선택하면 onChange 로그가 찍히지 않는 것을 확인할 수 있다.

## 문제 해결
아래 gif 와 같이 같은 파일을 선택해도 로그가 찍히도록 해보자.

![react-input-same-file-2.gif](/assets/img/react/react-input-same-file-2.gif)

### 기존 코드

```typescript
import React, {useState} from "react";
import {PhotoIcon} from "@heroicons/react/24/solid";
import ImagePreviewAndCancelable from "@/components/ImagePreviewAndCancelable";

const DragAndDropInput = () => {
    const [fileList, setFileList] = useState<FileList | null>();
    const inputRef = React.useRef<HTMLInputElement>(null);

    const onFileChange = (e: React.ChangeEvent<HTMLInputElement>) => {
        console.log('onFileChange')
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
                            <p className="pl-1">or drag and drop</p>
                        </div>
                        <p className="text-xs leading-5 text-gray-600">PNG, JPG, GIF up to 10MB</p>
                    </div>
                </div>
            </div>
        </div>
    )
}

export default DragAndDropInput
```

혹자에게 전체 코드가 필요할 수 있어 첨부하였을뿐 전체 코드를 다 살펴볼 필요없다.

간단한 흐름은 아래와 같다.
1. input type="file" 을 클릭하면 파일 선택 창이 뜬다.
2. 파일을 선택하면 onChange 가 발생한다.
3. state 인 fileList 를 업데이트한다.
4. ImagePreviewAndCancelable 컴포넌트에서 이미지 **선택 취소 버튼**을 누르면 **fileList** 를 **null** 로 업데이트한다.

### 문제 파악
> 컴포넌트의 state 변경이 제대로 이루어지지 않아 re-rendering 이 되지 않은 것인지 찾아보니 근원적으로 input 의 **onChange 가 발생하지 않았다**.

처음에는 납득이 되지 않다가 `re-rendering 이 되지 않아 같은 파일은 선택했을 때 input 에서 바뀌었다고 알려줘야할 필요가 없어서 이런 문제가 발생했나?` 라는 생각이 들었다.

그래서 key 값을 계속해서 업데이트함으로 강제 re-rendering 을 시켜보았다.

```typescript
    ...
    const [key, setKey] = useState(0);

    const onFileChange = (e: React.ChangeEvent<HTMLInputElement>) => {
        console.log('onFileChange')
        setKey(key + 1);
        ...
    }
    
    return (
        ...
        <input id="file-upload" name="file-upload" type="file" className="sr-only" key={"file-upload" + key}
            onChange={onFileChange} />
        ...
    )

```

key 를 업데이트하면서 re-rendering 을 시켜보니 문제가 해결되었다.


### 해결
그러나 이 해결방법은 찝찝한 느낌으로 근본적인 문제를 해결하지 못한 것 같아 다시 찾아보았다.

해결 방법은 간단했다. input 의 value 의 값을 초기화해주면 된다.

값을 초기화해야하는 코드에 이 코드를 추가하여 초기화를 진행해주면 된다. 
```typescript
if (inputRef.current?.files) {
    inputRef.current.value = '';
}
```

### 해결 코드
```typescript
import React, {useState} from "react";
import {PhotoIcon} from "@heroicons/react/24/solid";
import ImagePreviewAndCancelable from "@/components/ImagePreviewAndCancelable";

const DragAndDropInput = () => {
    const [fileList, setFileList] = useState<FileList | null>();
    const inputRef = React.useRef<HTMLInputElement>(null);

    const onFileChange = (e: React.ChangeEvent<HTMLInputElement>) => {
        console.log('onFileChange')
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
                            <p className="pl-1">or drag and drop</p>
                        </div>
                        <p className="text-xs leading-5 text-gray-600">PNG, JPG, GIF up to 10MB</p>
                    </div>
                </div>
            </div>
        </div>
    )
}

export default DragAndDropInput

```


### 참고 사이트
stackoverflow [How to allow input type=file to select the same file in react component](https://stackoverflow.com/questions/39484895/how-to-allow-input-type-file-to-select-the-same-file-in-react-component) 
