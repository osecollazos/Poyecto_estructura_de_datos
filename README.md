#include <iostream>
using namespace std;

const int MEMORIA_TOTAL = 5;
int memoriaDisponible = MEMORIA_TOTAL;
int procesosCreados = 0;
int siguienteID = 1;

struct Proceso {
    int id;
    string estado;
    string prioridad;
    int memoria[100]; // Pila de memoria
    int topeMemoria;
    Proceso* siguiente;
};

