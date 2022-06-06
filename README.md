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

void ChooseGameMode()
{
	GameModeMessage();

	char gameModeKey = '0';
	bool VaildKeyForGameMode = false;

	while (!VaildKeyForGameMode) // 防止用戶按下不適當的鍵
	{
		gameModeKey = _getch();

		if (gameModeKey == 'e' || gameModeKey == 'E' || gameModeKey == 'h' || gameModeKey == 'H')
		{
			VaildKeyForGameMode = true;
		}
	}

	if (gameModeKey == 'e' || gameModeKey == 'E')
	{
		Score59::setGameMode(1); // 1（真）是簡單模式					
	}

	else
	{
		Score59::setGameMode(0); // 0（假）是困難模式					
	}
}

bool StartGame()
{
	DrawWhiteSpace(0, 0, BORDER_RIGHT_WIDE, BORDER_DOWN);

	std::chrono::seconds timeLimit(TIME_LIMIT); // 將 int/double 轉換為 chrono 定義的秒數
	std::chrono::seconds duration(0); // 計算遊戲自開始以來的時間
	std::chrono::seconds leftTime(TIME_LIMIT); // 計算還剩多少秒
	auto start = chrono::steady_clock::now(); // 記錄開始時間
	int gameScore = 0; // 比賽成績

	Student std = Student(STUDENT_INITIAL_X, STUDENT_INITIAL_Y); // 學生一開始的位置
	list<Score59> scores; // 59("五九")的動態列表  
	list<Score59>::iterator s; // 列表的迭代器
	list<Pass> passes; // pass("及格")動態列表
	list<Pass>::iterator p; // 此列表的另一個迭代器

	srand(time(nullptr)); // 為 59s 的位置生成隨機數
	int mapWidth = BORDER_RIGHT - BORDER_LEFT + 1; // 包括邊界
	int mapLength = BORDER_DOWN - BORDER_UP + 1; // 包括邊界
	double rnX = 0;
	double rnY = 0; // 初始化隨機數
	for (int i = 0; i < SCORE59_CNT; i++) // 產生 59s
	{
		rnX = (rand() % mapWidth) + BORDER_LEFT;
		rnY = (rand() % (mapLength / 2)) + (BORDER_UP); //  (mapLength / 2) 一開始防止59s在地圖上太低
		scores.push_back(Score59(rnX, rnY));
	}

	while (duration < timeLimit) // 雖然遊戲還沒有結束
	{
		for (p = passes.begin(); p != passes.end(); p++) // 對於列表中的每一個通行證
		{
			p->Move(); // 向上移動一個 SPEED_PASS 單元
			if (p->isOut()) // 如果通行證到達地圖頂部
			{
				p->Erase(); // 在終端中清除它 
				p = passes.erase(p); // 在列表中刪除它
			}
		}

		for (s = scores.begin(); s != scores.end(); s++) // 對於列表中的每 59 個
		{
			s->Move(); // 向下移動一個 SPEED_SCORE59_EASY /SPEED_SCORE59_HARD 單位
			if (s->isOut()) // 如果 59 到達地圖底部
			{
				scores.push_back(Score59(rnX, BORDER_UP)); // 將新的 59 添加到列表中以保持遊戲中 59 的數量相同
				s->Erase(); // 在終端中清除它 
				s = scores.erase(s); // 在列表中刪除它

				rnX = (rand() % mapWidth) + BORDER_LEFT;
				break;
			}
			
		}

		for (s = scores.begin(); s != scores.end(); s++) // 對於列表中的每 59 個
		{
			// 檢查59是否撞到通行證
			for (p = passes.begin(); p != passes.end(); p++) // 對於列表中的每一個通行證
			{
				if (Collision(s->X(), s->Y(), p->X(), p->Y()))
				{
					scores.push_back(Score59(rnX, BORDER_UP)); // 將新的 59 添加到列表中以保持遊戲中 59 的數量相同
					gameScore += GET_GAME_POINT;
					p->Erase(); // 在終端中清除它
					s->Erase(); // 在終端中清除它  
					p = passes.erase(p); // 在列表中刪除它
					s = scores.erase(s); // 在列表中刪除它

					rnX = (rand() % mapWidth) + BORDER_LEFT;
					break;
				}
				
			}
		}

		for (s = scores.begin(); s != scores.end(); s++) // 對於列表中的每 59 個
		{
			// 檢查59是否撞到學生
			if (Collision(s->X(), s->Y(), std.X(), std.Y()))
			{
				scores.push_back(Score59(rnX, BORDER_UP)); // 將新的 59 添加到列表中以保持遊戲中 59 的數量相同
				gameScore -= LOSE_GAME_POINT;
				std.Erase(); // 在終端中清除它
				s->Erase(); // 在終端中清除它  
				Sleep(SHOW_MSG_SHORT);
				s = scores.erase(s); // 在列表中刪除它

				rnX = (rand() % mapWidth) + BORDER_LEFT;
				break;
			}
			
		}

		if (_kbhit())
		{
			char key = _getch();
			if (key == ' ') // 按空格鍵，然後將一個通行證添加到通行證列表
			{
				passes.push_back(Pass(std.X(), std.Y() - 1));
			}
		}

		std.Move(); // 學生移動

		auto t1 = chrono::steady_clock::now(); // 記錄當前時間
		duration = std::chrono::duration_cast<std::chrono::seconds>(t1 - start); // 計算遊戲進行了多長時間並將持續時間轉換為秒數
		leftTime = timeLimit - duration; // 計算還剩多少秒 

		UpdateInfoBar(gameScore, leftTime); // 向用戶更新遊戲信息

		Sleep(INTERVAL_BETWEEN_EACH_LOOP); // ESSENTIAL,否則遊戲將無法玩
	}

	if (gameScore >= VICTORY_GATE)
		return true;
	else
		return false;
}
	
	bool Collision(double x1, double y1, double x2, double y2) // 檢查 59 是否撞到任何通行證
{
	if (abs(x1 - x2) < EQUALITY_GAP_X) // 59的寬度和“”的寬度不一樣   
	{
		if (abs(y1 - y2) < EQUALITY_GAP_Y)
			return true;
		else
			return false;
	}

	return false;
}

void UpdateInfoBar(int gameScore, std::chrono::seconds leftTime) // 在遊戲過程中向用戶更新遊戲信息
{
	gotoxy(TIME_POS_X, TIME_POS_Y); cout << "剩餘時間: " << leftTime.count() << "	"; // 更新時間 
	// 因為有時數字的位數不同，所以在數字後面打印一些空格以擦除前一個數字中的數字 
	gotoxy(CUR_SCORE_POS_X, CUR_SCORE_POS_Y); cout << "分數: " << gameScore << "	"; // 更新遊戲分數 
	gotoxy(HIS_SCORE_POS_X, HIS_SCORE_POS_Y); cout << "歷史高分: " << HISTORY_HIGH_SCORE; // 更新歷史最高遊戲分數 	
	if (gameScore >= HISTORY_HIGH_SCORE)
	{
		HISTORY_HIGH_SCORE = gameScore;
		gotoxy(HIS_SCORE_POS_X, HIS_SCORE_POS_Y); cout << "歷史高分: " << HISTORY_HIGH_SCORE; // 更新歷史最高遊戲分數 	
	}
}

bool PlayAgainOrNot()
{
	PlayAgainMessage();

	char playAgainKey = '0';
	bool VaildKeyForPlayAgain = false;

	while (!VaildKeyForPlayAgain) // 防止用戶按下不適當的鍵
	{
		playAgainKey = _getch();

		if (playAgainKey == 'y' || playAgainKey == 'Y' || playAgainKey == 'n' || playAgainKey == 'N')
		{
			VaildKeyForPlayAgain = true;
		}
	}

	if (playAgainKey == 'n' || playAgainKey == 'N')
		return false;
	else
		return true;
}
void WelcomeMessage()
{
	DrawWhiteSpace(0, 0, BORDER_RIGHT_WIDE, BORDER_DOWN);
	int x = 20;
	int y = 10;
	gotoxy(x, y); cout << "    _________            /  |              | |             / |      |---------";
	gotoxy(x, y + 1); cout << "   |        |           /    |             |  |           /  |      |         ";
	gotoxy(x, y + 2); cout << "   |        |          /      |            |   |         /   |      |         ";
	gotoxy(x, y + 3); cout << "   |________|         /______  |           |    |       /    |      |---------";
	gotoxy(x, y + 4); cout << "            |        /          |          |     |     /     |      |         ";
	gotoxy(x, y + 5); cout << "            |       /            |         |      |   /      |      |         ";
	gotoxy(x, y + 6); cout << "    ________|      /              |        |       | /       |      |---------";
	gotoxy(40, y + 9); cout << "       物件導向小專題_第15組 ";
	gotoxy(31, y + 12); cout << "Press R/r to see game guide.	Press other keys to play.";
}

void GuideMessage()
{
	DrawWhiteSpace(0, 0, BORDER_RIGHT_WIDE, BORDER_DOWN);
	int x = 44;
	int y = 2;
	gotoxy(x, y); cout << " ";
	y = 5;
	gotoxy(50, y); cout << "       ";
	x = 35;
	gotoxy(x, y + 2); cout << "智仁是個大學教授，為了讓他的學生了解程式的重要性。";
    	gotoxy(x, y + 4); cout << "學生特別製作了一個小遊戲讓教授遊玩。        ";
    	gotoxy(x, y + 6); cout << "當教授回想起當年學程式的辛苦時，就是學生及格的時刻。         ";
    	gotoxy(x, y + 8); cout << "跟著智仁教授一起遊玩學生所精心製作的小遊戲吧 ! ";
    	gotoxy(48, y + 10); cout << "    遊戲玩法    ";
    	gotoxy(x, y + 12); cout << "遊戲當中你將扮演著智仁，遊玩著學生所做的遊戲。";
    	gotoxy(x, y + 14); cout << "發射方式 : 按下鍵盤的空白鍵    移動方式 : 按下鍵盤的方向鍵 ";
    	gotoxy(x, y + 16); cout << "計分方式 : 每消滅一個88即得" << GET_GAME_POINT << "分，若被88打到，倒扣" << LOSE_GAME_POINT << "分";
    	gotoxy(x, y + 18); cout << "遊戲時間共" << TIME_LIMIT << "秒，在時間內打到" << VICTORY_GATE << "分即可PASS。若無請明年再來一次!";
    	gotoxy(x, y + 20); cout << "按下任意鍵開始遊戲";
}

void GameModeMessage()
{
	DrawWhiteSpace(0, 0, BORDER_RIGHT_WIDE, BORDER_DOWN);
	int x = 40;
	int y = 10;
	gotoxy(x, y); cout << "                 ";
	gotoxy(x, y + 1); cout << "              請選擇遊戲難度:";
	gotoxy(x, y + 2); cout << "                ";
	gotoxy(x, y + 3); cout << "                簡單：按 E/e";
	gotoxy(x, y + 4); cout << "                 ";
	gotoxy(x, y + 5); cout << "                困難：按 H/h";
	gotoxy(x, y + 6); cout << "                 ";
}

void VictoryMessage()
{
	DrawWhiteSpace(BORDER_LEFT_WIDE, BORDER_UP, BORDER_RIGHT_WIDE, BORDER_DOWN);
	int x = 25;
	int y = 10;
	gotoxy(x, y); cout << "";
	gotoxy(x, y + 1); cout <<"";
	gotoxy(x, y + 2); cout << "";
	gotoxy(x, y + 3); cout << "";
	gotoxy(x, y + 4); cout << "";
	gotoxy(x, y + 5); cout << "";
	gotoxy(x, y + 6); cout << "";
	gotoxy(x, y + 8); cout << "恭喜過關";
}

void DefeatMessage()
{
	DrawWhiteSpace(BORDER_LEFT_WIDE, BORDER_UP, BORDER_RIGHT_WIDE, BORDER_DOWN);
	int x = 25;
	int y = 10;
	gotoxy(x, y); cout << " ";
	gotoxy(x, y + 1); cout << "";
	gotoxy(x, y + 2); cout << "";
	gotoxy(x, y + 3); cout << "";
	gotoxy(x, y + 4); cout << "";
	gotoxy(x, y + 5); cout << "";
	gotoxy(x, y + 6); cout << "";
	gotoxy(x, y + 8); cout << "再接再厲";
}

void PlayAgainMessage()
{
	DrawWhiteSpace(0, 0, BORDER_RIGHT, BORDER_DOWN);
	int x = 10;
	int y = 10;
	gotoxy(x, y); cout << " ";
	gotoxy(x, y + 1); cout << "";
	gotoxy(x, y + 2); cout << "";
	gotoxy(x, y + 3); cout << "";
	gotoxy(x, y + 4); cout << "";
	gotoxy(x, y + 5); cout << "";
	gotoxy(x, y + 6); cout << "";
	gotoxy(x, y + 8); cout <<"再次挑戰按Y 結束按N";
}

void GoodbyeMessage()
{
	DrawWhiteSpace(0, 0, BORDER_RIGHT_WIDE, BORDER_DOWN);
	int x = 15;
	int y = 10;
	gotoxy(x, y); cout << " ";
	gotoxy(x, y + 1); cout << " ";
	gotoxy(x, y + 2); cout << " ";
	gotoxy(x, y + 3); cout << "";
	gotoxy(x, y + 4); cout << "";
	gotoxy(x, y + 5); cout << "";
	gotoxy(x, y + 6); cout << " ";
	gotoxy(x, y + 8); cout << "Thanks for playing!";
}
