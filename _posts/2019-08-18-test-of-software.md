---
layout: post
title:  TestMySoftware
catalog:  true
tags:
        - coding
---


我今天写了一个日志记录软件，可以让我不在对文件内容格式头大。
源代码我即将贴上，有关编程习惯不喜勿喷。
UTF-8和GB18030也是一个问题。VS大坑。

***
### Source

```C++
#include <iostream>
#include <fstream>
#include <string>
#include <ctime>

using namespace std;

ofstream fout;



void mkfile(const char* timent)
{
fout.open(timent);
return;
}

void head(string tit)
{
fout << R"(
---
layout: post
title:  )";
fout << tit << endl << "---\n\n";
return;

}

void ending()
{
fout << R"(
***
[return](https://www.tsinghuamakerxian.cn/)
)";
return;
}

int main()
{

time_t now = time(0);
char* dt = ctime(&now);


cout << "Naive Tomcat Log Entry System\n" << dt << endl;

char Filename[100];
string title;

cout << "Please enter your file name, like yyyy-mm-dd-title.md\n";
cin >> Filename;
mkfile(Filename);
cout << "Enter your title\n";
cin >> title;
head(title);

cout << "Log entry recording start. Every row you entered \nwill be recorded and there's no chance to delete.\nType \"EOT EOT EOT EOT EOT\" to end.\nUse Markdown syntax.\n";

string temp;
getline(cin, temp);
while (temp != "EOT EOT EOT EOT EOT")
{
fout << temp << endl;
getline(cin, temp);
}
fout << "From logging software Naive Tomcat Log Entry System./n/n/n/n";
ending();

cout << "Log entry completed. Good Bye.";
cin.get();
cin.get();
return 0;
}
```
From logging software Naive Tomcat Log Entry System.




***
[return](https://www.tsinghuamakerxian.cn/)
