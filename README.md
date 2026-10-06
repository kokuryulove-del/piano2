# piano2
/*******************************/

/*  キーボードピアノ（Ubuntu版） */

/*******************************/

#include <stdio.h>

#include <stdlib.h>

#include <unistd.h>

#include <termios.h>

#include <sys/select.h>

#include <math.h>

#define PI 3.14159265358979323846

/* キーボード入力を非カノニカルモードにする */

void set_terminal(struct termios *old)

{

    struct termios new;

    tcgetattr(STDIN_FILENO, old);

    new = *old;

    new.c_lflag &= ~(ICANON | ECHO);

    new.c_cc[VMIN] = 0;

    new.c_cc[VTIME] = 0;

    tcsetattr(STDIN_FILENO, TCSANOW, &new);

}

/* 元の端末設定に戻す */

void reset_terminal(struct termios *old)

{

    tcsetattr(STDIN_FILENO, TCSANOW, old);

}

/* キーが押されているか確認 */

int kbhit(void)

{

    struct timeval tv = {0, 0};

    fd_set fds;

    FD_ZERO(&fds);

    FD_SET(STDIN_FILENO, &fds);

    return select(STDIN_FILENO + 1, &fds, NULL, NULL, &tv);

}

/* 音を鳴らす */

void beep(int Hz, int ms)

{

    char command[100];

    if (Hz <= 0)

        return;

    sprintf(command,

            "play -q -n synth %.3f sine %d >/dev/null 2>&1",

            ms / 1000.0, Hz);

    system(command);

}

int main(void)

{

    int n, nO = -1;

    int Hz;

    struct termios old;

    system("clear");

    printf("\n");

    printf(" |~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~|\n");

    printf(" |              キーボードピアノ（Ubuntu版）                 |\n");

    printf(" |          ※Shiftキーを押すと１オクターブ上がります          |\n");

    printf(" |          ※長押しはできません                               |\n");

    printf(" |                                                             |\n");

    printf(" *:._.:*~*:._.:*~*:._.:*~*:._.:*~*:._.:*~*:._.:*~*:._.:*~*:._.:*\n");

    printf("  |                                                           | \n");

    printf("  |----,----,----,----,----,----,----,----,----,----,----,----| \n");

    printf("  |ソ＃|ラ＃|    |ド＃|レ＃|    |ﾌｧ＃|ソ＃|ラ＃|    |ド＃|レ＃| \n");

    printf("  | Ａ | Ｓ | Ｄ | Ｆ | Ｇ | Ｈ | Ｊ | Ｋ | Ｌ | ； | ： | ］ | \n");

    printf("  '----'----'----'----'----'----'----'----'----'----'----'----' \n");

    printf("    | ラ | シ | ド | レ | ミ | ﾌｧ | ソ | ラ | シ | ド | レ |    \n");

    printf("    | Ｚ | Ｘ | Ｃ | Ｖ | Ｂ | Ｎ | Ｍ | ， | ． | ／ | ＼ |    \n");

    printf("    '----'----'----'----'----'----'----'----'----'----'----'    \n");

    /* キーボード設定 */

    set_terminal(&old);

    while (1)

    {

        /* キーが押されていない */

        if (kbhit() == 0)

        {

            n = -1;

            usleep(1000);

            continue;

        }

        /* キーを取得 */

        nO = n;

        read(STDIN_FILENO, &n, 1);

        /* ESCで終了 */

        if (n == 27)

            break;

        /* 同じキーなら無視 */

        if (n == nO)

            continue;

        switch (n)

        {

            /* 黒鍵 */

            case 'a': Hz = 208; break;

            case 's': Hz = 233; break;

            case 'f': Hz = 277; break;

            case 'g': Hz = 311; break;

            case 'j': Hz = 370; break;

            case 'k': Hz = 415; break;

            case 'l': Hz = 466; break;

            case ':': Hz = 554; break;

            case ']': Hz = 622; break;

            /* 白鍵 */

            case 'z': Hz = 220; break;

            case 'x': Hz = 247; break;

            case 'c': Hz = 262; break;

            case 'v': Hz = 294; break;

            case 'b': Hz = 330; break;

            case 'n': Hz = 349; break;

            case 'm': Hz = 392; break;

            case ',': Hz = 440; break;

            case '.': Hz = 494; break;

            case '/': Hz = 523; break;

            case '\\': Hz = 587; break;

            /* Shift */

            case 'A': Hz = 415; break;

            case 'S': Hz = 466; break;

            case 'F': Hz = 554; break;

            case 'G': Hz = 622; break;

            case 'H': Hz = 740; break;

            case 'J': Hz = 831; break;

            case 'K': Hz = 932; break;

            case '+': Hz = 1109; break;

            case '}': Hz = 1245; break;

            case 'Z': Hz = 440; break;

            case 'X': Hz = 494; break;

            case 'C': Hz = 523; break;

            case 'V': Hz = 587; break;

            case 'B': Hz = 659; break;

            case 'N': Hz = 698; break;

            case 'M': Hz = 784; break;

            case '<': Hz = 880; break;

            case '>': Hz = 988; break;

            case '?': Hz = 1047; break;

            case '_': Hz = 1175; break;

            default:

                Hz = 0;

        }

        beep(Hz, 200);

    }

    /* 元の端末設定に戻す */

    reset_terminal(&old);

    printf("\n終了しました。\n");

    return 0;

}



sudo apt update
sudo apt install sox
gcc piano.c -o piano -lm