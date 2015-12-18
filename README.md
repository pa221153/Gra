	#include <windows.h>
	
	#define WM_INIT		(WM_USER+1000)
	
	HWND Przycisk1;
	
	int x = 250, y = 250, //położenie...
	vx = 1, vy = 0, //prędkość...
	dx = 4, dy = 4; //rozmiary...
	const int wie = 200; //wielkość tablicy (wonsza)
	int tab[wie][2] = {0,0};	//tablica z współrzędnymi położenia wonsza
	
	unsigned char R = 255, G = 0, B = 0; //kolor wypełnienia
	
	void paint(HDC hdc, int x, int y, int g)
	{
		HPEN hpen = CreatePen(PS_SOLID, 5, RGB(0, g, 0));
		HBRUSH hbrush = CreateSolidBrush(RGB(G, B, B));
		SelectObject(hdc, hpen);
		SelectObject(hdc, hbrush);
		Ellipse(hdc, x, y, x + dx, y + dy);
		SelectObject(hdc, GetStockObject(NULL_PEN));
		SelectObject(hdc, GetStockObject(NULL_BRUSH));
		DeleteObject(hpen);
		DeleteObject(hbrush);
	}
	
	
	
	//NOWE
	//metoda dodająca do tablicy wszpółrzędne wonsza
	void tabAdd(HDC hdc, int x, int y, HWND hwnd)
	{
		for (int i = 1; i < wie-1; i++)
				{
					paint(hdc, tab[i][0], tab[i][1], 255);
				}
		paint(hdc, tab[wie-1][0], tab[wie-1][1], 255); //zamalowanie ostatniej współrzędnej wonsza na czarno
	
		for (int i = wie-1; i > 0; i--) // pętla po całej tablicy
		{
	
			tab[i][0] = tab[i-1][0];	//przesunięcie współrzędnych wonsza o jeden w dół w tablic
			tab[i][1] = tab[i-1][1];		
		}
	
		tab[0][0] = x;		//przypisanie współrzędnych głowy wonsza do 1 elementu tablicy
		tab[0][1] = y;
	
		
	}
	
	//FOK: Funkcja Obsługi Komunikatów (Okienkowych)
	LRESULT CALLBACK wnd_proc(HWND hwnd, UINT message, WPARAM wp, LPARAM lp)
	{
		switch (message)
		{
		case WM_CREATE:
			Przycisk1 = CreateWindowEx(0, "BUTTON", "Koniec", WS_CHILD | WS_VISIBLE, 1, 1, 150, 30, hwnd, NULL, NULL, NULL);
			break;
	
		case WM_KEYDOWN:
		{
	
			switch ((int)wp)
			{
			case VK_DOWN:
				vy = 1;
				vx = 0;
				break;
			case VK_UP:
				vy = -1;
				vx = 0;
				break;
			case VK_LEFT:
				vx = -1;
				vy = 0;
				break;
			case VK_RIGHT:
				vx = 1;
				vy = 0;
				break;	
			case VK_ESCAPE:
				DestroyWindow(hwnd);
				break;
			}
		}
		break;
	
		case WM_PAINT:
		{
			PAINTSTRUCT ps;
			BeginPaint(hwnd, &ps);
			paint(ps.hdc, x, y, 255);
			//NOWE
			//wywołanie metody, która dodaje współrzędne, które ma wonsz do tablicy
			tabAdd(ps.hdc, x, y, hwnd);
			//koniec NOWEgo
			EndPaint(hwnd, &ps);
		}
		break;
	
		case WM_CLOSE:
			KillTimer(hwnd, 1);
			DestroyWindow(hwnd);
			break;
	
		case WM_DESTROY:
			PostQuitMessage(0);
			break;
	
		case WM_COMMAND:
			if ((HWND)lp == Przycisk1)
				MessageBox(hwnd, "Gra zostanie wyłączona", "?!", MB_OK);
			exit(lp);
			break;
		case WM_TIMER:
			switch (wp) {
			case 1:
			{
				RECT r;
				GetClientRect(hwnd, &r);
				if (x + vx + dx > (r.right - r.left) || (x + vx) < 0) DestroyWindow(hwnd);
				if (y + vy + dy > (r.bottom - r.top) || (y + vy) < 0) DestroyWindow(hwnd);
				// NOWE
				// sprawdzanie czy wonsz na siebie wjeżdża
				for (int i = 1; i < wie-1; i++)
				{
					if (tab[i][0] == x + vx)
					{
						if (tab[i][1] == y + vy)
						{
							DestroyWindow(hwnd);
						}
					}
				}
				// koniec NOWEgo
				//przesuniecie o wektor
				x += vx;
				y += vy;
				InvalidateRect(hwnd, NULL, TRUE);
				UpdateWindow(hwnd);
			}
			break;
			}
			break;
		default:
			return DefWindowProc(hwnd, message, wp, lp);
		}
		return 0;
	}
	
	
	int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE, LPSTR, int) {
	
		WNDCLASSEX wc;
	
		wc.cbSize = sizeof(WNDCLASSEX);
		wc.cbClsExtra = 0;
		wc.style = 0;
		wc.lpfnWndProc = wnd_proc;
		wc.cbWndExtra = 0;
		wc.hInstance = hInstance;
		wc.hIcon = LoadIcon(NULL, IDI_APPLICATION);
		wc.hIconSm = LoadIcon(NULL, IDI_APPLICATION);
		wc.hCursor = LoadCursor(NULL, IDC_ARROW);
		wc.hbrBackground = (HBRUSH)(COLOR_BACKGROUND + 1);
		wc.lpszMenuName = NULL;
		wc.lpszClassName = "application_programming_class";
	
		RegisterClassEx(&wc);
	
		HWND window = CreateWindowEx(0, "application_programming_class", "Application programming",
			WS_OVERLAPPEDWINDOW | WS_VISIBLE, 0, 0, 1000, 1000, NULL, NULL, hInstance, NULL);
	
		if (!window) return -1;
	
		SetTimer(window, 1, 1, NULL);
	
		MSG msg;
		while (GetMessage(&msg, 0, 0, 0)) {
			TranslateMessage(&msg);
			DispatchMessage(&msg);
		}
	
		return 1;
	}
