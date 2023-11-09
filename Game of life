#include <ncurses.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#define ROWS 25
#define COLS 80
#define INTERVAL 50

int **dynMem(int rows, int cols, int inputFlag);
void freeAll(int **a, int rows);
int input(int **a, const int *rows, const int *cols);
void output(int **a, int rows, int cols);
int countNeighbours(int **matrix, int row, int col);
int getState(int **matrix, int row, int col);
char switcher(int **a, int row, int col);
void looper(int **matrixOld, int **matrixNew, int speed);
void controls(char key, int *speed);
void cursed(int **matrixOld, int **matrixNew);

int main(void) {
    int exitStatus = 0;
    int **matrixOld = dynMem(ROWS, COLS, 1);
    if (matrixOld) {
        int **matrixNew = dynMem(ROWS, COLS, 0);
        if (matrixNew) {
            if (freopen("/dev/tty", "r", stdin) != NULL) {
                cursed(matrixOld, matrixNew);
            }
            freeAll(matrixNew, ROWS);
        } else {
            exitStatus = 1;
        }
        freeAll(matrixOld, ROWS);
    } else {
        exitStatus = 1;
    }
    return exitStatus;
}

void cursed(int **matrixOld, int **matrixNew) {
    initscr();
    cbreak();
    noecho();
    nodelay(stdscr, TRUE);
    looper(matrixOld, matrixNew, INTERVAL * 5);
    endwin();
}

void looper(int **matrixOld, int **matrixNew, int speed) {
    int newState, curSpeed = speed;
    int key = getch();
    controls(key, &curSpeed);
    if (key == 'q') {
        return;
    }
    for (int row = 0; row < ROWS; row++) {
        for (int col = 0; col < COLS; col++) {
            newState = getState(matrixOld, row, col);
            matrixNew[row][col] = newState;
        }
    }
    output(matrixNew, ROWS, COLS);
    printw(
        "\n\nCurrent delay between generations is %d ms.\nPress '+' to increase it or '-' to decrease it by "
        "%d ms, 'q' to quit.\n",
        speed, INTERVAL);
    refresh();
    napms(curSpeed);
    clear();
    looper(matrixNew, matrixOld, curSpeed);
}

int countNeighbours(int **matrix, int row, int col) {
    int alive = 0;
    for (int i = row - 1; i <= row + 1; i++) {
        for (int j = col - 1; j <= col + 1; j++) {
            if (matrix[(ROWS + i) % ROWS][(COLS + j) % COLS]) {
                alive++;
            }
        }
    }
    if (matrix[row][col] && alive) {
        alive -= 1;
    }
    return alive;
}

int getState(int **matrix, int row, int col) {
    int curState = matrix[row][col], newState;
    int neighbours = countNeighbours(matrix, row, col);
    if (curState == 0 && neighbours == 3) {
        newState = 1;
    } else if (curState == 1 && (neighbours < 2 || neighbours > 3)) {
        newState = 0;
    } else {
        newState = curState;
    }
    return newState;
}

int **dynMem(int rows, int cols, int inputFlag) {
    int **ptrArray = calloc(rows, sizeof(int *));
    if (ptrArray) {
        for (int i = 0; i < rows; i++) {
            ptrArray[i] = calloc(cols, sizeof(int));
            if (ptrArray[i] == NULL) {
                ptrArray = NULL;
                break;
            }
        }
    } else {
        ptrArray = NULL;
    }
    if (inputFlag && input(ptrArray, &rows, &cols)) {
        ptrArray = NULL;
    }
    return ptrArray;
}

void freeAll(int **a, int rows) {
    for (int i = 0; i < rows; i++) {
        free(a[i]);
    }
    free(a);
}

int input(int **a, const int *rows, const int *cols) {
    int buffer;
    for (int row = 0; row < *rows; row++) {
        for (int col = 0; col < *cols; col++) {
            if (!scanf("%d", &buffer)) {
                return 1;
            }
            a[row][col] = buffer;
        }
    }
    return 0;
}

void output(int **a, int rows, int cols) {
    for (int row = 0; row < rows; row++) {
        for (int col = 0; col < cols; col++) {
            printw("%c", switcher(a, row, col));
        }
        if (row != rows - 1) {
            printw("\n");
        }
    }
}

char switcher(int **a, int row, int col) {
    char symbol;
    switch (a[row][col]) {
        case 0:
            symbol = ' ';
            break;

        case 1:
            symbol = '#';
            break;

        default:
            symbol = '?';
            break;
    }
    return symbol;
}

void controls(char key, int *speed) {
    switch (key) {
        case '-':
            if (*speed - INTERVAL >= INTERVAL) {
                *speed -= INTERVAL;
            }
            break;

        case '+':
            *speed += INTERVAL;
            break;

        default:
            break;
    }
}
