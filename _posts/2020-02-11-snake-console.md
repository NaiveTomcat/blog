---
layout: post
title:  控制台贪吃蛇游戏（C++)
catalog:  true
tags:
        - coding
---

# 疫情隔离在家，闲得无聊，于是写了个贪吃蛇小游戏。实现等在下面讲解。

## 实现思路

利用链表存储蛇的各个节点（像素）位置，各个节点采用包含了`COORD`结构的只有1个成员的结构存储，游戏中地表像素类型及方向分别用枚举实现，整张地图使用二维数组储存，用户交互采用无缓冲无回显输入函数`_getch()`实现。

## 代码展示

**因源码中已有注释，故讲解略**

### Snake.h

```C++
#pragma once
#include <list>
#include <Windows.h>
#include <dos.h>
#include <cstdlib>
#include <ctime>
#include <conio.h>
#include <iostream>


class Snake
{
private:
	enum direct { UP=1, RIGHT=2, DOWN=3, LEFT=4 };
	enum terrType { wall, food, air};
	struct SnakeNode
	{
		COORD pos;
	};

	typedef std::list<SnakeNode> SnakeList;
	SnakeList self;
	direct activeDirect; // Current direction
	direct newDirect;    // Next direction
	terrType map[40][55];
	void init();         // Initialize game
	bool move();         // Move snake
	void end();          // End game
	void draw();         // Draw map and snake
public:
	void start();        //Start game
};
```

### Snake.cpp

```C++
#include "Snake.h"


void Snake::init()
{
	//Initialize map
	for (int i = 0; i < 40; i++)
	{
		for (int j = 0; j < 55; j++)
		{
			if (i == 0 || i == 39)
				map[i][j] = wall;
			else if (j == 0 || j == 54)
				map[i][j] = wall;
			else
				map[i][j] = air;
		}
	}
	//Initialize snake
	self.push_back(SnakeNode{ {13,16} });
	self.push_back(SnakeNode{ {13,15} });
	self.push_back(SnakeNode{ {13,14} });
	activeDirect = RIGHT;
	//Initialize food
	std::srand(std::time(NULL));
	short tmp1;
	short tmp2;
	for (int i = 0; i < 3; i++)
	{
		tmp1 = std::rand() % 38 + 1;
		std::srand(std::rand());
		tmp2 = std::rand() % 53 + 1;
		std::srand(std::rand());
		map[tmp1][tmp2] = food;
	}
}
bool Snake::move()
{
	//Handle keydown
	if (_kbhit())      //Make game won't stuck if no one press the keyboard
	{
		char key;
		key = _getch();
		switch (key) {
		case 'w':
			newDirect = UP;
			break;
		case 'd':
			newDirect = RIGHT;
			break;
		case 's':
			newDirect = DOWN;
			break;
		case 'a':
			newDirect = LEFT;
			break;
		default:
			newDirect = activeDirect;
		}
	}
	else
		newDirect = activeDirect;
	
	//Check direction
	if (std::abs(int(newDirect) - int(activeDirect)) == 2)  // Means new direction is opposite of current direction
		newDirect = activeDirect;  // Reset direction
	activeDirect = newDirect;
	//Collidsion flags
	bool isFood;
	bool isWall;
	bool biteself = false;
	// Checkout new head position
	SnakeNode nextHead = \
		newDirect == UP ? \
		SnakeNode{ {self.front().pos.X,\
		self.front().pos.Y - 1} }\
		:newDirect == RIGHT ? \
		SnakeNode{ {self.front().pos.X + 1,\
		self.front().pos.Y} }\
		:newDirect == DOWN ? \
		SnakeNode{ {self.front().pos.X,\
		self.front().pos.Y + 1} }
		:SnakeNode{ {self.front().pos.X - 1,\
		self.front().pos.Y} };
	// Check collision and food
	isFood = map[nextHead.pos.Y][nextHead.pos.X] == food;
	isWall = map[nextHead.pos.Y][nextHead.pos.X] == wall;
	for (SnakeNode node : self)
		biteself |= (nextHead.pos.X == node.pos.X \
			&& nextHead.pos.Y == node.pos.Y);
	//Main move code
	if (!isWall && !biteself)
	{
		self.push_front(nextHead); // Move forward
		if (!isFood)            // If not eat food
			self.pop_back(); // Delete old tail
		else               // If food eaten
		{
			// Spawn new food
			short tmp1, tmp2;
			tmp1 = std::rand() % 38 + 1;
			std::srand(std::rand());
			tmp2 = std::rand() % 53 + 1;
			std::srand(std::rand());
			map[tmp1][tmp2] = food;
			// Delete eaten food
			map[nextHead.pos.Y][nextHead.pos.X] = air;
		}
		return true;
	}
	else
		return false; //If collide then sent signal to game func
}
void Snake::end()
{
	//Clear screen
	SetConsoleCursorPosition(GetStdHandle(STD_OUTPUT_HANDLE), COORD{ 0,0 });
	system("cls");
	//Print score
	std::cout << "\n\n\nGAME OVER\nYOUR SCORE" << self.size();
	std::cout << "\n\n\a\a\a";
}
void Snake::draw()
{
	COORD coor;
	//Print map
	for (int i = 0; i < 40; i++)
	{
		for (int j = 0; j < 55; j++)
		{
			coor.X = j, coor.Y = i;
			SetConsoleCursorPosition(GetStdHandle(STD_OUTPUT_HANDLE), coor);
			if (map[i][j] == wall)
				printf("▉");
			else if (map[i][j] == food)
				printf("0");
			else
				printf(" ");
		}
	}
	//Print snake
	for (SnakeNode node : self)
	{
		coor = node.pos;
		SetConsoleCursorPosition(GetStdHandle(STD_OUTPUT_HANDLE), coor);
		printf("S");
	}
}
void Snake::start()
{
	init();
	while (move()) {
		draw();
		Sleep(500-3*self.size());
	}
	end();
}
```

### main.cpp

```C++
#include "Snake.h"

int main()
{
	SetConsoleCursorPosition(GetStdHandle(STD_OUTPUT_HANDLE), COORD{ 0,0 }); //Set console cursole position to row 0 column 0
	std::cout << "\n\n\n\n\nPress N for new game and Q to quit.\n";
	char key = _getch();             // Non-buffered non-echoed input
	while (key=='n'||key=='N')
	{
		Snake snake;
		snake.start();
		std::cout << "\n\nPress N for new game and Q to quit.\n";
		key = _getch();
	}
	return 0;
}
```

