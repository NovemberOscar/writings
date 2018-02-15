# [TIPS 18기] 프로그래밍 실습 첫번째

[강좌 배너]

## 1. Visual Studion 프로젝트 생성

앞으로 진행할 모든 Win32 프로그래밍은 Visual Studio에서 진행하게 된다.  
먼저 Windows Desktop 응용 프로그램 프로젝트를 생성하는것을 알아보자.  


 파일 > 새로 만들기 > 프로젝트를 클릭하면 다음과 같은 메뉴가 나오게 된다.  

 [그림 1]

 [그림 2]

윈도우즈 데스크톱 응용 프로그램 만들기를 선택하고 확인을 누르면 다음과 같은 화면이 나타날 것이다.

[그림 3]

## 2. Win32 프로그래밍의 기초

간략한 Desktop 응용 프로그램의 소스는 다음과 같다. 

```C++

#include "stdafx.h"
#include "MyFirstWin32.h"

LRESULT CALLBACK WndProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    if (uMsg == WM_DESTROY) PostQuitMessage(0);
    else if (uMsg == WM_CLOSE) {
        if (IDCANCEL == MessageBox(hWnd, L"마지막으로 아르토리아 최고인거 인정하고 가자", L"어 인정", MB_OKCANCEL));
        return 1;
    }
    return DefWindowProc(hWnd, uMsg, wParam, lParam);
}

int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow)
{
    WNDCLASS wc;

    wchar_t my_class_name[] = L"HelloWin32";
    wc.cbClsExtra = NULL;
    wc.cbWndExtra = NULL;
    wc.hbrBackground = (HBRUSH)COLOR_WINDOW;
    wc.hCursor = LoadCursor(NULL, IDC_ARROW);
    wc.hIcon = LoadIcon(NULL, IDI_APPLICATION);
    wc.hInstance = hInstance;
    wc.lpfnWndProc = WndProc;
    wc.lpszClassName = my_class_name;
    wc.lpszMenuName = NULL;
    wc.style = CS_HREDRAW | CS_VREDRAW;

    RegisterClass(&wc);

    HWND hWnd = CreateWindow(my_class_name, L"Artoria First",
        WS_OVERLAPPEDWINDOW, 100, 90, 400, 350, NULL, NULL, hInstance, NULL);
    ShowWindow(hWnd, nCmdShow);
    UpdateWindow(hWnd);

    MSG msg;
    while (GetMessage(&msg, NULL, 0, 0)) {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }

    return msg.wParam;
}

```

위의 소스는 크게 4가지의 부분으로 구성된다. 

> - 윈도우 클래스 등록  
> - 윈도우 생성
> - 메시지 루프
> - 메시지 처리기

### 2.1 윈도우 클래스 등록

```C++

WNDCLASS wc;

wchar_t my_class_name[] = L"HelloWin32";
wc.cbClsExtra = NULL;
wc.cbWndExtra = NULL;
wc.hbrBackground = (HBRUSH)COLOR_WINDOW;
wc.hCursor = LoadCursor(NULL, IDC_ARROW);
wc.hIcon = LoadIcon(NULL, IDI_APPLICATION);
wc.lpfnWndProc = WndProc;
wc.lpszClassName = my_class_name;
wc.lpszMenuName = NULL;
wc.style = CS_HREDRAW | CS_VREDRAW;

RegisterClass(&wc);

```
윈도우 클래스 등록은 운영체제에 이러한 프로그램을 만들겠다고 선언하는 것이며 클래스 등록은 다음과 같은 과정을 따른다.  
윈도우 클래스 구조체의 변수를 만들고 멤버를 설정하여 윈도우 클래스 등록을 위한 준비를 마치고 RegisterClass()를 사용하여 윈도우 클래스를 운영체제에 등록한다.

### 2.2 윈도우 생성
```C++

HWND hWnd = CreateWindow(my_class_name, L"Artoria First",WS_OVERLAPPEDWINDOW, 100, 90, 400, 350, NULL, NULL, hInstance, NULL);

ShowWindow(hWnd, nCmdShow);
UpdateWindow(hWnd);

```
핸들을 이용하여 윈도우를 만들고 운영체제에 윈도우 화면 갱신을 요구하는 ShowWindow와 UpdateWindow 함수를 사용해 운영체제가 가용 자원이 적을 때에도 윈도우가 나타날 수 있게 만든다. 

### 2.3 메시지 루프

```C++

MSG msg;

while (GetMessage(&msg, NULL, 0, 0)) {
    TranslateMessage(&msg);
    DispatchMessage(&msg);
}

```
읽어온 메시지를 저장할 구조체를 만든 후 반복문을 사용해 프로그램이 종료될 때까지(GetMessage가 0 반환) 메시지 큐에서 메시지를 읽어와 ASCII 값으로 해독하고(Translate)하고 처리(Dispatch)한다.

### 2.4 메시지 처리기

```C++

LRESULT CALLBACK WndProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    return DefWindowProc(hWnd, uMsg, wParam, lParam);
}

```
윈도우는 수많은 메시지를 발생 시키지만 대부분은 DefWindowProc을 사용해 예약된 작업을 수행하게 하고 여기에서는 윈도우에서 발생시키는 수많은 메시지중 필수적으로 처리할 메시지를 처리하면 된다. 

하지만 위의 코드는 단순한 원형일 뿐 실제로는 문제를 가지고 있다. 윈도우가 파괴된다는 메시지가 들어와도 아무 처리를 하지 않고 윈도우만이 파괴되어서 백그라운드 프로세스는 남아있기 때문이다.   
따라서 윈도우가 파괴된다는 WM_DESTROY 메시지를 처리해 메시지 큐에 WM_QUIT 메시지를 삽입해 프로그램이 정상 종료되도록 해야 한다. 

다음은 정상 종료가 가능하게 WM_DESTROY 메시지를 처리하는 예시이다. 

```C++

LRESULT CALLBACK WndProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    if (uMsg == WM_DESTROY) { PostQuitMessage(0); }
    return DefWindowProc(hWnd, uMsg, wParam, lParam);
}

```

다음은 WM_CLOSE 메시지를 받으면 종료 전에 확인 메시지를 띄우는 코드이다. 

```C++

LRESULT CALLBACK WndProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    if (uMsg == WM_DESTROY) { PostQuitMessage(0); }
    else if (uMsg == WM_CLOSE) {
        if (IDCANCEL == MessageBox(hWnd, L"마지막으로 아르토리아 최고인거 인정하고 가자", L"어 인정", MB_OKCANCEL));

        return 1;
    return DefWindowProc(hWnd, uMsg, wParam, lParam);
}

```
결과적으로 우리가 Windows 프로그래밍을 한다는 것은 메시지 처리기에서 메시지를 받아 이벤트를 처리하는 작업이다. 
