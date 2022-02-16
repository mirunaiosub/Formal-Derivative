#include <fstream>
#include <iostream>
#include <graphics.h>
#include <winbgim.h>
#include <cstring>
#include <stdio.h>
#include <stdlib.h>
#include <windows.h>
#include <mmsystem.h>
using namespace std;

ifstream fin("derivative.in");
ofstream fout("derivative.out");

char infix[100], postfixat[100], derivata1[250],derivata2[250];

struct nod
{
    char info;
    nod* urm;
};
struct arbore {
    char info[10];
    arbore* st, * dr;
};

struct nodarbore {
    arbore* info;
    nodarbore* urm;
};


struct stiva
{
    nod* varf;
    int nrElemente;
};

struct stivaarbore {
    nodarbore* varf;
    int nrElemente;
};

struct coada
{
    nod* prim;
    nod* ultim;
    int nrElemente;
};

coada postfix;

bool esteVida(stivaarbore S) {
    return S.nrElemente == 0;
}

void initializeazastivaarbore(stivaarbore& S) {
    S.varf = NULL;
    S.nrElemente = 0;
}

bool esteVida(stiva S)
{
    return S.nrElemente == 0;
}

bool esteVida(coada C)
{
    return C.nrElemente == 0;
}

void initstiva(stiva& S)
{
    S.varf = NULL;
    S.nrElemente = 0;
}

void initcoada(coada& C)
{
    C.prim = NULL;
    C.ultim = NULL;
    C.nrElemente = 0;
}

int nrNiveluri(arbore* a) {

    if (a == NULL)return 0;
    else {
        int n1 = nrNiveluri(a->st);
        int n2 = nrNiveluri(a->dr);
        return 1 + max(n1, n2);
    }
}

int nrColoane(arbore* a) {

    if (a == NULL)return 0;
    else {
        int n1 = nrColoane(a->st);
        int n2 = nrColoane(a->dr);
        return 1 + n1 + n2;
    }
}
// functie care verifica daca un element din coada notatiei postfix este operator (adaugarea elementelor arborelui in stiva
int esteoperator1(char element[]) { 

    if (strchr("+-*/^", element[0]) && element[1] == NULL) return 2;
    if (strchr("sctgl", element[0]) && strchr("iogtn", element[1]) && element[1] != NULL)
        return 1;
    return 0;
}

// adauga un element la stiva de arbori, in varf
void pusharbore(stivaarbore& S, arbore* element) {
    nodarbore* nod_nou;

    if (esteVida(S)) {
        S.nrElemente = 1;
        S.varf = new nodarbore;
        S.varf->info = element;
        S.varf->urm = NULL;
    }
    else {
        S.nrElemente++;
        nod_nou = new nodarbore;
        nod_nou->info = element;
        nod_nou->urm = S.varf;
        S.varf = nod_nou;
    }
}

// scoate elementul din varful stivei de arbori
void poparbore(stivaarbore& S) { 
    if (!esteVida(S)) {
        nodarbore* varf_nou;
        varf_nou = S.varf->urm;
        delete S.varf;
        S.varf = varf_nou;
        S.nrElemente--;
    }
}

void deseneazaNod(char elem[], int xc, int yc, int latime, int inaltime) {
    char arr[50];
    sprintf(arr, "%s", elem);
    setcolor(LIGHTMAGENTA);
    setfillstyle(SOLID_FILL, LIGHTMAGENTA);
    fillellipse(xc, yc, 23, 23);
    settextstyle(SANS_SERIF_FONT, HORIZ_DIR, 1);
    setbkcolor(LIGHTMAGENTA);
    setcolor(5);
    settextjustify(1, 1);
    settextstyle(3, 0, 2);
    outtextxy(xc, yc + 4, arr);
}

void deseneazaArbore(arbore* a, int niv, int stanga, int latime, int inaltime) {

    if (a != NULL) {
        int n1 = nrColoane(a->st);
        int yc = niv * inaltime - inaltime / 2;
        int xc = stanga + latime * n1 + latime / 2;

        if (a->st != NULL) {
            int xcs = stanga + latime * nrColoane(a->st->st) + latime / 2;
            setcolor(5);
            line(xc, yc, xcs, yc + inaltime);
        }

        if (a->dr != NULL) {
            int xcd = stanga + latime * (n1 + 1) + latime * nrColoane(a->dr->st) + latime / 2;
            setcolor(5);
            line(xc, yc, xcd, yc + inaltime);
        }
        deseneazaArbore(a->st, niv + 1, stanga, latime, inaltime);
        deseneazaArbore(a->dr, niv + 1, stanga + latime * (n1 + 1), latime, inaltime);
        deseneazaNod(a->info, xc, yc, latime, inaltime);
    }
}

//adaugam notatia postfixata in stiva de arbori
void adaugaLaArboreElement(coada C, stivaarbore& S) {
    int i = 0;
    arbore* arbore_nou;
    char sir[10];
    memset(sir, '\0', 10); // reinitializare sir de caractere

    while (C.prim != NULL) {
        if (C.prim->info == ',') {
            if (!(esteoperator1(sir))) { // operand - copiaza din sir adresa si o adauga la stiva ca fiind un nou arbore
                arbore_nou = new arbore;
                strcpy(arbore_nou->info, sir);
                arbore_nou->st = NULL;
                arbore_nou->dr = NULL;
                pusharbore(S, arbore_nou);
            }
            else if (esteoperator1(sir) == 2) { // +, -, *, /, ^, e - copiaza adresa din sir, adaugandu-i fiii la stanga si dreapta (elemente din stiva pe care le va sterge), adaugand-o in stiva ca arbore nou
                arbore_nou = new arbore;
                strcpy(arbore_nou->info, sir);
                arbore_nou->dr = S.varf->info;
                poparbore(S);
                arbore_nou->st = S.varf->info;
                poparbore(S);
                pusharbore(S, arbore_nou);
            }
            else { // este functie (sin, cos, tg, ctg, ln) - copiaza din sir adresa adaugandu i la stanga valoarea din stiva(pe care o va sterge), adaugand o in stiva ca arbore nou
                arbore_nou = new arbore;
                strcpy(arbore_nou->info, sir);
                arbore_nou->dr = NULL;
                arbore_nou->st = S.varf->info;
                poparbore(S);
                pusharbore(S, arbore_nou);
            }
               memset(sir, '\0', strlen(sir)); //reinitializare sir cu strlen(sir)
            i = 0;
        }
        else
            sir[i++] = C.prim->info; //daca nu este virgula, vom adauga elementul la sir
        C.prim = C.prim->urm;
    }

    if (esteoperator1(sir) == 2) { // +, -, *, /, ^, e - copiaza adresa din sir, adaugandu-i fiii la stanga si dreapta (elemente din stiva pe care le va sterge), adaugand-o in stiva ca arbore nou
        arbore_nou = new arbore;
        strcpy(arbore_nou->info, sir);
        arbore_nou->dr = S.varf->info;
        poparbore(S);
        arbore_nou->st = S.varf->info;
        poparbore(S);
        pusharbore(S, arbore_nou);
    }
    else { // este functie (sin, cos, tg, ctg, ln) - copiaza din sir adresa adaugandu i la stanga valoarea din stiva(pe care o va sterge), adaugand o in stiva ca arbore nou
        arbore_nou = new arbore;
        strcpy(arbore_nou->info, sir);
        arbore_nou->dr = NULL;
        arbore_nou->st = S.varf->info;
        poparbore(S);
        pusharbore(S, arbore_nou);
    }
}

//stergere element din stiva
void pop(stiva& S)
{
    if (!esteVida(S))
    {

        nod* varf_nou;
        varf_nou = S.varf->urm;
        delete S.varf;
        S.varf = varf_nou;
        S.nrElemente--;
    }
}

//adaugare element in stiva
void push(stiva& S, char element)
{
    nod* nod_nou;
    if (esteVida(S))
    {
        S.nrElemente = 1;
        S.varf = new nod;
        S.varf->info = element;
        S.varf->urm = NULL;
    }
    else
    {
        S.nrElemente++;
        nod_nou = new nod;
        nod_nou->info = element;
        nod_nou->urm = S.varf;
        S.varf = nod_nou;
    }
}

//adaugare element in coada
void incoada(coada& C, char element)
{
    nod* nod_nou;
    if (esteVida(C))
    {
        C.nrElemente = 1;
        C.prim = new nod;
        C.prim->info = element;
        C.prim->urm = NULL;
        C.ultim = C.prim;
    }
    else
    {
        C.nrElemente++;
        nod_nou = new nod;
        nod_nou->info = element;
        nod_nou->urm = NULL;
        C.ultim->urm = nod_nou;
        C.ultim = nod_nou;
    }
}
//prioritatea operatorilor - pt a scoate din stiva in caz de avem o paranteza deschisa la notatia infix si a adauga la postfix
int prioritate(char c) 
{
    switch (c)
    {
    case '-':
    case '+':
        return 1;
    case '*':
    case '/':
        return 2;
    case '^':
        return 3;
    case 's':
    case 'c':
    case 't':
    case 'g':
    case 'l':
    case 'e':
        return 4;
    case '(':
        return 5;
    case ')':
        return 6;
    }
}
//adaugare in notatia postfixata (din stiva)
void adaugainpostfix(coada& postfix, stiva& S) 
{
    if (S.varf->info == 's')
    {
        incoada(postfix, 's');
        incoada(postfix, 'i');
        incoada(postfix, 'n');
    }
    else
        if (S.varf->info == 'c')
    {
        incoada(postfix, 'c');
        incoada(postfix, 'o');
        incoada(postfix, 's');
    }
    else
        if (S.varf->info == 't')
    {
        incoada(postfix, 't');
        incoada(postfix, 'g');
    }
    else
        if (S.varf->info == 'g')
    {
        incoada(postfix, 'c');
        incoada(postfix, 't');
        incoada(postfix, 'g');
    }
    else
        if (S.varf->info == 'e')
    {
        incoada(postfix, 'l');
        incoada(postfix, 'n');
    }
    else
        incoada(postfix, S.varf->info);

    pop(S);
}
//adaugare din coada in stiva
void adaugainstiva(stiva& S, coada& postfix, char& x, int& i) 
{
    if (x == 's' && infix[i + 1] == 'i' && infix[i + 2] == 'n')
    {
        push(S, 's');
        i = i + 2;
    }
    else
        if (x == 'c' && infix[i + 1] == 'o' && infix[i + 2] == 's')
    {
        push(S, 'c');
        i = i + 2;
    }
    else
        if (x == 't' && infix[i + 1] == 'g')
    {
        push(S, 't');
        i = i + 1;
    }
    else
        if (x == 'c' && infix[i + 1] == 't' && infix[i + 2] == 'g')
    {
        push(S, 'g');
        i = i + 2;
    }
    else
        if (x == 'l' && infix[i + 1] == 'n')
    {
        push(S, 'e');
        i = i + 1;
    }
    else
        push(S, x);
}

//verificare daca un element din coada este operator - pentru a verifica daca in notatia infix avem operanzi pt a-i adauga direct in coada postfix
bool verifoperator(char infix[], int i) 
{
    if (strchr("+-*/^(),", infix[i])) return 1;
    if (strchr("sctl", infix[i]) && strchr("iogtn", infix[i + 1]))
        return 1;
    return 0;
}

//transformarea in forma postfixata
void InfixToPostfix(char infix[], coada& postfix) 
{
    int i, n = strlen(infix), j = 0;
    char x;
    stiva S;
    initstiva(S);

    for (i = j; i < n; i++) {
        x = infix[i];
        if (!verifoperator(infix, i)) // daca e operand: 1,2,3,x,n
            incoada(postfix, x);
        else {

            if (x != '(' && infix[i - 1] != ')' && !strchr("sctl", x))
                incoada(postfix, ',');

            if (x == ')') //daca e finalul unei paranteze scoatem din stiva tot ce se afla in paranteza
            {
                while (S.varf->info != '(') {
                    adaugainpostfix(postfix, S);
                    incoada(postfix, ',');
                }

                pop(S);
            }
            else //daca stiva mai are ca un singur element, paranteza deschisa, si x ul nu este paranteza inchisa(cea mai mare prioritate) atunci scoatem din stiva toata paranteza, adaugand in stiva
                {
                   while (!esteVida(S) && (S.varf->info != '(') && (prioritate(S.varf->info) >= prioritate(x)))
                        {
                            adaugainpostfix(postfix, S);
                            incoada(postfix, ',');
                        }
                    adaugainstiva(S, postfix, x, i); //daca e ( sau sin, cos, tg, ctg doar o adaugam in stiva
                }
        }
    }
    if (!verifoperator(infix, i - 1))
        incoada(postfix, ','); //verificam daca un operand a ramas in notatia infix

    while (S.nrElemente > 1) {
        adaugainpostfix(postfix, S); //adaugam la coada toate elementele ramase in stiva, fara ultimul element (deoarece se v a adauga o virgula automat)
        incoada(postfix, ',');
    }

    adaugainpostfix(postfix, S);

}

void afiseaza(coada C) { 
    nod* nod_curent;
    nod_curent = C.prim;
    int i = 0;

    while (nod_curent != NULL)
    {
        fout << nod_curent->info;
        postfixat[i++] = nod_curent->info;
        nod_curent = nod_curent->urm;

    }
}

//PENTRU MENIU
void titlu(int x, int y, char* n) { 
    settextstyle(1, 0, 6);
    setcolor(5);
    setbkcolor(15);
    setlinestyle(0, 0, 8);
    readimagefile("imagine.jpg", 0, 0, 1400, 1000);
    rectangle(3, 3, 1397, 100);
    setfillstyle(EMPTY_FILL, 2);
    outtextxy(x, y, n);
}

void butoane() { 
    rectangle(3, 100, 1400, 185);
    settextstyle(9, 0, 5);
    outtextxy(600, 123, "MENU");

    setlinestyle(0, 0, 3);
    rectangle(0, 185, 1200, 260);
    settextstyle(9, 0, 2);
    outtextxy(80, 215, "Info");

    rectangle(200, 185, 1000, 260);
    settextstyle(9, 0, 2);
    outtextxy(240, 215, "Keyboard");

    rectangle(400, 185, 800, 260);
    settextstyle(9, 0, 2);
    outtextxy(422, 215, "Derivative.in");

    rectangle(600, 185, 600, 260);
    settextstyle(9, 0, 2);
    outtextxy(650, 215, "Postfix");

    rectangle(800, 185, 400, 260);
    settextstyle(9, 0, 2);
    outtextxy(870, 215, "Tree");

    rectangle(1000, 185, 200, 260);
    settextstyle(9, 0, 2);
    outtextxy(1017, 215, "1st Derivative");

    rectangle(1200, 185, 0, 260);
    settextstyle(9, 0, 2);
    outtextxy(1212, 215, "2nd Derivative");
}


// pentru butoane
void click(int& ok) { 
    int coordx, coordy;
    clearmouseclick(WM_LBUTTONDOWN);
    //daca se da click pe mouse
    while (!ismouseclick(WM_LBUTTONDOWN))
    {
        coordx = mousex();
        coordy = mousey();
    }
 if (coordx > 0 && coordx < 200 && coordy>185 && coordy < 260) //INFO
      {

       PlaySound(TEXT("2w.wav"),NULL,SND_ASYNC);
        ok = 1;}
   else if (coordx > 200 && coordx < 400 && coordy>185 && coordy < 260) // KEYBOARD
      {
           PlaySound(TEXT("2w.wav"),NULL,SND_ASYNC);
       ok = 2;}
    else if (coordx > 400 && coordx < 600 && coordy>185 && coordy < 260) //FISIER
       {
            PlaySound(TEXT("2w.wav"),NULL,SND_ASYNC);
        ok = 3;}
    else if (coordx > 600 && coordx < 800 && coordy>185 && coordy < 260) //POSTFIX
        { PlaySound(TEXT("2w.wav"),NULL,SND_ASYNC);
            ok = 4;}
    else if (coordx > 800 && coordx < 1000 && coordy>185 && coordy < 260) //TREE
        { PlaySound(TEXT("2w.wav"),NULL,SND_ASYNC);
            ok = 5;}
    else if (coordx > 1000 && coordx < 1200 && coordy>185 && coordy < 260) //DERIVATIVE 1
        { PlaySound(TEXT("2w.wav"),NULL,SND_ASYNC);
            ok = 6;}
    else if (coordx > 1200 && coordx < 1400 && coordy>185 && coordy < 260) //DERIVATIVE 2
      {
           PlaySound(TEXT("2w.wav"),NULL,SND_ASYNC);
        ok = 7; }
}


arbore* simplificare(arbore* a) { 
    if (!strcmp(a->info, "*")) {
        if (!strcmp(a->st->info, "0") || !strcmp(a->dr->info, "0")) {
            strcpy(a->info, "0");
            a->st = NULL;
            a->dr = NULL;
        }
        else if (!strcmp(a->st->info, "1")) {
            a = a->dr;
        }
        else if (!strcmp(a->dr->info, "1")) {
            a = a->st;
        }
    }

    if (!strcmp(a->info, "/")) {
        if (!strcmp(a->st->info, "0")) {
            strcpy(a->info, "0");
            a->st = NULL;
            a->dr = NULL;
        }
        else if (!strcmp(a->dr->info, "1")) {
            a = a->st;

        }
    }

    if (!strcmp(a->info, "+")) {
        if (!strcmp(a->st->info, "0")) {
            a = a->dr;

        }
        else if (!strcmp(a->dr->info, "0")) {
            a = a->st;

        }
        else if (!strcmp(a->dr->info, "-")) {
            a->dr = a->dr->dr;
            strcpy(a->info, "-");
        }
    }

    if (!strcmp(a->info, "-")) {
        if (!strcmp(a->dr->info, "0")) {
            a = a->st;
        }
        else if (!strcmp(a->dr->info, "-")) {
            a->dr = a->dr->dr;
            strcpy(a->info, "+");
        }
    }

    if (!strcmp(a->info, "^")) {
        if (!strcmp(a->st->info, "0")) {
            strcpy(a->info, "0");
            a->st = NULL;
            a->dr = NULL;
        }
        else if (!strcmp(a->dr->info, "0")) {
            strcpy(a->info, "1");
            a->st = NULL;
            a->dr = NULL;
        }
        else if (!strcmp(a->st->info, "1")) {
            strcpy(a->info, "1");
            a->st = NULL;
            a->dr = NULL;
        }
        else if (!strcmp(a->dr->info, "1")) {
            a = a->st;
        }
    }

    if (!strcmp(a->info, "ln")) {
        if (!strcmp(a->st->info, "e")) {
            strcpy(a->info, "1");
            a->st = NULL;
            a->dr = NULL;
        }
    }
    return a;
}


arbore* deriveaza(arbore* a, arbore* d) { 

  arbore* a1, * a2, * a3, * a4, * a5, * a6, * a7, * a8;

    if (strchr("0123456789", a->info[0])) strcpy(d->info, "0"); // numere pozitive

    if (a->info[0] == 'x' || a->info[0] == 'n') strcpy(d->info, "1"); // este variabila

   
    if(a->info[0]=='+' && a->info[1]==NULL){
        a1=new arbore;
        a1->st=NULL;
        a1->dr=NULL;
        a2 = new arbore;
        a2->st = NULL;
        a2->dr = NULL;
        deriveaza(a->st, a1);
            a1=simplificare(a1);
            d->st=NULL;
            deriveaza(a->dr,a2);
            simplificare(a2);
            d->dr=a2;
            strcpy(d->info, a->info);
            d=simplificare(d);
    }
    if (a->info[0] == '*') { // derivam inmultirea
        a1 = new arbore;
        a1->st = NULL;
        a1->dr = NULL;
        a2 = new arbore;
        a2->st = NULL;
        a2->dr = NULL;
        a3 = new arbore;
        a3->st = NULL;
        a3->dr = NULL;
        a4 = new arbore;
        a4->st = NULL;
        a4->dr = NULL;
        deriveaza(a->st, a1);
        a1 = simplificare(a1);
        deriveaza(a->dr, a2);
        a2 = simplificare(a2);
        strcpy(a3->info, "*");
        a3->st = a1;
        a3->dr = a->dr;
        a3 = simplificare(a3);
        strcpy(a4->info, "*");
        a4->st = a->st;
        a4->dr = a2;
        a4 = simplificare(a4);
        strcpy(d->info, "+");
        d->st = a3;
        d->dr = a4;
        d = simplificare(d);
    }

    if (a->info[0] == '/') { // derivam impartirea
        a1 = new arbore;
        a1->st = NULL;
        a1->dr = NULL;
        a2 = new arbore;
        a2->st = NULL;
        a2->dr = NULL;
        a3 = new arbore;
        a3->st = NULL;
        a3->dr = NULL;
        a4 = new arbore;
        a4->st = NULL;
        a4->dr = NULL;
        a5 = new arbore;
        a5->st = NULL;
        a5->dr = NULL;
        a6 = new arbore;
        a6->st = NULL;
        a6->dr = NULL;
        a7 = new arbore;
        a7->st = NULL;
        a7->dr = NULL;
        strcpy(a7->info, "2");
        deriveaza(a->st, a1);
        a1 = simplificare(a1);
        deriveaza(a->dr, a2);
        a2 = simplificare(a2);
        strcpy(a3->info, "*");
        a3->st = a1;
        a3->dr = a->dr;
        a3 = simplificare(a3);
        strcpy(a4->info, "*");
        a4->st = a->st;
        a4->dr = a2;
        a4 = simplificare(a4);
        strcpy(a5->info, "-");
        a5->st = a3;
        a5->dr = a4;
        a5 = simplificare(a5);
        strcpy(a6->info, "^");
        a6->st = a->dr;
        a6->dr = a7;
        a6 = simplificare(a6);
        strcpy(d->info, "/");
        d->st = a5;
        d->dr = a6;
        d = simplificare(d);
    }

    if (a->info[0] == '^') { // derivam ridicarea la puterea/exponentiala
        if (strchr("0123456789e", a->st->info[0])) { // derivam exponentiala
            a1 = new arbore;
            a1->st = NULL;
            a1->dr = NULL;
            a2 = new arbore;
            a2->st = a->st;
            a2->dr = NULL;
            strcpy(a2->info, "ln");
            a2 = simplificare(a2);
            a3 = new arbore;
            a3->st = a;
            a3->dr = a2;
            strcpy(a3->info, "*");
            a3 = simplificare(a3);
            deriveaza(a->dr, a1);
            a1 = simplificare(a1);
            strcpy(d->info, "*");
            d->st = a3;
            d->dr = a1;
            d = simplificare(d);
        }
        else { // derivam ridicarea la putere a lui x
            a1 = new arbore;
            a1->st = NULL;
            a1->dr = NULL;
            a2 = new arbore;
            a2->st = NULL;
            a2->dr = NULL;
            strcpy(a2->info, "1");
            a3 = new arbore;
            a3->st = a->dr;
            a3->dr = a2;
            strcpy(a3->info, "-");
            a3 = simplificare(a3);
            a3->info[0] = '0' + a3->st->info[0] - a3->dr->info[0];
            a3->st = NULL;
            a3->dr = NULL;
            a4 = new arbore;
            a4->st = a->st;
            a4->dr = a3;
            strcpy(a4->info, "^");
            a4 = simplificare(a4);
            a5 = new arbore;
            a5->st = a->dr;
            a5->dr = a4;
            strcpy(a5->info, "*");
            a5 = simplificare(a5);
            deriveaza(a->st, a1);
            a1 = simplificare(a1);
            strcpy(d->info, "*");
            d->st = a5;
            d->dr = a1;
            d = simplificare(d);
        }
    }

    if (!strcmp("sin", a->info)) { // derivam sinus
        a1 = new arbore;
        a1->st = NULL;
        a1->dr = NULL;
        a2 = new arbore;
        a2->st = a->st;
        a2->dr = NULL;
        strcpy(a2->info, "cos");
        deriveaza(a->st, a1);
        a1 = simplificare(a1);
        strcpy(d->info, "*");
        d->st = a2;
        d->dr = a1;
        d = simplificare(d);
    }

    if (!strcmp("cos", a->info)) { // derivam cosinus
        a1 = new arbore;
        a1->st = NULL;
        a1->dr = NULL;
        a2 = new arbore;
        a2->st = a->st;
        a2->dr = NULL;
        strcpy(a2->info, "sin");
        a3 = new arbore;
        a3->st = NULL;
        a3->dr = NULL;
        deriveaza(a->st, a1);
        a1 = simplificare(a1);
        strcpy(a3->info, "*");
        a3->st = a2;
        a3->dr = a1;
        a3 = simplificare(a3);
        strcpy(d->info, "-");
        d->st = NULL;
        d->dr = a3;
        d = simplificare(d);
    }

    if (!strcmp("tg", a->info)) { // derivam tangent
        a1 = new arbore;
        a1->st = NULL;
        a1->dr = NULL;
        strcpy(a1->info, "1");
        a2 = new arbore;
        a2->st = NULL;
        a2->dr = NULL;
        strcpy(a2->info, "2");
        a3 = new arbore;
        a3->st = a->st;
        a3->dr = NULL;
        strcpy(a3->info, "cos");
        a4 = new arbore;
        a4->st = a3;
        a4->dr = a2;
        strcpy(a4->info, "^");
        a4 = simplificare(a4);
        a5 = new arbore;
        a5->st = a1;
        a5->dr = a4;
        strcpy(a5->info, "/");
        a5 = simplificare(a5);
        a6 = new arbore;
        a6->st = NULL;
        a6->dr = NULL;
        deriveaza(a->st, a6);
        a6 = simplificare(a6);
        strcpy(d->info, "*");
        d->st = a5;
        d->dr = a6;
        d = simplificare(d);
    }

    if (!strcmp("ctg", a->info)) { // derivam cotangent
        a1 = new arbore;
        a1->st = NULL;
        a1->dr = NULL;
        strcpy(a1->info, "1");
        a2 = new arbore;
        a2->st = NULL;
        a2->dr = NULL;
        strcpy(a2->info, "2");
        a3 = new arbore;
        a3->st = a->st;
        a3->dr = NULL;
        strcpy(a3->info, "sin");
        a4 = new arbore;
        a4->st = a3;
        a4->dr = a2;
        strcpy(a4->info, "^");
        a4 = simplificare(a4);
        a5 = new arbore;
        a5->st = a1;
        a5->dr = a4;
        strcpy(a5->info, "/");
        a5 = simplificare(a5);
        a6 = new arbore;
        a6->st = NULL;
        a6->dr = NULL;
        deriveaza(a->st, a6);
        a6 = simplificare(a6);
        a8 = new arbore;
        a8->st = a5;
        a8->dr = a6;
        strcpy(a8->info, "*");
        a8 = simplificare(a8);
        strcpy(d->info, "-");
        d->st = NULL;
        d->dr = a8;
        d = simplificare(d);
    }

    if (!strcmp("ln", a->info)) { // derivam logaritm natural
        a1 = new arbore;
        a1->st = NULL;
        a1->dr = NULL;
        strcpy(a1->info, "1");
        a2 = new arbore;
        a2->st = a1;
        a2->dr = a->st;
        strcpy(a2->info, "/");
        a2 = simplificare(a2);
        a3 = new arbore;
        a3->st = NULL;
        a3->dr = NULL;
        deriveaza(a->st, a3);
        a3 = simplificare(a3);
        strcpy(d->info, "*");
        d->st = a2;
        d->dr = a3;
        d = simplificare(d);
    }
    return d;
}
//ne foloseste la parcurgerea arborelui
int esteoperator2(char element[]) {

    if (strchr("+-*/^", element[0]) && element[1] == NULL) return 2;
    if (strchr("sctgl", element[0]) && strchr("iogtn", element[1]) && element[1] != NULL)
        return 1;
    return 0;
}

//prioritatea operatorilor pentru a compara la parcurgerea in inordine
int prioritate1(char c[]) {
    if (!strcmp(c, "+"))
        return 1;
    if (!strcmp(c, "-"))
        return 2;
    if (!strcmp(c, "*"))
        return 3;
    if (!strcmp(c, "^"))
        return 4;
    if (!strcmp(c, "/"))
        return 5;
    else
        return 6;
}

// parcurgem arborele derivatei si retinem formula intr-un sir de caractere
void parcurgere(arbore* a, char deriv[]) {
    if (a != NULL) {
        if (esteoperator2(a->info) == 1) {
            strcat(deriv, a->info);
            strcat(deriv, "(");
            parcurgere(a->st, deriv);
            parcurgere(a->dr, deriv);
            strcat(deriv, ")");
        }
        else {
            parcurgere(a->st, deriv);
            strcat(deriv, a->info);
            if ((a->dr != NULL) && (prioritate1(a->info) > prioritate1(a->dr->info))) {
                strcat(deriv, "(");
                parcurgere(a->dr, deriv);
                strcat(deriv, ")");
            }
            else parcurgere(a->dr, deriv);
        }

    }
}

void meniu() {
    int ok;
    click(ok);
    clearmouseclick(WM_LBUTTONDOWN);
    switch (ok) {
        click(ok);
    case 0: {
        closegraph();
        break;
    }
    case 1: { // INFORMATII PROIECT 
        setactivepage(0);
        titlu(530, 30, "DERIVE!");
        butoane();
        bar(0, 261, 1400, 697);
        setvisualpage(0);
        outtextxy(80, 290, "1. The program accepts the following functions: +, -, *, /, ^, e^(expression), sin(expression), cos(expression), tg(expression),");
        outtextxy(80, 320, "ctg(expression), ln(expression).");
        outtextxy(80, 350, "2. The program accepts as operands: integers and rational numbers (for rational ones you can use '.' as a");
        outtextxy(80, 380, "separator.)");
        outtextxy(80, 410, "3. The program accepts as variables 'x' and 'n'.");
        outtextxy(80, 440, "4. After entering the formula from the keyboard, press the ENTER key to continue. ");
        outtextxy(80, 470, "5. The program can read a mathematical expression from the file, by pressing the button DERIVATIVE.IN.");
        outtextxy(80, 500, "6. If you enter an incorrect character, press the BACKSPACE key to delete it.");
        outtextxy(80, 530, "7. The postfixed notation, derivative 1 and derivative of order 2 will be saved in the file derivative.out.");
        outtextxy(80, 580, "Student:");
        outtextxy(80, 610, "Iosub Miruna Elena");
        outtextxy(890, 580, "Professor:");
        outtextxy(890, 610, "Lect. dr. Patrut Bogdan");
        outtextxy(860, 630, "https://profs.info.uaic.ro/~introp/");
        meniu();
        break;
    }
    case 2: { // CITIRE DE LA TASTATURA 
        setactivepage(0);
        titlu(530, 30, "DERIVE!");
        butoane();
        bar(0, 261, 1400, 697);
        setvisualpage(0);
        memset(infix, NULL, 100);
        char c;
        int poz = 0;
        outtextxy(80, 290, "Enter the formula from keyboard:");
        c = getch();
        while (c != 13) {
            if (c == 8) {
                poz--;
                infix[poz] = '\0';
                setactivepage(0);
                titlu(530, 30, "DERIVE!");
                butoane();
                bar(0, 261, 1400, 697);
                setvisualpage(0);
                outtextxy(80, 290, "Enter the formula from keyboard:");
            }
            else {
                infix[poz] = c;
                poz++;
                infix[poz] = '\0';
            }
            outtextxy(80, 330, infix);
            c = getch();
        }
        meniu();
        break;
    }
    case 3:
        { 
        setactivepage(0);
        titlu(530, 30, "DERIVE!");
        butoane();
        bar(0, 261, 1400, 697);
        setvisualpage(0);
        memset(infix, NULL, 100);
        fin >> infix;
        outtextxy(80, 290, "The mathematical expression in deriv.in file:");
        outtextxy(80, 330, infix);
        meniu();
        break;
        }
    case 4:
        {
        setactivepage(0);
        titlu(530, 30, "DERIVE!");
        butoane();
        bar(0, 261, 1400, 697);
        setvisualpage(0);
        initcoada(postfix);
        InfixToPostfix(infix, postfix);
        fout << "The postfixed notation of: " << infix << " is: ";
        afiseaza(postfix);
        fout << endl;
        outtextxy(80, 290, "The mathematical expression:");
        outtextxy(80, 330, infix);
        outtextxy(80, 370, "The postfixed notation:");
        outtextxy(80, 410, postfixat);

        meniu();
        break;
        }
    case 5: { //arborele 
        setactivepage(0);
        titlu(530, 30, "DERIVE!");
        butoane();
        bar(0, 261, 1400, 697);
        setvisualpage(0);

        initcoada(postfix);
        InfixToPostfix(infix, postfix);
        stivaarbore S;
        initializeazastivaarbore(S);
        adaugaLaArboreElement(postfix, S);

        int height = 697, width = 1400;
        int window1 = initwindow(width, height, "Tree");
        setcurrentwindow(window1);
        setcolor(WHITE);
        setbkcolor(WHITE);
        cleardevice();
        rectangle(1, 1, width - 1, height - 1);
        int latime, inaltime;
        latime = width / nrColoane(S.varf->info);
        inaltime = height / nrNiveluri(S.varf->info);
        deseneazaArbore(S.varf->info, 1, 0, latime, inaltime);
        setcolor(WHITE);
        setlinestyle(0, 0, 1);
        setbkcolor(LIGHTMAGENTA);
        rectangle(50, 20, 175, 70);
        settextstyle(4, 0, 2);
        outtextxy(110, 50, "EXIT");
        int coordx, coordy;
        clearmouseclick(WM_LBUTTONDOWN);
        while (!ismouseclick(WM_LBUTTONDOWN)) {
            coordx = mousex();
            coordy = mousey();
        }
        if (coordx > 50 && coordx < 200 && coordy > 20 && coordy < 100)
            closegraph(CURRENT_WINDOW);

        setcurrentwindow(0);
        meniu();
        break;
    }
    case 6: { // DERIVATA INTAI

        setactivepage(0);
        titlu(530, 30, "DERIVE!");
        butoane();
        bar(0, 261, 1400, 697);
        setvisualpage(0);

        initcoada(postfix);
        InfixToPostfix(infix, postfix);
        stivaarbore S;
        initializeazastivaarbore(S);
        adaugaLaArboreElement(postfix, S);

        arbore* d;
        d = new arbore;
        d->st = NULL;
        d->dr = NULL;
        d = deriveaza(S.varf->info, d);

        arbore* d1;
        d1 = new arbore;
        d1->st = NULL;
        d1->dr = NULL;
        d1 = deriveaza(d, d1);

        memset(derivata1, NULL, 200);
        parcurgere(d, derivata1);
        outtextxy(80, 290, "The mathematical expression:");
        outtextxy(80, 330, infix);
        outtextxy(80, 370, "The first derivative:");
        if (strlen(derivata1) < 50)
            outtextxy(80, 410, derivata1);

        memset(derivata2, NULL, 200);
        parcurgere(d1, derivata2);
        outtextxy(80, 450, "The mathematical expression:");
        outtextxy(80, 490, infix);
        outtextxy(80, 530, "The second derivative:");
        outtextxy(80, 570, derivata2);



        setcolor(WHITE);
        setlinestyle(0, 0, 1);
        setbkcolor(5);
        settextstyle(4, 0, 2);
        outtextxy(1050, 270, "TREE");
        int coordx, coordy;
        clearmouseclick(WM_LBUTTONDOWN);
        while (!ismouseclick(WM_LBUTTONDOWN)) {
            coordx = mousex();
            coordy = mousey();
        }
        if (coordx > 1040 && coordx < 1200 && coordy > 270 && coordy < 320)
        {
            int height = 697, width = 1400;
            int window2 = initwindow(width, height, "TREE OF FIRST DERIVATIVE");
            setcurrentwindow(window2);
            setcolor(BLACK);
            setbkcolor(WHITE);
            cleardevice();
            rectangle(1, 1, width - 1, height - 1);
            int latime, inaltime;
            latime = width / nrColoane(d);
            inaltime = height / nrNiveluri(d);
            deseneazaArbore(d, 1, 0, latime, inaltime);

            setcolor(WHITE);
            setlinestyle(0, 0, 1);
            setbkcolor(LIGHTMAGENTA);

            settextstyle(4, 0, 2);
            outtextxy(110, 50, "EXIT");
            int coordx, coordy;
            clearmouseclick(WM_LBUTTONDOWN);
            while (!ismouseclick(WM_LBUTTONDOWN))
            {
                coordx = mousex();
                coordy = mousey();
            }
        if (coordx > 50 && coordx < 175 && coordy > 20 && coordy < 70)
            closegraph(CURRENT_WINDOW);
        }
        fout << "First derivative of: " << infix << " is: " << derivata1 << endl;
        setcurrentwindow(0);
        meniu();
        break;
    }
    case 7: { // DERIVATA DE ORDINUL 2
        setactivepage(0);
        titlu(530, 30, "DERIVE!");
        butoane();
        bar(0, 261, 1400, 697);
        setvisualpage(0);

        initcoada(postfix);
        InfixToPostfix(infix, postfix);
        stivaarbore S;
        initializeazastivaarbore(S);
        adaugaLaArboreElement(postfix, S);

        arbore* d;
        d = new arbore;
        d->st = NULL;
        d->dr = NULL;
        d = deriveaza(S.varf->info, d);

        arbore* d1;
        d1 = new arbore;
        d1->st = NULL;
        d1->dr = NULL;
        d1 = deriveaza(d, d1);

        memset(derivata2, NULL, 200);
        parcurgere(d1, derivata2);
        outtextxy(80, 290, "The mathematical expression:");
        outtextxy(80, 330, infix);
        outtextxy(80, 370, "The second derivative:");
        outtextxy(80, 410, derivata2);
        setcolor(WHITE);
        setlinestyle(0, 0, 1);
        settextstyle(4, 0, 2);
        setbkcolor(5);
        outtextxy(1250, 270, "TREE");
        int coordx, coordy;
        clearmouseclick(WM_LBUTTONDOWN);
        while (!ismouseclick(WM_LBUTTONDOWN))
        {
            coordx = mousex();
            coordy = mousey();
        }
        if (coordx > 1240 && coordx < 1400 && coordy > 270 && coordy < 320) {
            int height = 697, width = 1400;
            int window3 = initwindow(width, height, "TREE OF SECOND DERIVATIVE");
            setcurrentwindow(window3);
            setcolor(BLACK);
            setbkcolor(WHITE);
            cleardevice();
            rectangle(1, 1, width - 1, height - 1);
            int latime, inaltime;
            latime = width / nrColoane(d1);
            inaltime = height / nrNiveluri(d1);
            deseneazaArbore(d1, 1, 0, latime, inaltime);
            setcolor(WHITE);
            setlinestyle(0, 0, 1);

            settextstyle(4, 0, 2);
            setbkcolor(LIGHTMAGENTA);
            outtextxy(110, 50, "EXIT");
            int coordx, coordy;
            clearmouseclick(WM_LBUTTONDOWN);
            while (!ismouseclick(WM_LBUTTONDOWN)) {
                coordx = mousex();
                coordy = mousey();
            }
            if (coordx > 50 && coordx < 175 && coordy > 20 && coordy < 70)
                closegraph(CURRENT_WINDOW);
        }

        fout << "Second derivative of: " << infix << " is: " << derivata2 << endl;

        setcurrentwindow(0);
        meniu();
        break;
    }

    }
}

int main() {
    initwindow(1400, 697, "DERIVE!");
    setactivepage(0);
    titlu(530, 30, "DERIVE!");
    butoane();
    bar(0, 261, 1400, 697);
    setvisualpage(0);
    meniu();
    return 0;
}


