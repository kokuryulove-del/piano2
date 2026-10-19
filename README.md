#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define DICE_COUNT 5
#define CATEGORY_COUNT 12
#define PLAYER_COUNT 2

// カテゴリ名
const char* category_names[CATEGORY_COUNT] = {
    "ONES",
    "TWOS",
    "THREES",
    "FOURS",
    "FIVES",
    "SIXES",
    "CHOICE",
    "FOUR OF A KIND",
    "FULL HOUSE",
    "SMALL STRAIGHT",
    "LARGE STRAIGHT",
    "YACHT"
};

// サイコロを振る
void roll_dice(int dice[], int keep[])
{
    int i;

    for (i = 0; i < DICE_COUNT; i++)
    {
        if (keep[i] == 0)
        {
            dice[i] = rand() % 6 + 1;
        }
    }
}

// サイコロを表示
void show_dice(int dice[])
{
    int i;

    printf("\nサイコロ：");

    for (i = 0; i < DICE_COUNT; i++)
    {
        printf("[%d] ", dice[i]);
    }

    printf("\n");
}

// 指定した数字が何個あるか調べる
int count_number(int dice[], int number)
{
    int i;
    int count = 0;

    for (i = 0; i < DICE_COUNT; i++)
    {
        if (dice[i] == number)
        {
            count++;
        }
    }

    return count;
}

// 得点を計算
int calculate_score(int dice[], int category)
{
    int i;
    int score = 0;

    // ONES ～ SIXES
    if (category >= 0 && category <= 5)
    {
        int number = category + 1;

        for (i = 0; i < DICE_COUNT; i++)
        {
            if (dice[i] == number)
            {
                score += number;
            }
        }
    }

    // CHOICE
    else if (category == 6)
    {
        for (i = 0; i < DICE_COUNT; i++)
        {
            score += dice[i];
        }
    }

    // FOUR OF A KIND
    else if (category == 7)
    {
        int n;

        for (n = 1; n <= 6; n++)
        {
            if (count_number(dice, n) >= 4)
            {
                for (i = 0; i < DICE_COUNT; i++)
                {
                    score += dice[i];
                }

                break;
            }
        }
    }

    // FULL HOUSE
    else if (category == 8)
    {
        int n;
        int three = 0;
        int two = 0;

        for (n = 1; n <= 6; n++)
        {
            int count = count_number(dice, n);

            if (count == 3)
            {
                three = 1;
            }

            if (count == 2)
            {
                two = 1;
            }
        }

        if (three == 1 && two == 1)
        {
            score = 25;
        }
    }

    // SMALL STRAIGHT
    else if (category == 9)
    {
        int start;

        for (start = 1; start <= 3; start++)
        {
            int found = 1;
            int n;

            for (n = start; n < start + 4; n++)
            {
                if (count_number(dice, n) == 0)
                {
                    found = 0;
                }
            }

            if (found == 1)
            {
                score = 15;
                break;
            }
        }
    }

    // LARGE STRAIGHT
    else if (category == 10)
    {
        int one = count_number(dice, 1);
        int two = count_number(dice, 2);
        int three = count_number(dice, 3);
        int four = count_number(dice, 4);
        int five = count_number(dice, 5);
        int six = count_number(dice, 6);

        if ((one == 1 && two == 1 && three == 1 &&
             four == 1 && five == 1) ||
            (two == 1 && three == 1 && four == 1 &&
             five == 1 && six == 1))
        {
            score = 30;
        }
    }

    // YACHT
    else if (category == 11)
    {
        if (count_number(dice, dice[0]) == 5)
        {
            score = 50;
        }
    }

    return score;
}

// 得点表を表示
void show_score(int score[PLAYER_COUNT][CATEGORY_COUNT])
{
    int i;

    printf("\n========================================\n");
    printf("                SCORE\n");
    printf("========================================\n");

    printf("%-20s %6s %6s\n",
           "CATEGORY", "P1", "P2");

    for (i = 0; i < CATEGORY_COUNT; i++)
    {
        printf("%-20s", category_names[i]);

        if (score[0][i] == -1)
        {
            printf("%6s", "--");
        }
        else
        {
            printf("%6d", score[0][i]);
        }

        if (score[1][i] == -1)
        {
            printf("%6s", "--");
        }
        else
        {
            printf("%6d", score[1][i]);
        }

        printf("\n");
    }

    printf("========================================\n");
}

// 残すサイコロを選択
void select_dice(int keep[])
{
    int number;
    int i;

    // 一度全部解除
    for (i = 0; i < DICE_COUNT; i++)
    {
        keep[i] = 0;
    }

    printf("\n残したいサイコロの番号を入力してください。\n");
    printf("例：1 3 5\n");
    printf("全部振り直す場合：0\n");
    printf("> ");

    while (1)
    {
        scanf_s("%d", &number);

        if (number == 0)
        {
            break;
        }

        if (number >= 1 && number <= DICE_COUNT)
        {
            keep[number - 1] = 1;
        }
        else
        {
            printf("1～5を入力してください。\n");
        }

        if (getchar() == '\n')
        {
            break;
        }
    }
}

// 使用するカテゴリを選択
int select_category(int score[], int dice[])
{
    int category;
    int i;

    while (1)
    {
        printf("\nカテゴリを選択してください。\n\n");

        for (i = 0; i < CATEGORY_COUNT; i++)
        {
            if (score[i] == -1)
            {
                printf("%2d : %-20s %2d点\n",
                       i + 1,
                       category_names[i],
                       calculate_score(dice, i));
            }
        }

        printf("\n番号：");
        scanf_s("%d", &category);

        category--;

        if (category >= 0 &&
            category < CATEGORY_COUNT &&
            score[category] == -1)
        {
            return category;
        }

        printf("そのカテゴリは選択できません。\n");
    }
}

// 1人分のターン
void play_turn(int player, int turn,
               int score[PLAYER_COUNT][CATEGORY_COUNT])
{
    int dice[DICE_COUNT];
    int keep[DICE_COUNT] = { 0 };
    int roll;
    int category;

    printf("\n\n");
    printf("========================================\n");
    printf("      第%dターン  PLAYER %d\n",
           turn, player + 1);
    printf("========================================\n");

    // 3回まで振る
    for (roll = 1; roll <= 3; roll++)
    {
        roll_dice(dice, keep);

        printf("\n%d回目のロール\n", roll);

        show_dice(dice);

        if (roll < 3)
        {
            select_dice(keep);
        }
    }

    // カテゴリ選択
    category = select_category(score[player], dice);

    score[player][category] =
        calculate_score(dice, category);

    printf("\nPLAYER %d は %s を選択しました。\n",
           player + 1,
           category_names[category]);

    printf("獲得点数：%d点\n",
           score[player][category]);
}

// メイン
int main(void)
{
    int score[PLAYER_COUNT][CATEGORY_COUNT];
    int total[PLAYER_COUNT] = { 0, 0 };

    int player;
    int turn;
    int category;

    // 乱数の初期化
    srand((unsigned int)time(NULL));

    // 全カテゴリを未使用にする
    for (player = 0; player < PLAYER_COUNT; player++)
    {
        for (category = 0;
             category < CATEGORY_COUNT;
             category++)
        {
            score[player][category] = -1;
        }
    }

    printf("========================================\n");
    printf("              YACHT GAME\n");
    printf("             2 PLAYER MATCH\n");
    printf("========================================\n");

    printf("\nPLAYER 1 と PLAYER 2 の対戦です。\n");
    printf("各プレイヤー12ターンです。\n");

    // 12ターン
    for (turn = 1; turn <= CATEGORY_COUNT; turn++)
    {
        // PLAYER 1
        play_turn(0, turn, score);

        show_score(score);

        printf("\nPLAYER 2 の番です。\n");
        printf("Enterキーを押してください。");

        while (getchar() != '\n')
        {
        }

        getchar();

        // PLAYER 2
        play_turn(1, turn, score);

        show_score(score);

        if (turn < CATEGORY_COUNT)
        {
            printf("\n次のターンへ進みます。\n");
            printf("Enterキーを押してください。");

            while (getchar() != '\n')
            {
            }

            getchar();
        }
    }

    // 最終得点
    for (player = 0; player < PLAYER_COUNT; player++)
    {
        for (category = 0;
             category < CATEGORY_COUNT;
             category++)
        {
            total[player] += score[player][category];
        }
    }

    // 結果表示
    printf("\n\n");
    printf("========================================\n");
    printf("               GAME OVER\n");
    printf("========================================\n");

    printf("\nPLAYER 1：%d点\n", total[0]);
    printf("PLAYER 2：%d点\n", total[1]);

    if (total[0] > total[1])
    {
        printf("\n★ PLAYER 1 WIN! ★\n");
    }
    else if (total[1] > total[0])
    {
        printf("\n★ PLAYER 2 WIN! ★\n");
    }
    else
    {
        printf("\n★ DRAW! ★\n");
    }

    return 0;
}
