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

Proceso* cabeza = NULL;

string obtenerEstado(int opcion) {
    switch (opcion) {
        case 1: return "listo";
        case 2: return "ejecutando";
        case 3: return "bloqueado";
        default: return "invalido";
    }
}

string obtenerPrioridad(int opcion) {
    switch (opcion) {
        case 1: return "Alta";
        case 2: return "Media";
        case 3: return "Baja";
        default: return "invalida";
    }
}

bool idExiste(int id) {
    Proceso* aux = cabeza;
    while (aux != NULL) {
        if (aux->id == id)
            return true;
        aux = aux->siguiente;
    }
    return false;
}

void crearProceso(string estado, string prioridad) {
    if (estado == "invalido" || prioridad == "invalida") {
        cout << "Estado o prioridad inválida.\n";
        return;
    }

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

    procesosCreados++;
    cout << "Proceso creado con ID: " << nuevo->id << ", Estado: " << estado << ", Prioridad: " << prioridad << endl;

    if (procesosCreados == 4)
        cout << " Memoria casi llena (por cantidad de procesos).\n";
    else if (procesosCreados == 5)
        cout << " Memoria llena (por cantidad de procesos).\n";
}

void mostrarProcesos() {
    Proceso* aux = cabeza;
    cout << "\n--- Procesos registrados ---\n";
    while (aux != NULL) {
        cout << "ID: " << aux->id
             << ", Estado: " << aux->estado
             << ", Prioridad: " << aux->prioridad
             << ", Memoria usada (pila): " << aux->topeMemoria + 1;
        if (aux->topeMemoria + 1 >= 90)
            cout << " (¡Memoria casi llena!)";
        cout << endl;
        aux = aux->siguiente;
    }

    cout << "Memoria disponible del sistema: " << memoriaDisponible << " unidades\n";

    if (memoriaDisponible == 0)
        cout << " Memoria del sistema llena.\n";
    else if (memoriaDisponible <= 1)
        cout << " Advertencia: Memoria del sistema casi llena.\n";
}

void cambiarEstado(int id, string nuevoEstado) {
    if (nuevoEstado == "invalido") {
        cout << "Estado inválido.\n";
        return;
    }

    Proceso* aux = cabeza;
    while (aux != NULL) {
        if (aux->id == id) {
            aux->estado = nuevoEstado;
            cout << "Estado del proceso " << id << " cambiado a " << nuevoEstado << endl;
            return;
        }
        aux = aux->siguiente;
    }

    cout << "Proceso no encontrado.\n";
}

void usarMemoria(int id, int cantidad) {
    Proceso* aux = cabeza;
    while (aux != NULL) {
        if (aux->id == id) {
            if (aux->topeMemoria + cantidad >= 100) {
                cout << "Memoria del proceso llena (máximo 100).\n";
                return;
            }
            if (memoriaDisponible < cantidad) {
                cout << "Error: No hay suficiente memoria del sistema.\n";
                return;
            }
            for (int i = 0; i < cantidad; i++)
                aux->memoria[++aux->topeMemoria] = 1;

            memoriaDisponible -= cantidad;
            cout << "Memoria apilada al proceso " << id << ": " << cantidad << " unidades.\n";
            return;
        }
        aux = aux->siguiente;
    }

    cout << "Proceso no encontrado.\n";
}

void quitarMemoria(int id, int cantidad) {
    Proceso* aux = cabeza;
    while (aux != NULL) {
        if (aux->id == id) {
            if (aux->topeMemoria < cantidad - 1) {
                cout << "No hay suficiente memoria en la pila del proceso.\n";
                return;
            }
            for (int i = 0; i < cantidad; i++)
                aux->topeMemoria--;

            memoriaDisponible += cantidad;
            cout << "Memoria desapilada del proceso " << id << ": " << cantidad << " unidades.\n";
            return;
        }
        aux = aux->siguiente;
    }

    cout << "Proceso no encontrado.\n";
}

void eliminarProceso(int id) {
    Proceso* actual = cabeza;
    Proceso* anterior = NULL;

    while (actual != NULL) {
        if (actual->id == id) {
            int memoriaLiberada = actual->topeMemoria + 1;
            memoriaDisponible += memoriaLiberada;

            if (anterior == NULL) {
                cabeza = actual->siguiente;
            } else {
                anterior->siguiente = actual->siguiente;
            }

            delete actual;
            cout << "Proceso " << id << " eliminado. Memoria liberada: " << memoriaLiberada << " unidades.\n";
            return;
        }
        anterior = actual;
        actual = actual->siguiente;
    }

    cout << "Proceso no encontrado.\n";
}

void liberarEspacioMemoria() {
    while (cabeza != NULL) {
        mostrarProcesos();
        cout << "\n¿Desea eliminar el primer proceso? (s/n): ";
        char opcion;
        cin >> opcion;

        if (opcion == 's' || opcion == 'S') {
            int idEliminar = cabeza->id;
            eliminarProceso(idEliminar);
        } else {
            cout << "Proceso no eliminado. Fin de la limpieza.\n";
            break;
        }
    }

    if (cabeza == NULL) {
        cout << "Todos los procesos han sido eliminados.\n";
    }
}

void menu() {
    int opcion;
    do {
        cout << "\n===== MENÚ DE GESTIÓN DE PROCESOS =====\n";
        cout << "1. Crear proceso\n";
        cout << "2. Cambiar estado de proceso\n";
        cout << "3. Agregar memoria a proceso (Push)\n";
        cout << "4. Mostrar todos los procesos\n";
        cout << "5. Salir\n";
        cout << "6. Eliminar proceso completamente\n";
        cout << "7. Liberar espacio de memoria\n";
        cout << "8. Quitar memoria a proceso (Pop)\n";
        cout << "Seleccione una opción: ";
        cin >> opcion;

        int id, cantidad, estadoNum, prioridadNum;
        string estado, prioridad;

        switch (opcion) {
            case 1:
                while (true) {
                    cout << "Estado:\n 1. listo\n 2. ejecutando\n 3. bloqueado\nSeleccione: ";
                    cin >> estadoNum;
                    estado = obtenerEstado(estadoNum);
                    if (estado == "invalido") {
                        cout << "ERROR, la opción que marcó no existe. Elija una de las 3 opciones dadas: [1. listo, 2. ejecutando, 3. bloqueado]\n";
                    } else {
                        break;
                    }
                }

                while (true) {
                    cout << "Prioridad:\n 1. Alta\n 2. Media\n 3. Baja\nSeleccione: ";
                    cin >> prioridadNum;
                    prioridad = obtenerPrioridad(prioridadNum);
                    if (prioridad == "invalida") {
                        cout << "ERROR, la opción que marcó no existe. Elija una de las 3 opciones dadas: [1. Alta, 2. Media, 3. Baja]\n";
                    } else {
                        break;
                    }
                }

                crearProceso(estado, prioridad);
                break;

            case 2:
                cout << "ID del proceso: ";
                cin >> id;
                if (!idExiste(id)) {
                    cout << "Proceso no encontrado.\n";
                    break;
                }

                while (true) {
                    cout << "Nuevo estado:\n 1. listo\n 2. ejecutando\n 3. bloqueado\nSeleccione: ";
                    cin >> estadoNum;
                    estado = obtenerEstado(estadoNum);
                    if (estado == "invalido") {
                        cout << "ERROR, la opción que marcó no existe. Elija una de las 3 opciones dadas: [1. listo, 2. ejecutando, 3. bloqueado]\n";
                    } else {
                        break;
                    }
                }

                cambiarEstado(id, estado);
                break;

            case 3:
                cout << "ID del proceso: ";
                cin >> id;
                if (!idExiste(id)) {
                    cout << "Proceso no encontrado.\n";
                    break;
                }
                cout << "Cantidad de memoria a apilar: ";
                cin >> cantidad;
                usarMemoria(id, cantidad);
                break;

            case 4:
                mostrarProcesos();
                break;

            case 5:
                cout << "Saliendo del sistema...\n";
                break;

            case 6:
                cout << "ID del proceso a eliminar: ";
                cin >> id;
                eliminarProceso(id);
                break;

            case 7:
                liberarEspacioMemoria();
                break;

            case 8:
                cout << "ID del proceso: ";
                cin >> id;
                if (!idExiste(id)) {
                    cout << "Proceso no encontrado.\n";
                    break;
                }
                cout << "Cantidad de memoria a desapilar: ";
                cin >> cantidad;
                quitarMemoria(id, cantidad);
                break;

            default:
                cout << "Opción inválida.\n";
        }
    } while (opcion != 5);
}

int main() {
    menu();
    return 0;
}
