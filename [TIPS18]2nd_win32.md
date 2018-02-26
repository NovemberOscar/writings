# [TIPS 18기] 프로그래밍 실습 두번째

[강좌 배너]

## 1. 비트맵이란?

비트맵은 컴퓨터 분야에서 디지털 이미지를 저장하는 데 쓰이는 이미지 파일 포맷 또는 메모리 저장 방식의 한 형태이다. 
우리는 32비트 방식을 사용하며 Unsigned INT를 사용하여 각 0~255 값으로 R, G, B에 1바이트씩 배분하며 나머지 1바이트는 알파값에 배분된다.

윈도우에서는 메모리 배열에 리틀 엔디언을 사용하기 때문에 B, G, R 순으로 배치된다.

## 2.GDI(Graphics Device Interface)

프로그래밍을 할 때 비트맵 패턴을 직접 사용할 수도 있겠지만 그렇다며 고려해야할 경우의 수가 너무 많아지기 때문에 윈도우에서는 가상 그래픽 카드 개념으로 중간에 GDI라는 것을 만들어 동일한 코드로 다양한 기기와 방식에 대응할 수 있게 해 준다.

그러나 오래된 기술이기 때문에 최신 그래픽 환경에 대응하지 못하고 알파 표현이 가능한 개선판인 GDI+도 근본적으로 해결할 수 없었다. 

## 3. GDI object

그리기 관련 요소를 오브젝트로 GDI를 한단계 더 추상화 한 것으로 Bitmap, Pen, Brush등이 있다. 

- Bitmap : 비트 패턴을 추상화
- Pen : 선 그리기 추상화
- Brush : 채우기 추상화

## 4. DC(Device context)
DC는 그림을 그리는데 사용 중인 GDI Object를 관리하기 위해 현재 사용되는 GDI Object의 핸들값을 저장하는 객체이다. 
따라서 우리가 그림을 그리려면 DC 객체의 핸들 값을 얻어야한다. 

먼저 그림을 그리기 위해서 관련된 정보를 얻기위해 GetDC함수를 사용하게 된다.
Get DC란 그리기 위해 DC를 만드는 함수로, 초기화 될때는 검은색 실선, 흰색 영역으로 채우기가 기본이다.

Get DC로 만들어진 DC를 제거하기 위해서는 Release DC를 사용한다. 

GDI object를 변경시키기 위해서는 SelectObect 함수를 사용하여 DC에 저장된 GDI Object 핸들 값을 변경해 Bitmap, Pen, Brush 등을 변경한다.

```C++

// 생략

LRESULT CALLBACK WndProc(HWND hWnd, UINT uMsg, WPARAM,  wParam, LPARAM lParam) {

    HDC h_dc =  GetDC(hWnd); 

    auto x = LOWORD(lParam);
    auto y = HIWORD(lParam);
    Rectangle(h_dc, 10, 10, 200, 200);

    ReleaseDC(hWnd, h_dc);

    return DefWindowProc(hWnd, uMsg, wParam, lParam);
}

// 생략

```
기본적인 DC 사용 방법이다. GetDC 함수로 현재 사용하는 DC의 핸들값을 받아오고 ReleasDC로 DC를 반환한다.

다음은 마우스 포인터를 클릭한 위치에 사각형을 만들어 내는 예제이다. 

```C++

// 생략

LRESULT CALLBACK WndProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    HDC h_dc = GetDC(hWnd); 

    int x = LOWORD(lParam);
    int y = LOWORD(lParam);
    
    Rectangle(h_dc , x-10, y-10, x+10, y+10);
    
    ReleaseDC(hWnd , h_dc);
    
    return DefWindowProc(hWnd, uMsg, wParam, lParam);
}

// 생략

```
다음은 SelectObject 함수를 사용하여 사각형에 파란색을 칠하는 예제이다. 

```C++

// 생략

LRESULT CALLBACK WndProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    HDC h_dc = GetDC(hWnd); 

    int x = LOWORD(lParam);
    int y = LOWORD(lParam);

    HBRUSH h_blue_brush = CreateSolidBrush(RGB(0, 0, 255));
    
    Rectangle(h_dc , x-10, y-10, x+10, y+10);
    SelectObject(h_dc, h_blue_brush);
    
    ReleaseDC(hWnd , h_dc);
    
    return DefWindowProc(hWnd, uMsg, wParam, lParam);
}

// 생략


