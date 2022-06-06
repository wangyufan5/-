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
