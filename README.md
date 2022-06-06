# -
學校作業
#include <iostream>
#include <cstdlib> // 生成隨機數 
#include <cmath>
#include <ctime> // 隨機樹種子
#include <chrono> // 定時
#include <list> // 在遊戲中使用動態列表作為 59s 和 pass("及格") 的容器
#include <conio.h> //  從鍵盤輸入到終端
#include <windows.h> // 控制終端
using namespace std;

#define KEY_UP    72 // 箭頭鍵的 ascii 數字
#define KEY_DOWN  80
#define KEY_LEFT  75
#define KEY_RIGHT 77
#define BORDER_UP 2 // 遊戲中物件的邊框
#define BORDER_DOWN 28
#define BORDER_LEFT 43  
#define BORDER_LEFT_WIDE 2  
#define BORDER_RIGHT 73 
#define BORDER_RIGHT_WIDE 115 
#define STUDENT_INITIAL_X 58.5 // 遊戲開始時學生的位置
#define STUDENT_INITIAL_Y 25
#define EQUALITY_GAP_X 1.5 // 用於檢查 59s 是否撞到學生
#define EQUALITY_GAP_Y 1
#define CUR_SCORE_POS_X 50 // 遊戲中游戲信息的位置
#define CUR_SCORE_POS_Y 0  
#define HIS_SCORE_POS_X 80
#define HIS_SCORE_POS_Y 0 
#define TIME_POS_X 20 
#define TIME_POS_Y 0 
#define SPEED_STUDENT 1.1 // 學生的速度
#define SPEED_SCORE59_EASY 0.1 // 簡單模式下59的速度
#define SPEED_SCORE59_HARD 0.5 // 困難模式下59的速度
#define SPEED_PASS 0.5 // 通過速度 
#define TIME_LIMIT 30 // 遊戲時間限制
#define INTERVAL_BETWEEN_EACH_LOOP 20 // 遊戲中每個while循環之間的間隔
#define SHOW_MSG_SHORT 100 // Sleep() 以 ms 為單位
#define SHOW_MSG_LONG 1500
#define SCORE59_CNT 10 // 遊戲中出現的 59 數量 
#define GET_GAME_POINT 2
#define LOSE_GAME_POINT 5
#define VICTORY_GATE 60

void gotoxy(double x, double y); // 允許使用坐標在終端內移動 // 類型為雙精度，因此對象可以移動小於 1 個單位 
void DrawWhiteSpace(int a_x, int a_y, int b_x, int b_y); // 清理終端中的某個空間 
void Initialize(); // 設置控制台標題並隱藏控制台光標 
void ChooseGameMode(); // 選擇遊戲級別
bool StartGame(); // 射擊遊戲本身 
bool Collision(double x1, double y1, double x2, double y2); // 檢查 59 是否撞到任何通行證/學生 
void UpdateInfoBar(int gameScore, std::chrono::seconds leftTime); // 在比賽期間更新比分和時間
bool PlayAgainOrNot(); // 詢問用戶是否再次玩遊戲
void WelcomeMessage(); // 以下是在程序的不同階段給用戶的一些消息
void GuideMessage();
void GameModeMessage();
void VictoryMessage();
void DefeatMessage();
void PlayAgainMessage();
void GoodbyeMessage();

int HISTORY_HIGH_SCORE = 0; // 用於記錄整個節目中的最高分

class Student
{
private:
	double x; // x座標
	double y; // y座標
public:
	Student(double x, double y) { this->x = x; this->y = y; }
	double X() { return x; }
	double Y() { return y; }
	void Draw(); // 在終端中顯示學生的當前位置
	void Erase(); // 清除終端中學生的當前位置 
	void Move(); // 移動學生 (Erase()+Draw())
};

void Student::Draw()
{
	gotoxy(x, y); cout << "傑";
}

void Student::Erase()
{
	gotoxy(x, y); cout << "  ";
}

void Student::Move()
{
	if (_kbhit())
	{
		Erase();

		char key = _getch();
		switch (key)
		{
		case KEY_LEFT: if (x - SPEED_STUDENT > BORDER_LEFT) { x -= SPEED_STUDENT; break; }
		case KEY_RIGHT: if (x + SPEED_STUDENT < BORDER_RIGHT) { x += SPEED_STUDENT; break; }
		case KEY_UP: if (y - SPEED_STUDENT > BORDER_UP) { y -= SPEED_STUDENT; break; }
		case KEY_DOWN: if (y + SPEED_STUDENT < BORDER_DOWN) { y += SPEED_STUDENT; break; }
		}
	}

	Draw(); // 無論用戶按鍵，終端中始終有一個學生	
}

class Score59
{
private:
	double x; // x 座標
	double y; // y 座標
	static bool gameMode;
public:
	static void setGameMode(bool level); // true(1) 是簡單模式，而 false(0) 是困難模式
	Score59(double x, double y) { this->x = x; this->y = y; }
	double X() { return x; }
	double Y() { return y; }
	void Draw(); // 在終端顯示 59 的當前位置
	void Erase(); // 清除終端中學生的當前位置
	void Move(); // 移動 59 
	bool isOut(); // 檢測59是否出界
};

void Score59::Draw()
{
	gotoxy(x, y); cout << "59";
}

void Score59::Erase()
{
	gotoxy(x, y); cout << "  ";
}

void Score59::Move()
{
	Erase();

	if (gameMode == 1)
		y += SPEED_SCORE59_EASY; // 向下移動一個 SPEED_SCORE59 單位
	else
		y += SPEED_SCORE59_HARD;

	Draw();
}

bool Score59::isOut()
{
	if (y > BORDER_DOWN)
		return true;
	else
		return false;
}

bool Score59::gameMode = 1; // 用戶默認以簡單模式玩遊戲

void Score59::setGameMode(bool level)
{
	gameMode = level;
}

class Pass
{
private:
	int x; // x 座標
	int y; // y 座標 
public:
	Pass(double x, double y) { this->x = x; this->y = y; }
	double X() { return x; }
	double Y() { return y; }
	void Draw(); // 在終端顯示當前 pass 的位置 
	void Erase(); // 清除終端中pass的當前位置
	void Move(); // 移動通行證
	bool isOut(); // 檢測傳球是否出界
};

void Pass::Draw()
{
	gotoxy(x, y); cout << "及"; //  當 (y - 1) 表示 y 上方的一行
	gotoxy(x, y + 1); cout << "格"; // (y + 1) 表示 y 下面的一行

}

void Pass::Erase()
{
	gotoxy(x, y); cout << "  ";
	gotoxy(x, y + 1); cout << "  ";
}

void Pass::Move()
{
	Erase();
	y -= SPEED_PASS; // 向上移動一個 SPEED_PASS 單元
	Draw();
}

bool Pass::isOut()
{
	if (y < BORDER_UP)
		return true;
	else
		return false;
}

int main()
{
	Initialize(); // 一些背景設置 

	WelcomeMessage();
	char guideKey = _getch();

	if (guideKey == 'r' || guideKey == 'R') // 按 R/r 查看遊戲指南
	{
		GuideMessage();
		_getch(); // 按任意鍵玩遊戲
	}

	bool playAgain = true;

	while (playAgain)
	{
		ChooseGameMode();

		bool gameVictory = true;
		gameVictory = StartGame();

		if (gameVictory)
		{
			VictoryMessage();
			Sleep(SHOW_MSG_LONG); // 確保用戶可以清楚地看到此消息
		}

		else
		{
			DefeatMessage();
			Sleep(SHOW_MSG_LONG); // 確保用戶可以清楚地看到此消息 	
		}

		playAgain = PlayAgainOrNot();
	}

	GoodbyeMessage();
	Sleep(SHOW_MSG_LONG);
	DrawWhiteSpace(0, 0, BORDER_RIGHT_WIDE, BORDER_DOWN);

	return 0;
}

void gotoxy(double x, double y) // 允許使用坐標在終端內移動 
{	// 類型是雙精度的，所以物體可以移動小於 1 個單位 
	HANDLE hCon = GetStdHandle(STD_OUTPUT_HANDLE);
	COORD dwPos;
	dwPos.X = x; // 從0開始
	dwPos.Y = y; // 從0開始
	SetConsoleCursorPosition(hCon, dwPos);
}

void DrawWhiteSpace(int a_x, int a_y, int b_x, int b_y) // 清理終端中的某個空間 
{
	for (int i = a_x; i <= b_x; i++)
	{
		for (int j = a_y; j <= b_y; j++)
		{
			gotoxy(i, j);
			cout << " ";
		}
	}
}

void Initialize() // 設置控制台標題並隱藏控制台光標
{
	// 設置控制台標題
	SetConsoleTitle("Flunk You");

	// 隱藏控制台光標
	HANDLE hCon = GetStdHandle(STD_OUTPUT_HANDLE);
	CONSOLE_CURSOR_INFO cci;
	cci.dwSize = 1;
	cci.bVisible = FALSE;
	SetConsoleCursorInfo(hCon, &cci);
}
