#include <stdio.h>
#include <string.h>

#define PLAYER_COUNT 4
#define NAME_LEN 20
#define MAX_ROUNDS 100

typedef struct {
    char name[NAME_LEN];
    int score;
} Player;

typedef struct {
    int round;
    char winner[NAME_LEN];
    char loser[NAME_LEN];
    int isSelfDraw;
    int fan; // 番數
} RoundRecord;

Player players[PLAYER_COUNT];
RoundRecord records[MAX_ROUNDS];
int roundCount = 0;

// 找玩家 index
int findPlayerIndex(char *name) {
    for (int i = 0; i < PLAYER_COUNT; i++) {
        if (strcmp(players[i].name, name) == 0) {
            return i;
        }
    }
    return -1;
}

// 顯示目前分數
void showScores() {
    printf("\n目前分數：\n");
    for (int i = 0; i < PLAYER_COUNT; i++) {
        printf("%s：%d 分\n", players[i].name, players[i].score);
    }
    printf("\n");
}

// 寫入紀錄到檔案
void saveToFile() {
    FILE *fp = fopen("record.txt", "w");
    if (!fp) {
        printf("無法開啟檔案。\n");
        return;
    }
    fprintf(fp, "麻將對局紀錄\n\n");
    for (int i = 0; i < roundCount; i++) {
        RoundRecord r = records[i];
        fprintf(fp, "第 %d 局：%s 勝，", r.round, r.winner);
        if (r.isSelfDraw) {
            fprintf(fp, "自摸，番數：%d\n", r.fan);
        } else {
            fprintf(fp, "放槍者：%s，番數：%d\n", r.loser, r.fan);
        }
    }
    fprintf(fp, "\n最終分數：\n");
    for (int i = 0; i < PLAYER_COUNT; i++) {
        fprintf(fp, "%s：%d 分\n", players[i].name, players[i].score);
    }
    fclose(fp);
    printf("紀錄已儲存到 record.txt\n");
}

int main() {
    printf("台灣麻雀枱數計算系統\n請輸入 4 位玩家名字：\n");
    for (int i = 0; i < PLAYER_COUNT; i++) {
        printf("玩家 %d 名字：", i + 1);
        scanf("%s", players[i].name);
        players[i].score = 0;
    }

    while (1) {
        showScores();
        char winName[NAME_LEN], loseName[NAME_LEN];
        int selfDraw, fan;

        printf("輸入贏家名字（exit 結束）：");
        scanf("%s", winName);
        if (strcmp(winName, "exit") == 0) break;

        int winIdx = findPlayerIndex(winName);
        if (winIdx == -1) {
            printf("找不到玩家。\n");
            continue;
        }

        printf("是否自摸？(1: 是, 0: 否)：");
        scanf("%d", &selfDraw);
        printf("請輸入番數（例如混一色+自摸=4）：");
        scanf("%d", &fan);

        int loseIdx = -1;
        if (!selfDraw) {
            printf("請輸入放槍者名字：");
            scanf("%s", loseName);
            loseIdx = findPlayerIndex(loseName);
            if (loseIdx == -1 || loseIdx == winIdx) {
                printf("放槍者無效。\n");
                continue;
            }
        }

        // 更新分數
        if (selfDraw) {
            for (int i = 0; i < PLAYER_COUNT; i++) {
                if (i != winIdx) {
                    players[i].score -= fan;
                    players[winIdx].score += fan;
                }
            }
        } else {
            players[winIdx].score += fan;
            players[loseIdx].score -= fan;
        }

        // 紀錄此局
        RoundRecord r;
        r.round = ++roundCount;
        strcpy(r.winner, winName);
        if (!selfDraw) strcpy(r.loser, loseName);
        r.isSelfDraw = selfDraw;
        r.fan = fan;
        records[roundCount - 1] = r;
    }

    // 結束儲存
    saveToFile();
    return 0;
}
