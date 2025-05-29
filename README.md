#include <iostream>
using namespace std;

struct Proceso {
    int id;
    string estado; // "listo", "ejecutando", "bloqueado"
    int prioridad;
    int memoria[100]; // pila de memoria simple (arreglo)
    int topeMemoria;
    Proceso* siguiente;
};

Proceso* cabeza = NULL;
int siguienteID = 1;

// Agregar proceso al final de la lista
void crearProceso(string estado, int prioridad) {
    Proceso* nuevo = new Proceso;
    nuevo->id = siguienteID++;
    nuevo->estado = estado;
    nuevo->prioridad = prioridad;
    nuevo->topeMemoria = -1;
    nuevo->siguiente = NULL;

    if (cabeza == NULL) {
        cabeza = nuevo;
    } else {
        Proceso* aux = cabeza;
        while (aux->siguiente != NULL)
            aux = aux->siguiente;
        aux->siguiente = nuevo;
    }

    cout << "Proceso creado con ID: " << nuevo->id << ", Estado: " << estado << endl;
}

// Mostrar todos los procesos
void mostrarProcesos() {
    Proceso* aux = cabeza;
    while (aux != NULL) {
        cout << "ID: " << aux->id
             << ", Estado: " << aux->estado
             << ", Prioridad: " << aux->prioridad
             << ", Memoria usada: " << aux->topeMemoria + 1 << endl;
        aux = aux->siguiente;
    }
}

// Cambiar estado de un proceso
void cambiarEstado(int id, string nuevoEstado) {
    Proceso* aux = cabeza;
    while (aux != NULL) {
        if (aux->id == id) {
            aux->estado = nuevoEstado;
            cout << "Estado del proceso " << id << " cambiado a " << nuevoEstado << endl;
            return;
        }
        aux = aux->siguiente;
    }
    cout << "Proceso no encontrado." << endl;
}

// Función principal
int main() {
    crearProceso("listo", 3);
    crearProceso("ejecutando", 1);
    crearProceso("bloqueado", 2);

    cout << "\n--- Procesos actuales ---" << endl;
    mostrarProcesos();

    cout << "\n--- Cambiando estado del proceso 2 ---" << endl;
    cambiarEstado(2, "bloqueado");

    cout << "\n--- Procesos actualizados ---" << endl;
    mostrarProcesos();

    return 0;
}

