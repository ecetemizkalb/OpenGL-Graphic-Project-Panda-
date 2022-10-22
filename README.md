# OpenGL Graphic Project(Panda)
#ifdef _MSC_VER
#define _CRT_SECURE_NO_WARNINGS
#endif
#include <GL/glut.h>
#include <stdio.h>
#include <stdlib.h>
#include <math.h>
#include <string.h>
#include <stdarg.h>
#include <time.h>

#define WINDOW_WIDTH  900
#define WINDOW_HEIGHT 600

#define TIMER_PERIOD  20 // Period for the timer.
#define TIMER_ON       1 // 0:disable timer, 1:enable timer
#define D2R 0.0174532

/* Global Variables for Template File */
#define  START 0
#define  RUN   1

double  cx = 0, cy = 0,//cloud x and y
R = 0, G = 0, B = 0; //Panda colors

int x, y, direction = 0, MODE = 0, GAME = 0, px = 0, py = 0;
bool up = false, down = false, right = false, left = false;
int  winWidth, winHeight; // current Window width and height

//
// to draw circle, center at (x,y)
// radius r
//
void circle(int x, int y, int r)
{
#define PI 3.1415
    float angle;
    glBegin(GL_POLYGON);
    for (int i = 0; i < 100; i++)
    {
        angle = 2 * PI * i / 100;
        glVertex2f(x + r * cos(angle), y + r * sin(angle));
    }
    glEnd();
}

void circle_wire(int x, int y, int r)
{
#define PI 3.1415
    float angle;

    glBegin(GL_LINE_LOOP);
    for (int i = 0; i < 100; i++)
    {
        angle = 2 * PI * i / 100;
        glVertex2f(x + r * cos(angle), y + r * sin(angle));
    }
    glEnd();
}

void print(int x, int y, const char* string, void* font)
{
    int len, i;

    glRasterPos2f(x, y);
    len = (int)strlen(string);
    for (i = 0; i < len; i++)
    {
        glutBitmapCharacter(font, string[i]);
    }
}

// display text with variables.
// vprint(-winWidth / 2 + 10, winHeight / 2 - 20, GLUT_BITMAP_8_BY_13, "ERROR: %d", numClicks);
void vprint(int x, int y, void* font, const char* string, ...)
{
    va_list ap;
    va_start(ap, string);
    char str[1024];
    vsprintf_s(str, string, ap);
    va_end(ap);

    int len, i;
    glRasterPos2f(x, y);
    len = (int)strlen(str);
    for (i = 0; i < len; i++)
    {
        glutBitmapCharacter(font, str[i]);
    }
}

// vprint2(-50, 0, 0.35, "00:%02d", timeCounter);
void vprint2(int x, int y, float size, const char* string, ...) {
    va_list ap;
    va_start(ap, string);
    char str[1024];
    vsprintf_s(str, string, ap);
    va_end(ap);
    glPushMatrix();
    glTranslatef(x, y, 0);
    glScalef(size, size, 1);

    int len, i;
    len = (int)strlen(str);
    for (i = 0; i < len; i++)
    {
        glutStrokeCharacter(GLUT_STROKE_ROMAN, str[i]);
    }
    glPopMatrix();
}

//
// To display onto window using OpenGL commands
//

void displaybackground()
{
    glColor3f(0, 0, 0);
    vprint(-360, 250, GLUT_BITMAP_8_BY_13, "Ece Temizkalb");
    vprint(-80, 200, GLUT_BITMAP_9_BY_15, " - Flying Panda? -");
    vprint(-360, -230, GLUT_BITMAP_8_BY_13, "Press <F1> to switch the mode and <arrow keys> control the Panda.");
    vprint(-360, -245, GLUT_BITMAP_8_BY_13, "Press <F2> to change the color of the Panda.");

    switch (direction)
    {
    case 0: vprint(-360, -215, GLUT_BITMAP_8_BY_13, "Direction: NOT SET");
        break;
    case 1: vprint(-360, -215, GLUT_BITMAP_8_BY_13, "Direction: UP");
        break;
    case 2: vprint(-360, -215, GLUT_BITMAP_8_BY_13, "Direction: DOWN");
        break;
    case 3: vprint(-360, -215, GLUT_BITMAP_8_BY_13, "Direction: LEFT");
        break;
    case 4: vprint(-360, -215, GLUT_BITMAP_8_BY_13, "Direction: RIGHT");
        break;
    }
   

    switch (MODE)
    {
    case 0:
        vprint(-360, -200, GLUT_BITMAP_8_BY_13, "Mode: Autonomous");
        break;
    default:
        vprint(-360, -200, GLUT_BITMAP_8_BY_13, "Mode: Manual");
    }

    glColor3ub(100, 173, 175);
    glRectf(-360, -190, 360, 190);

    glColor3ub(26, 81, 3);
    glRectf(-360, -190, 360, -155);

    //sun
    glColor4f(1, 1, 1, 0.2);
    circle(-180, 120, 40);
    glColor4f(1, 1, 1, 0.1);
    circle(-180, 120, 80);
    glColor3ub(255, 255, 66);
    circle(-180, 120, 25);


    //clouds
    glColor3f(1, 1, 1);
    circle(cx, cy + 110, 30);
    circle(cx + 20, cy + 90, 20);
    circle(cx + 30, cy + 120, 20);
    circle(cx + 50, cy + 105, 25);

    circle(cx - 210, cy + 80, 25);
    circle(cx - 230, cy + 60, 20);
    circle(cx - 240, cy + 90, 20);
    circle(cx - 260, cy + 75, 25);

    //bamboos
    glColor3ub(24, 61, 2);
    glRectf(-340, -190, -320, 170);
    glRectf(-300, -190, -290, 150);
    glRectf(-260, -190, -250, 160);
    glRectf(+330, -190, +320, 170);
    glRectf(+300, -190, +280, 155);

    if(GAME==START)
     vprint(-150, 0, GLUT_BITMAP_9_BY_15, "CLICK TO START/RELOCATE THE PANDA");
    
}

void displaypanda()
{
    //outlines of the panda.
    glColor3f(0, 0, 0);
    circle(px, py, 51);
    circle(px + 27, py - 40, 14);
    circle(px - 27, py - 40, 14);

    glColor3ub(R, G, B);
    circle(px, py, 50);
    circle(px+27, py-40, 13);
    circle(px-27, py-40, 13);
    glColor3f(1, 1, 1);
    circle(px, py, 30);
    circle(px - 15, py + 23, 9);
    circle(px + 15, py + 23, 9);
    glColor3ub(R, G, B);
    circle(px+12, py, 10);
    circle(px-12, py, 10);
    glColor3f(1, 1, 1);
    circle(px+12, py, 3);
    circle(px-12, py, 3);
    glColor3ub(R, G, B);
    glBegin(GL_TRIANGLES);
    glVertex2f(px, py - 15);
    glVertex2f(px - 7, py - 10);
    glVertex2f(px + 7, py - 10);
    glEnd();
    glLineWidth(2);
    glBegin(GL_LINES);
    glVertex2f(px, py - 15);
    glVertex2f(px - 6, py - 18);
    glVertex2f(px, py - 15);
    glVertex2f(px + 6, py - 18);
    glEnd();

    
}

void display() {
    //
    // clear window to black
    //
    glClearColor(1, 1, 1, 0);
    glClear(GL_COLOR_BUFFER_BIT);
    
    displaybackground();


    if (GAME == RUN)
        displaypanda();


    glutSwapBuffers();
}

//
// key function for ASCII charachters like ESC, a,b,c..,A,B,..Z
//
void onKeyDown(unsigned char key, int x, int y)
{
    // exit when ESC is pressed.
    if (key == 27)
        exit(0);
  
    // to refresh the window it calls display() function
    glutPostRedisplay();
}

void onKeyUp(unsigned char key, int x, int y)
{
    // exit when ESC is pressed.
    if (key == 27)
        exit(0);

    // to refresh the window it calls display() function
    glutPostRedisplay();
}

//
// Special Key like GLUT_KEY_F1, F2, F3,...
// Arrow Keys, GLUT_KEY_UP, GLUT_KEY_DOWN, GLUT_KEY_RIGHT, GLUT_KEY_RIGHT
//
void onSpecialKeyDown(int key, int x, int y)
{
       
    // Write your codes here.
    switch (key) {
    case GLUT_KEY_UP: up = true; 
        direction = 1;
        if (MODE == 1 && py <= 140)
            py+=2;
              break;
    case GLUT_KEY_DOWN: down = true;
        direction = 2;
        if (MODE == 1 && py >= -135)
            py-= 2; break;
    case GLUT_KEY_LEFT: left = true;
        direction = 3;
        if (MODE == 1 && px >= -310)
            px-= 2; break;
    case GLUT_KEY_RIGHT: right = true;
        direction = 4;
        if (MODE == 1 && py <= 310)
            px+= 2; break;
    }

    if (key == GLUT_KEY_F1)
        if (MODE == 0)
            MODE = 1;
        else
            MODE = 0;


    //changing the color of the panda.
    srand(time(NULL));
    if (key == GLUT_KEY_F2)
    {
        R = rand() % 256;
        G = rand() % 256;
        B = rand() % 256;
    }

    // to refresh the window it calls display() function
    glutPostRedisplay();
}

//
// Special Key like GLUT_KEY_F1, F2, F3,...
// Arrow Keys, GLUT_KEY_UP, GLUT_KEY_DOWN, GLUT_KEY_RIGHT, GLUT_KEY_RIGHT
//
void onSpecialKeyUp(int key, int x, int y)
{
    // Write your codes here.
    switch (key) {
    case GLUT_KEY_UP: up = false; break;
    case GLUT_KEY_DOWN: down = false; break;
    case GLUT_KEY_LEFT: left = false; break;
    case GLUT_KEY_RIGHT: right = false; break;
    }

    // to refresh the window it calls display() function
    glutPostRedisplay();
}

//
// When a click occurs in the window,
// It provides which button
// buttons : GLUT_LEFT_BUTTON , GLUT_RIGHT_BUTTON
// states  : GLUT_UP , GLUT_DOWN
// x, y is the coordinate of the point that mouse clicked.
//
void onClick(int button, int stat, int x, int y)
{
    // Write your codes here.
    if (button == GLUT_LEFT_BUTTON && stat == GLUT_DOWN && x > 134 && y > 156 && x < 763 && y < 440)
        if (GAME == START)
        {
            GAME = RUN;
            px = x - winWidth / 2;
            py = winHeight / 2 - y;
        }
        else
        {
            if (GAME == RUN)
            {
                px = x - winWidth / 2;
                py = winHeight / 2 - y;
            }

        }
        


    // to refresh the window it calls display() function
    glutPostRedisplay();
}

//
// This function is called when the window size changes.
// w : is the new width of the window in pixels.
// h : is the new height of the window in pixels.
//
void onResize(int w, int h)
{
    winWidth = w;
    winHeight = h;
    glViewport(0, 0, w, h);
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    glOrtho(-w / 2, w / 2, -h / 2, h / 2, -1, 1);
    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();
    display(); // refresh window.
}

void onMoveDown(int x, int y) {
    // Write your codes here.



    // to refresh the window it calls display() function   
    glutPostRedisplay();
}

// GLUT to OpenGL coordinate conversion:
//   x2 = x1 - winWidth / 2
//   y2 = winHeight / 2 - y1
void onMove(int x, int y) {
    // Write your codes here.



    // to refresh the window it calls display() function
    glutPostRedisplay();
}

#if TIMER_ON == 1
void onTimer(int v) {

    glutTimerFunc(TIMER_PERIOD, onTimer, 0);
    // Write your codes here.
    
    if (MODE == 0)
    {
        switch (direction)
        {
        case 1:if (py <= 140)
                     py++;
                 if (py == 140)
                     direction = 2;
                  break;
        case 2:if (py >= -135)
                    py--; 
            if (py == -135)
                direction = 1;
                    break;
        case 3:if (px >= -310)
            px--; 
            if (px == -310)
                direction = 4; 
            break;
        case 4:if (px <= 310)
            px++; 
            if (px == 310)
            direction = 3;
            break;
        }
 
    }

    if (cx == 646)
        cx = -430;
    else
        cx += 0.5;
    

    // to refresh the window it calls display() function
    glutPostRedisplay(); // display()

}
#endif

void Init() {

    // Smoothing shapes
    glEnable(GL_BLEND);
    glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);

}

void main(int argc, char* argv[]) {
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_RGB | GLUT_DOUBLE);
    glutInitWindowSize(WINDOW_WIDTH, WINDOW_HEIGHT);
    glutInitWindowPosition(600, 300);
    glutCreateWindow("E.Temizkalb-OpenGL Graphic Project(FlyingPanda)");

    glutDisplayFunc(display);
    glutReshapeFunc(onResize);

    //
    // keyboard registration
    //
    glutKeyboardFunc(onKeyDown);
    glutSpecialFunc(onSpecialKeyDown);

    glutKeyboardUpFunc(onKeyUp);
    glutSpecialUpFunc(onSpecialKeyUp);

    //
    // mouse registration
    //
    glutMouseFunc(onClick);
    glutMotionFunc(onMoveDown);
    glutPassiveMotionFunc(onMove);

#if  TIMER_ON == 1
    // timer event
    glutTimerFunc(TIMER_PERIOD, onTimer, 0);
#endif

    Init();

    glutMainLoop();
}
