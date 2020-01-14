---
title: 笑傲江湖DEMO
date: 2019-02-28 21:22:20
categories: C
tags: [C, DEMO, Game]
---
<h1 style="text-align:center">My first console game demo <h1>

![2019-02-2831b3de.png](https://miao.su/images/2019/02/28/2019-02-2831b3de.png)
<!-- more -->


这个demo简单的实现了用户输入方向键, 数字键达到在不同地图切换, 购买商品, 打怪升级等功能, 希望在后期能够实现更加丰富的功能并且能连接网络对战, 
开发环境为Atom,CodeBlocks,MinGW


## 数据类型
1. 道具
2. 地图
3. 背包(背包包括道具类型)
4. 门派
5. 玩家(玩家包含背包以及门派数据类型)
6. 怪物

### 道具 (Prop)

**属性字段**

|属性|定义|
|:-:|:-:|
|编号|int id|
|名称|char name[30]|
|价格|double price|
|等级|int level|
|库存|int stock|
|类别|propType type(使用枚举)|
|最小属性|union(使用联合体)|
|最大属性|union(使用联合体)|
|描述|char desc[200]|


枚举代码
~~~
typedef enum _propType {
    Weapon,     //武器
    Armor,      //防具
    Con,        //消耗品
    Frag,       //碎片
} Type;
~~~
联合体代码  (不同类别拥有不同属性类别)
~~~
union {
    int minAttack;     //如果是武器类别, 则最 低/高 攻击力为
    int minDefence;    //如果是防具类别, 则最 低/高 防御力为
    int minPower;      //如果是消耗品类别, 则最 低/高 回复值为
}
~~~
~~~
要注意若定义枚举中使用 ; 分隔开, 则报错如下(这是我自己踩过的一个坑)
Expected '= constant-expression' or end of enumerator definition
~~~


### 地图(Map)
- 该DEMO的地图大小为8 * 8
- 后期增加地图进入的等级限制

**属性字段** 

|属性|定义|
|:-:|:-:|
|编号|int id|
|名称|char name[20]|
|进入地图的最低等级|minLevel|
|坐标结构|COORD coord|
|地图描述|desc[500]|
~~~
地图的坐标使用window.h头文件下的COORD结构, 使用方式为coord.X = 0, coord.Y = 0
~~~

### 背包 (Bag)
- 背包类型是玩家的一个属性
- 后期改进实现人民币玩家可以提升背包容量上限

**属性字段**

|属性|定义|
|:-:|:-:|
|所属玩家ID|int playId|
|当前的道具数量|int propsCount|
|背包最大容量|maxCount|
|道具数组|Prop props|

### 门派 (Martial)
- 门派为玩家的一个属性
- 后期改进实现人名币玩家解锁更多帮派功能

**属性字段**

|属性|定义|
|:-:|:-:|
|编号|int id|
|名称|char name[20]|
|门派类型|char type[10]|
|门派总部坐标结构|COORD hqCoord|
|是否对玩家开放|int isOpen|
|门派描述|char desc[1000]|

### 玩家 (Player)

**属性字段**

|属性|定义|
|:-:|:-:|
|名称|char name[30]|
|编号|id[10]|
|密码|char passwd[30]|
|等级|int level|
|经验值|int exp|
|血量|int hp|
|内力|int mp|
|金币|int gold|
|门派结构|Martial martial|
|武器道具|Prop weapon|
|防具道具|Prop armor|
|玩家当前坐标|COORD coord|
|背包结构|Bag bag|

### 怪物 (Monster)
- 后期加入怪物掉落装备功能

**属性字段**

|属性|定义|
|:-:|:-:|
|编号|int id|
|名称|char name[20]|
|等级|int level|
|血量|int hp|
|攻击力|int attact|
|防御力|int defence|
|掉落的最小金币|int minMoney|
|掉落的最大金币|int maxMoney|
|玩家能够获取的最大经验|int exp|
|怪物的状态(0死1活)|int state|
|怪物的坐标|COORD coord|
***
## 功能实现

**通过系统函数辅助实现的方法, 以下方法在GameLib源文件中**
1. 更改控制台标题(SetTitle)
2. 更改控制台内字体(SetColor)
3. 设置控制台内光标的位置(SetPosition)
  
### 更改控制台标题(SetTitle)
~~~
/** 设置控制台的标题 */
void SetTitle(char *title) {
    SetConsoleTitle(title);     //调用windows.h头文件下的系统函数
}
~~~
### 更改控制台内字体(SetColor)
~~~
/* 设置控制台的颜色
 * 0-黑色,   1-蓝色,   2-绿色,    3-浅绿色,  4-红色,
 * 5-紫色,   6-黄色,   7-白色,    8-灰色,   9-淡蓝色,
 * 10-淡绿色,  11-淡浅绿色   12-淡红色   13-淡紫色
 * 14-淡黄色   15-亮白色
 */
void SetColor(int ForeColor, int BackColor) {//分别设置前景色与背景色
     //HANDLE: 句柄, GetStdHandle(STD_OUTPUT_HANDLE)获得标准输出流的句柄, 均为windows.h下的系统函数
     HANDLE handle = GetStdHandle(STD_OUTPUT_HANDLE);       //获取当前窗口句柄
     SetConsoleTextAttribute(handle, ForeColor + BackColor * 0x10);//设置颜色
   
    //句柄(HANDLE)补充:
    //WINDOWS程序中并不是用物理地址来标识一个内存块，文件，任务或动态装入模块的。
    //相反，WINDOWS API给这些项目分配确定的句柄并将句柄返回给应用程序，
    //然后通过句柄来进行操作(句柄不是指针, 是32或64位的无符号整型)
 }
~~~
### 设置控制台内光标的位置(SetPosition)
~~~
void SetPosition(int x, int y) {
     HANDLE winHandle = GetStdHandle(STD_OUTPUT_HANDLE);
     COORD pos = {x, y};
     //SetConsoleCursorPosition()为windows.h下系统函数
     SetConsoleCursorPosition(winHandle, pos);      //设置光标位置
 }
~~~
***
**以下方法为自己定义实现游戏主要逻辑功能中比较重要的几个, 在Game源文件中**
1. **GameProcess():** 接收并判断用户的输入, 返回游戏的各个功能(例如用户输入方向键将会移动玩家的所在地图,输入1会显示玩家的个人信息窗口等等)这是游戏最重要的一个方法, 决定了游戏的逻辑实现, 本人水平较差, 当初没有想到将所有的操作均封装到此方法来, 因此使得判断用户输入包括方向键的接收为一个方法并且放入于main()中, 导致了维护困难, 代码不清晰等等问题 
2. **Clear():** 功能为清理指定区域内的所有字符, 用'\0'替代, 达到刷新屏幕的功能, 由于清屏操作需要使用的次数极多因此封装成方法
3. **ShowMonster():** 该方法为显示当前地图所有怪物的信息: 等级名称, 但是我却在一个方法包含了接收用户输入导致方法过于复杂难以复用维护, 而且此方法打印怪物使用指针以及辅助数组较难理解
4. **MonsterFight():** 接收用户输入的怪物编号, 让玩家与怪物进行战斗, 以当前时间为种子, 攻击力防御力取范围该范围内的随机数, 战斗后返回判断打印不同情况的结果 
5. **PropInit():** 初始化游戏玩家数据, 用于测试功能 
6. **Trade():** 交易方法, 接收游戏玩家编号与商品编号, 重点为判断无法交易的情况以及对背包物品的添加(背包已有该类别商品/背包暂无此类别商品) 
7. **ShowProps():** 显示商城的信息, 此方法与ShowMonster()类似,  同样我没考虑到方法的复用以及维护, 将Show 和 接收用户的输入 两个方法写在一个方法里了
8. **showTrade():** 接收Trade()返回的值, 对应交易成功与失败分别执行不同操作
9. **Login():** 登陆方法, 接收用户输入的用户名以及密码(无回显)进行判断以达到登陆功能

#### 以下为主要方法的源代码
- 一些宏定义, 例如游戏窗口的各个部分的行数等等, 在移动光标时使用频率极高
~~~
#define COL 78                //游戏的界面的总宽度
#define MAXGIN_X 20           //游戏界面与控制台的左边距
#define ENDINFO 78            //各个框架结束'|'的列数
#define MAPSART_START_Y 3     //游戏地图开始的行
#define MAPSART_END_Y 11      //游戏地图结束行
#define INFORMATION_START_Y 12//游戏信息开始行
#define INFORMATION_END_Y 19  //游戏信息结束行
#define MAINMENUE_START_Y 20  //游戏主菜单开始行
#define MAINMENUE_END_Y 28    //游戏主菜单结束行
#define SEP "*******************************************************************************"
~~~
- 游戏主体的逻辑实现
~~~
接收main()传递的key, 分别执行不同的功能
/** 执行游戏主菜单功能 */
void GameProcess(char key) {
    switch (key) {
        case '1': ShowPlayerInfo();//显示玩家个人信息
            break;
        case '2': ShowMonster();//显示当前地图的怪物(内有打怪)
            break;
        case '3': Move(currPlayer->martial.hqCoord.X, currPlayer->martial.hqCoord.Y);//移动到玩家的帮派地图中
            break;
        case '4': ShowProps();//展示游戏商品
            break;
        default : printf("少侠究竟想要作甚?");
    }
}

以下为游戏主逻辑, 死循环中对用户输入进行判断并对应相应的输出(即实现不同游戏功能)
****************************************************************************
main()包括了判断字符进行移动玩家所在地图的功能(未封装成方法!)
****************************************************************************
int main() {//最好通过一个游戏进程函数将main方法里边的各种方法封装, 通过不同状态执行不同方法
    char key;           //接收用户输入
    SetColor(2,0);      //设置控制台文本颜色(front color为绿色, background color为黑色)
    SetConsoleTitle("笑傲江湖之精忠报国 C语言实现 BY 左手工匠");    //设置控制台标题
    PropInit();         //初始化测试人物
    ShowWelcome();      //显示欢迎栏
    Login();            //登陆, 登陆成功继续执行
    ShowMap();          //显示地图
    ShowInformation();  //显示信息栏
    ShowMainMenu();     //显示菜单栏
    while (1) {//死循环, 除非用户选择退出, 否则停止等待用户输入
    fflush(stdin);
    //接收用户输入(无回显)
    key = getch();
    fflush(stdin);
    if(key == '1' || key == '2' || key == '3' || key == '4'){
        GameProcess(key);//输入正常, 将值传递给GameProcess()来对应输出
        continue;
    }
    else if(key == '5' || key == '6' ){
        SetPosition(20, 12);
        Clear(20 , 12, 7);
        SetPosition(20 + 5, 12 + 2);
        printf("放过我吧, 还没开发完呐!!!\n");
    }
    else if(key == '7') {
        break;
    }
    else if(key == VK_UP || key == 72) {//X, Y为游戏坐标, 控制X, YS实现地图移动
        Y--;            //输入上
    }
    else if(key == VK_RIGHT || key == 77) {
        X++;            //输入右
    }
    else if(key == VK_DOWN || key == 80) {
        Y++;            //输入下
    }
    else if(key == VK_LEFT || key == 75) {
        X--;            //输入左
    }
    if(X > 7) {//移动到最边的话重新回到0, 如贪食蛇穿过地图边缘
        X = 0;
    }
    if(X < 0) {
        X = 7;
    }
    if(Y > 7) {
        Y = 0;
    }
    if(Y < 0) {
        Y = 7;
    }
    ShowMap();
}
    return 0;
}
~~~

- 清空指定区域内所有字符 
~~~
/** 清除信息区内所有内容(给定坐标及清理的行数) */
void Clear(int x, int y, int rowCount) {
    for(int i = 0; i < rowCount; i ++) {
        SetPosition(x + 1, y + i);
        for(int j = 0; j < COL - 1; j ++)  {    //使用' '替换指定区域
            printf(" ");
        }
    }
}
~~~

- 打印怪物的信息以及接收并判断玩家的输入的怪物编号, 玩家选择查看怪物信息时调用该方法
~~~
/** 显示当前地图的怪物********(功能略复杂些)*************/
void ShowMonster() {
    //给怪物各个等级对应相应称号
    //例如：3级怪物，那么就显示这个怪物的描述为粗通皮毛
    char *monsterlevelNames[] = {"乳臭未干", "初出茅庐", "粗通皮毛", "青年才俊", "略有小成", "心领神会", "出类拔萃", "所向无敌", "天人合一"};
    Clear(MAXGIN_X,  INFORMATION_START_Y, 7);   //清空信息区的所有内容
    int monsterCount;
    // 记录整个游戏的怪物总数
    monsterCount = sizeof(monsterArray) / sizeof(Monster);
    //记录当前地图的怪数量 记得变量定义时赋初值!!! debug了好久...
    int currMapMonsterCount = 0;
    // 定义一个只有九个空间的一维数组, 用以记录怪物在怪物列表里的下标
    int monsterIndex[9];
    for(int i = 0; i < monsterCount; i++) {
        // 关键!!! 满足怪物坐标与当前地图坐标相等 并且 该怪物的状态为存活的条件, 将怪物与怪物数组的下标记录并且当前地图怪物数量自增
        if(monsterArray[i].coord.X == X && monsterArray[i].coord.Y == Y && monsterArray[i].state == 1) {
            // 记录下标
            monsterIndex[currMapMonsterCount] = i;
            currMapMonsterCount++;
            // 设定每个地图最多显示 9 个怪物
            if(currMapMonsterCount == 9) {
                break;
            }
        }
    }

    // 打印怪物
    SetPosition(MAXGIN_X + 5, INFORMATION_START_Y + 1);
    if(currMapMonsterCount == 0) {
        SetColor(7, 0);//白字黑底
        printf("这里冷冷清清的什么也没有, 少侠还是到别处看看吧, 呆着也没银子给你!");
        SetColor(2, 0);//恢复颜色
        return;
    }

/*********以下语句为判定用户输入的编号来返回地图信息或进行打怪操作*******/
    int pkMonsterId = 0;// 接收玩家输入的怪物编号, 定义在循环外部是因为循环外仍需使用进行判断是否打印地图信息
    while(1){
        pkMonsterId = -1;
        Clear(MAXGIN_X,  INFORMATION_START_Y, 7);   //清空信息区的所有内容
        SetPosition(MAXGIN_X + 20, INFORMATION_START_Y + 0);
        printf("少侠, 这儿有怪物! 要为百姓除暴安良吗?" );
        SetPosition(MAXGIN_X + 5, INFORMATION_START_Y + 2);
        for(int i = 0; i < currMapMonsterCount; i++) {
            if(i % 3 == 0 && i != 0){//每行打印三个怪物
                SetPosition(MAXGIN_X + 5, INFORMATION_START_Y + 2 + i / 3);
            }
            //打印怪物, 较为繁琐
            //monsterIndex[i]即为该怪物列表中的怪物编号
            SetColor(4, 0);//红字怪物
            printf("%d.%s(%s)\t  ", i + 1, monsterArray[monsterIndex[i]].name, monsterlevelNames[monsterArray[monsterIndex[i]].level - 1]);
        }
        SetColor(2, 0);//恢复颜色
        SetPosition(MAXGIN_X + 5, INFORMATION_END_Y - 1);
        printf("少侠想攻击几号怪物呢? (按0返回)");
        scanf("%d", &pkMonsterId);
        if(pkMonsterId == 0) {//输入0, 退出循环来返回显示地图信息
            break;
        }
        else if(pkMonsterId < 0 || pkMonsterId > currMapMonsterCount) {
            //输入的怪物标号在当前地图不存在
            SetPosition(MAXGIN_X + 5, INFORMATION_END_Y - 1);
            //简易清屏
            printf("                                                             ");
            SetPosition(MAXGIN_X + 5, INFORMATION_END_Y - 1);
            printf("少侠选错了吧, 怪物不存在呢!(按任意键重新输入)");
            char recoverKey = getch();
            if(recoverKey){//停顿直到接收任意键后重新显示怪物
                continue;
            }
        }
        else {//编号对应地图所有的怪物时, 将怪物地址传入打怪方法
            MonsterFight(&monsterArray[monsterIndex[pkMonsterId - 1]]);
            break;
        }
    }
    if(pkMonsterId == 0) {//输入0, 返回显示地图信息
        ShowMapInfo();
    }
}
~~~
- 玩家与指定地图指定怪兽进行战斗
~~~
/** 选定怪物进行对战 */
void MonsterFight(Monster *monster) {
    SetPosition(MAXGIN_X + 5, INFORMATION_END_Y - 1);
    printf("                                                             ");//简易清屏(偷懒)
    SetPosition(MAXGIN_X + 5, INFORMATION_END_Y - 1);
    srand(time(NULL));//以当前时间为种子
    /* random函数的用法(只能0 - 32767之间)
    * 随机一个a ~ b的数字, 即 rand() % (b - a + 1) + a; 例如0 ~ 5, 即rand() % 6;
    */
    int playerAttact = 0;
    // 玩家的实际攻击力
    playerAttact = rand() % (currPlayer->weapon.maxAttack - currPlayer->weapon.minAttack + 1) + currPlayer->weapon.minAttack;
    int playerDeffence =0;
    //玩家实际防御力
    playerDeffence = rand() % (currPlayer->armor.maxDefence - currPlayer->armor.minDefence + 1) + currPlayer->armor.minDefence;
    //怪物实际掉落的金钱
    int monsterGold = 0;
    monsterGold = rand() % (monster->maxMoney - monster->minMoney + 1) + monster->minMoney;
    int pkRound = 0;    //记录战斗的轮次
    while(1){//打斗循环, 任意一方死亡退出
        //玩家血量变化
        currPlayer->hp -= monster->attact - playerDeffence;
        //怪物的血量变化
        monster->hp -= playerAttact - monster->defence;
        if(monster->hp <= 0) {//怪物的血量少于等于0时退出
            break;
        }
        if(currPlayer->hp <= 0) {//玩家血量小于等于0时退出
            break;
        }
        //以下为打印战斗详情
        SetPosition(MAXGIN_X + 5, INFORMATION_END_Y - 1);
        printf("                                                                         ");//简易清屏(偷懒)
        SetPosition(MAXGIN_X + 5, INFORMATION_END_Y - 1);
        pkRound++;
        //战斗详情使用白底红字
        SetColor(4,7);
        //该语句较长......
        printf("第 %d 轮战斗详情:%s 对 %s 造成了 %d 伤害且受到 %s %d 的伤害", pkRound, currPlayer->name, monster->name, playerAttact - monster->defence, monster->name, monster->attact - playerDeffence);
        //放慢过程
        usleep(1000 * 800);//1000毫秒*800
        SetColor(2, 0);
    }
    SetPosition(MAXGIN_X + 5, INFORMATION_END_Y - 1);
    printf("                                                                   ");//简易清屏(偷懒)
    SetPosition(MAXGIN_X + 5, INFORMATION_END_Y - 1);
    if(currPlayer->hp <= 0) {
        //玩家死亡输入完任意键后返回
        printf("江湖快讯: 大侠 %s 在与 %s 的对决中壮烈牺牲! (请按任意键返回)", currPlayer->name, monster->name);
        //重置人物的金币与血量
        currPlayer->hp = 100;
        currPlayer->gold = 100;
        getch();
        return;
    }
    //以下即为怪物死亡的情况
    printf("%s 轻而易举的便被 %s 大侠收拾了, 同时掉落了 ", monster->name, currPlayer->name);
    //将怪物的状态调整至死亡
    monster->state = 0;
    //设置金币的front color为黄色, background color为亮白色
    SetColor(6, 15);
    printf("%d 金币   ", monsterGold);
    // 设置经验的front color为黄色, background color为亮白色
    SetColor(1, 15);
    printf("%d 经验  ", monster->exp);
    SetColor(2, 0); //颜色恢复
    //玩家获得金币与经验值的加成
    currPlayer->gold += monsterGold;
    currPlayer->exp += monster->exp;
    getch();
    ShowMonster();
}
~~~
初始化游戏数据: 将地图, 玩家, 装备, 怪物置入, 测试各功能
~~~
Player *currPlayer;
void PropInit() {//初始化游戏数据 (测试用数据)
    currPlayer = &playArray[0];
    currPlayer->weapon = propArray[8];
    currPlayer->armor = propArray[4];
    currPlayer->martial = martials[4];
    Bag bag = {95001, 0, 8};
    currPlayer->bag = bag;
}
~~~
交易, 玩家购买指定道具
~~~
/** 交易
  * 参数1: 玩家的地址
  * 参数2: 商品的编号
  * 返回交易是否成功, 0 失败, 1 成功
  */
int Trade(Player *player, int propsId) {
    Prop *tradeProp = NULL;     //定义一个用于指向及交易商品的指针
    for(int i = 0; i < sizeof(propArray) / sizeof(Prop); i++) {
        if(propsId == propArray[i].id) {//找到该商品, 并让tradeProp指向它
            tradeProp = propArray + i;  //即tradeProp = &propArray[i]
            break;
        }
    }
    //判断商品是否有库存
    if(tradeProp->stock <= 0) {
        Clear(MAXGIN_X, INFORMATION_END_Y - 1, 1);
        SetPosition(MAXGIN_X + 5, INFORMATION_END_Y - 1);
        printf("地主家都没有余粮！商店都被买空啦！");
        return 0;
    }
    //判断玩家是否金币足够支付
    if(currPlayer->gold < tradeProp->price) {
        Clear(MAXGIN_X, INFORMATION_END_Y - 1, 1);
        SetPosition(MAXGIN_X + 5, INFORMATION_END_Y - 1);
        printf("钱包都是瘪的，这可是个看钱的江湖！！!");
        return 0;
    }
    //判断玩家背包容量是否充足
    if(currPlayer->bag.propsCount > currPlayer->bag.maxCount && currPlayer->bag.propsCount != 0) {
        //玩家的背包道具数量大于最大容量并且背包道具数不为0则判定为失败
        Clear(MAXGIN_X, INFORMATION_END_Y - 1, 1);
        SetPosition(MAXGIN_X + 5, INFORMATION_END_Y - 1);
        printf("背包已满，交易失败！");
        return 0;
    }
    //满足交易条件，执行交易的业务操作
    //1、商品库存-1
    tradeProp->stock--;
    //2、玩家金币-商品单价
    player->gold -= tradeProp->price;

    //*****************************玩家背包道具增加*******************************//
    //判断玩家背包中是否已有该商品 !!!
    int i;
    for(i = 0; i < currPlayer->bag.propsCount; i++) {
        if(propsId == currPlayer->bag.props[i].id) {
            currPlayer->bag.props[i].stock++;
            break;
        }
    }
    //若为循环正常结束, i = currPlayer->bag.propsCount, 即为背包无此商品, 需要给背包添加该商品!!!
    if(i == currPlayer->bag.propsCount) {
        //向背包中创建一个商品-复制一份要交易的商品信息到背包中
        currPlayer->bag.props[currPlayer->bag.propsCount].id = tradeProp->id;
        currPlayer->bag.props[currPlayer->bag.propsCount].price = tradeProp->price;
        currPlayer->bag.props[currPlayer->bag.propsCount].stock = 1;
        strcpy(currPlayer->bag.props[currPlayer->bag.propsCount].name, tradeProp->name);
        strcpy(currPlayer->bag.props[currPlayer->bag.propsCount].desc, tradeProp->desc);
        currPlayer->bag.propsCount++;
    }
    //交易成功
    return 1;
}
~~~
- 打印商品列表, 玩家选择购买道具调用该方法
~~~
/** 展示游戏商品 */
void ShowProps() {
    //此方法应拆分为打印商品 以及用户输入并检验商品编号两个方法!!!!!!!!
    /*******************************/
    Clear(MAXGIN_X, INFORMATION_START_Y, 7);
    SetPosition(MAXGIN_X + 5, INFORMATION_START_Y);
    SetColor(8, 0);     //灰字黑底
    printf("欢迎大侠 %s 来到 %s 童叟无欺的杂货铺, 瞧一瞧看一看可有需要的?", currPlayer->name, mapArray[X][Y].name);
    SetColor(6, 0);     //黄字黑底
    SetPosition(MAXGIN_X + 5, INFORMATION_START_Y + 2);
    int propCount = sizeof(propArray) / sizeof(Prop) >= 9 ? 9 : sizeof(propArray) / sizeof(Prop);
    for(int i = 0; i < propCount; i++) {
        if(i % 3 == 0) {//打印商品, 每行 3 个, 且只打印最多 9 个
            SetPosition(MAXGIN_X + 5, INFORMATION_START_Y + 2 + i / 3);
        }
        printf("%-3d.%-10s(%-2d)%-4c", propArray[i].id, propArray[i].name, propArray[i].stock, ' ');
    }
    SetColor(2, 0);//恢复颜色

    int tradeId;//接收用户输入的商品编号
    while(1) {
        //用于判断输入编号是否正确(此判断未包括检验玩家金币及商品库存及玩家背包是否已满等状态)
        Clear(MAXGIN_X, INFORMATION_END_Y - 1, 1);
        SetPosition(MAXGIN_X + 5, INFORMATION_END_Y - 1);
        printf("大侠需要什么, 请输入编号立即购买(按 0 退出)");
        scanf("%d", &tradeId);
        if(tradeId == 0) {//输入0返回显示地图信息
            ShowMapInfo();
            break;
        }
        else if(tradeId < 0 || tradeId > 9) {//输入的商品编号错误的情况, 重新输入
            Clear(MAXGIN_X, INFORMATION_END_Y - 1, 1);
            SetPosition(MAXGIN_X + 5, INFORMATION_END_Y - 1);
            printf("少侠选错了吧, 商品不存在呢!(按任意键重新输入)");
            char recoverKey = getch();
            if(recoverKey){//停顿直到接收任意键后重新显示怪物
                continue;
            }
        }
        else {//输入在1 - 9的情况, 将编号传入交易函数
            showTrade(Trade(currPlayer, tradeId), tradeId);//******调用交易方法, 展示交易信息方法******//
            //以下为重新刷新商品列表, 应该封装为一个打印商品的方法, 该ShowProps()方法过于繁琐, 加入了打印以及检验两个功能
            /********************************/
            Clear(MAXGIN_X, INFORMATION_START_Y, 7);
            SetPosition(MAXGIN_X + 5, INFORMATION_START_Y);
            SetColor(8, 0);     //灰字黑底
            printf("欢迎大侠 %s 来到 %s 童叟无欺的杂货铺, 瞧一瞧看一看可有需要的?", currPlayer->name, mapArray[X][Y].name);
            SetColor(6, 0);     //黄字黑底
            SetPosition(MAXGIN_X + 5, INFORMATION_START_Y + 2);
            int propCount = sizeof(propArray) / sizeof(Prop) >= 9 ? 9 : sizeof(propArray) / sizeof(Prop);
            for(int i = 0; i < propCount; i++) {
                if(i % 3 == 0) {//打印商品, 每行 3 个, 且只打印最多 9 个
                    SetPosition(MAXGIN_X + 5, INFORMATION_START_Y + 2 + i / 3);
                }
                printf("%-3d.%-10s(%-2d)%-4c", propArray[i].id, propArray[i].name, propArray[i].stock, ' ');
            }
            SetColor(2, 0);//恢复颜色
            /********************************/
            }
        }
}
~~~
- 返回交易结束后的信息
~~~
/** 接收Trade()返回的值, 判断交易是否成功 */
void showTrade(int flag, int propId) {
    if(flag == 0) {
        ShowMapInfo();
        return;
    }
    if(flag == 1) {
        Clear(MAXGIN_X, INFORMATION_END_Y - 1, 1);
        SetPosition(MAXGIN_X + 5, INFORMATION_END_Y - 1);
        printf("交易成功, ");
        SetColor(6, 0);     //打印装备为黑底黄字
        printf("%s", propArray[propId - 1].name);
        SetColor(2, 0);     //恢复黑底绿字
        printf(" 已经加入你的背包, 赶紧瞧一瞧吧!(按任意键继续)");
        getch();            //暂停
    }
}
~~~
验证登陆信息
~~~
/** 玩家登陆(暂无注册) */
void Login() {
    while(1) {
        char id[10];
        char key[10];       //接收用户输入的id 和 passwd
        SetColor(2, 0);
        Clear(MAXGIN_X, MAPSART_START_Y, 4);
        SetPosition(MAXGIN_X + 24, MAPSART_START_Y);
        printf("请在下方输入您的用户名和密码");
        SetPosition(MAXGIN_X + 5, MAPSART_START_Y + 1);
        printf("用户名: ");
        scanf("%s", id);
        SetPosition(MAXGIN_X + 5, MAPSART_START_Y  + 2);
        printf("密码(无回显哦回车确定): ");
        for(int i = 0; 1; i++) {
            key[i] = getch();//无回显输入密码
            if(key[i] == 13) {
                //接收到回车跳出循环
                //将回车字符重置为空字符
                key[i] = '\0';
                break;
            }
        }
        //判断玩家输入的信息是否与玩家id, passwd匹配
        if(strcmp(currPlayer->id, id) == 0 && strcmp(currPlayer->passwd, key) == 0) {
            //只有玩家输入成功才跳出死循环
            break;
        }
        else {
            SetPosition(MAXGIN_X + 5, MAPSART_START_Y + 3);
            SetColor(12, 0);    //浅红色字体
            printf("用户名或密码错误, 请检查后再次输入!(按任意键继续)");
            getch();
        }
    }
    //用户输入成功后
    SetColor(12, 0);
    SetPosition(MAXGIN_X, MAPSART_START_Y);
    Clear(MAXGIN_X, MAPSART_START_Y, 4);
    SetPosition(MAXGIN_X + 15, MAPSART_START_Y);
    printf("欢迎 ");
    SetColor(1, 0);      //蓝字显示玩家name
    printf("%s", currPlayer->name);
    SetColor(2, 0);
    printf(" 成功登陆笑傲江湖的世界(按任意键继续)");
    getch();    //暂停
}
~~~
## 拓展功能
以下的功能为此DEMO适合拓展的有趣功能:
- 玩家的背包容量: 初始设定只有9, 可以通过充值元宝等方式扩充背包最大容量
- 注册功能, 玩家设定任意符合指定规则的用户名及密码, 并且在下次可以继续使用
- NPC, 逻辑与怪物相似, 地图随机刷新NPC并且贩卖普通商店没有的道具
- 打斗过程加入回复, 增益等功能(打斗前使用消耗品提升属性, 打斗过程回复血量, 内力)
- 怪物指定时间刷新
- 连接网络, 满足两个玩家在同一张地图进行, 怪物道具资源同步刷新, 并且玩家可以决斗(遥远的目标啊)

## 最后的吐槽

先说说为什么要写这个极小极小的DEMO吧, 寒假前半段在学数据结构, 时间较短, 仅仅实现了顺序表, 单链表, 循环链表等几个简单的数据结构的定义以及实现, 过程不算太美好: 虽然大二上讲过数据结构与算法, 但是非常的粗略, 对数据结构与算法的理解仅仅为有那么几种表, 有那么几种排序方式, 而且C是大一上学的, 早已忘得一干二净, 很多语法尤其是之前就不过关的指针的不熟悉, 应验了那句: 当C的指针指向你的时候, 菜鸡原形毕露! 被头指针头结点折腾的头皮发麻, 尽管磕磕绊绊用代码实现了, 但我深刻感受到了效率的低下, 进度十分缓慢, 遂觉得应该恶补以下C, 于是有了以上的DEMO

前前后后一共花了十天, 四天的时间都是疯狂刷视频, 把指针, 结构体, 字符串, 数组这四个章节重点看了一到两遍, 然后六天把这个DEMO完成了, 跟着老九的视频, 每天看完示范再理解完敲代码实现, 慢慢的也完成了, 自己也没想到可以写出1000行的东西(可能是我注释比较多吧捂脸), 过程最难过的不是理解也不是一行一行敲出一个功能, 而是无穷无尽的BUG, 早期是用Clion + Cmake的环境, 第二天开始实现游戏的界面栏目并且改变控制台标题栏以及字体颜色时, 因为要在控制台检验, 但是发现Clion + Cmake不能调出Windows的Console, 一顿查阅亦未果, 因此使用了最早使用的Codeblocks(还是这个简单多了...), 嫌弃Cb丑陋UI的我又入坑了Atom, 用Atom码代码, Cb编译.....十分的奇葩, 但是也算是完成了

总体来讲, 对我感触最深的东西应该就是封装以及设计了, 这次的初版的DEMO我总体时非常不满意的, 糟糕的代码, 凌乱的代码, 混乱的设计, 可能源于我过于追求速度, 也许我做出了和视频效果相似甚至更加花里胡哨的打印结果, 但是里面的东西差距时非常遥远的, 空有其表, 败絮其中. 这种代码在日后非常难以维护, 即使我加了非常多的代码, 缩进也尽量按照标准来, 但是我自己都很难找到一个方法在哪里, 非常难以修改, 修改了一个方法可能需要同时修改其他的很多地方, 或者是修改完出现各种各样的内存问题等等...我现在认为一个方法仅仅对于一个功能是非常重要的, 而且便于记忆以及修改的宏定义我也尝到了它的甜头, 还有对整体程序的逻辑设计, 在不考虑这些东西而仅仅根据几条需求如同大一考试对于问题对于解出来就完事儿了的态度去码代码, 码出来的程序可能是正常运行的, 但是接下来的维护呢, 况且随心所欲的写方法我现在感觉是非常可怕的, 乱码一时爽, 码完以后火葬场, 每次DEBUG的时间都与真实写代码的时间一样长了, 每次改完以后心里憔悴, 这都是不好好规划设计, 拿起手就是敲的后果, 正所谓: 代码千万条, 规范第一条, 代码不规范, 亲人两行泪!写完这个DEMO也让我更加深入理解了一些数据结构, 如同此DEMO中的玩家结构, 玩家中有背包, 背包中有道具, 道具中有类别对应不同属性, 武器对应攻击力, 防具对应防御力等等, 现在我可能会思考, 需要经常插入而又很少需要随机读取的类型会不使用链表更加好一些等等的问题, 看来这波不亏

最后, 可能我不会再继续修改这个DEMO了, 要么也是重写了(实在太烂了...) 感谢这个DEMO, 感谢老九, 用这样一种方式让我认识到了上课所无法理解的思想, 让我对未来的代码有了期待, 但是这只是一个极小的开始, 这只是站在了一个小石子儿看世界, 要想看到真正的世界, 还需走的更远, 爬到更高的地方去眺望, 去欣赏, 去体味)