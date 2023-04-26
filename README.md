[productos.txt](https://github.com/Diego-Leon24/Proyecto-Minimarket/files/11329011/productos.txt)
[usuarios.txt](https://github.com/Diego-Leon24/Proyecto-Minimarket/files/11329012/usuarios.txt)
[ventas.txt](https://github.com/Diego-Leon24/Proyecto-Minimarket/files/11329013/ventas.txt)

//SI NO IMPRIME BIEN EL TEXTO ES POR QUE EXISTE UN ESPACIO (Revisar el archivo txt)
//Procuar al eliminar un producto ser bien expecificos porque puede eliminar con un mismo parentesco de caracter
#include <iostream>
#include <string>
#include <iomanip> // Incluir la biblioteca iomanip para utilizar setw
#include <algorithm>
#include <vector>
#include <fstream>
#include <time.h>
#include <ctime>

using namespace std;

struct cliente
{
    string u_dni;
    int usuario;
    string n_dni;

} cliente;

struct Producto {
    string nombre;
    int cantidad;
    double precio;
};
void agregarProducto() {
    ofstream archivo("productos.txt", ios::app);
    if (!archivo) {
        cerr << "No se pudo abrir el archivo" << endl;
        return;
    }
    cin.ignore();
    string nombre;
    int cantidad;
    double precio;
    cout << "Ingrese el nombre del producto: ";
    getline(cin, nombre);
    cout << "Ingrese la cantidad de " << nombre << " disponible: ";
    cin >> cantidad;
    cout << "Ingrese el precio de " << nombre << " por unidad: ";
    cin >> precio;
    archivo << nombre << " " << cantidad << " " << precio << endl;
    archivo.close();
    cout << "El producto se ha agregado correctamente al archivo." << endl;
}

const int ESPACIO_NOMBRE = 35;
const int ESPACIO_CANTIDAD = 10;
const int ESPACIO_PRECIO = 10;
const int CODIGO = 10;

bool compararPorNombre(const Producto& a, const Producto& b) {
    return a.nombre < b.nombre;
}

void mostrarProductos() {
    ifstream archivo("productos.txt");
    if (!archivo) {
        cerr << "No se pudo abrir el archivo" << endl;
        return;
    }
    vector<Producto> productos;
    Producto p;
    while (archivo >> p.nombre >> p.cantidad >> p.precio) {
        productos.push_back(p);
    }
    archivo.close();

    sort(productos.begin(), productos.end(), compararPorNombre);

    cout << left << setw(ESPACIO_NOMBRE) << "Nombre"<< setw(ESPACIO_CANTIDAD) << "Cantidad"<< setw(ESPACIO_PRECIO) << "Precio" << endl;
    cout << setfill('-') << setw(ESPACIO_NOMBRE + ESPACIO_CANTIDAD + ESPACIO_PRECIO) << "" << setfill(' ') << endl;
    for (const auto& producto : productos) {
        cout << left << setw(ESPACIO_NOMBRE) << producto.nombre << setw(ESPACIO_CANTIDAD) << producto.cantidad << "$" << setw(ESPACIO_PRECIO) << fixed << setprecision(2) << producto.precio << endl;
    }
}
void eliminar_producto() {
    
    string nombre_producto;
    cout << "Ingrese el nombre del producto que desea eliminar: ";
    cin.ignore();
    getline(cin, nombre_producto);
    
    // Verificamos que el nombre del producto ingresado no estÃ© vacÃ­o o contenga solo espacios en blanco
    if (nombre_producto.empty() || nombre_producto.find_first_not_of(' ') == string::npos) {
        cout << "Nombre de producto invÃ¡lido. Por favor, ingrese un nombre vÃ¡lido." << endl;
        return;
    }
    
    vector<string> productos; // vector para almacenar los productos
    
    ifstream archivo("productos.txt");
    if (archivo.is_open()) {
        string linea;
        while (getline(archivo, linea)) {
            //Se verifica que la lÃ­nea no estÃ© vacÃ­a mediante la funciÃ³n empty() y que el nombre del producto a eliminar coincida con la cadena al principio de la 
            //lÃ­nea utilizando la funciÃ³n substr().
            if (!linea.empty() && linea.substr(0, nombre_producto.size()) == nombre_producto) {
                // el producto coincide con el nombre a eliminar, no lo agregamos al vector
                continue;
            }
            productos.push_back(linea); // agregamos el producto al vector
        }
        archivo.close();
        
        ofstream archivo("productos.txt");
        if (archivo.is_open()) {
            for (const auto& producto : productos) {
                archivo << producto << endl; // escribimos los productos actualizados en el archivo
            }
            archivo.close();
            cout << "Producto eliminado correctamente." << endl;
        } else {
            cout << "No se pudo abrir el archivo para escribir." << endl;
        }
    } else {
        cout << "No se pudo abrir el archivo para leer." << endl;
    }
}


 void MenuAdmin();
 void MenuDueo();
 void MenuEmpleado();
// FunciÃ³n para el inicio de sesiÃ³n
bool iniciarSesion(const string& username, const string& password) {
    // Abrir el archivo para leer los datos de usuario existentes
    ifstream usuarios("usuarios.txt");
    string line;

    // Leer el archivo lÃ­nea por lÃ­nea y comparar los detalles de inicio de sesiÃ³n ingresados con los detalles existentes
    while (getline(usuarios, line)) {
        vector<string> campos;
        size_t pos = 0;
        string token;

        while ((pos = line.find(' ')) != string::npos) {
            token = line.substr(0, pos);
            campos.push_back(token);
            line.erase(0, pos + 1);
        }

        campos.push_back(line);

        if (campos.size() != 3) continue; // Saltear lÃ­neas con formato incorrecto

        if (campos[0] == username && campos[1] == password) {
            usuarios.close();

            // Verificar los permisos del usuario
            // Cada uno cuenta con un menu distinto
            if (campos[2] == "owner") {
                cout << "Inicio de sesion exitoso como Owner." << endl;
                MenuDueo();
            } else if (campos[2] == "admin") {
                cout << "Inicio de sesion exitoso como Administrador." << endl;
                MenuAdmin();
            } else if (campos[2] == "empleado") {
                cout << "Inicio de sesion exitoso como Empleado." << endl;
                MenuEmpleado();
            }

            return true; // Los detalles de inicio de sesiÃ³n son correctos
        }
    }
    usuarios.close();
    return false; // Los detalles de inicio de sesiÃ³n son incorrectos
}

// FunciÃ³n para mostrar todos los usuarios registrados
void mostrarUsuarios() {
    // Abrir el archivo para leer los datos de usuario existentes
    ifstream usuarios("usuarios.txt");
    string line;

    // Leer el archivo lÃ­nea por lÃ­nea y mostrar los usuarios
    while (getline(usuarios, line)) {
        vector<string> campos;
        size_t pos = 0;
        string token;

        // Extraer los campos de la lÃ­nea
        while ((pos = line.find(' ')) != string::npos) {
            token = line.substr(0, pos);
            campos.push_back(token);
            line.erase(0, pos + 1);
        }
        campos.push_back(line);

        // Verificar que la lÃ­nea tenga el formato correcto
        if (campos.size() != 3) {
            continue; // Saltear lÃ­neas con formato incorrecto
        }

        // Mostrar el usuario
        cout << "Nombre de usuario: " << campos[0] << endl;
        cout << "Contrasena: " << campos[1] << endl;
        cout << "Permiso: " << campos[2] << endl;
        cout << "-------------------------------------" << endl;
    }

    // Cerrar el archivo
    usuarios.close();
}


// FunciÃ³n para registrar un nuevo usuario
void registrarUsuario() {
    string username, password, permiso;
    
    // Solicitar los detalles del nuevo usuario
    cout << "Ingrese el nombre de usuario: ";
    cin >> username;
    cout << "Ingrese la contrasena: ";
    cin >> password;
    cout << "Ingrese el tipo de permiso (admin/empleado/owner): ";
    cin >> permiso;
    
    // Validar el tipo de permiso ingresado
    while (permiso != "admin" && permiso != "owner" && permiso != "empleado") {
        cout << "Tipo de permiso invÃ¡lido. Ingrese 'admin' o 'empleado' o 'owner': ";
        cin >> permiso;
    }
    
    // Abrir el archivo para agregar datos al final
    ofstream usuarios("usuarios.txt", ios::app);
    
    // Agregar los detalles del nuevo usuario al archivo
    usuarios << username << " " << password << " " << permiso << endl;
    
    // Cerrar el archivo
    usuarios.close();
}
void PantallaSesion() {
    string username, password;
    bool Identificador = false;
    int contador = 0;
    int Max = 3;
    
    do {
        cout << "1. Iniciar sesion" << endl;
        cout << "Nombre de usuario: ";
        cin >> username;
        cout << "Contrasena: ";
        cin >> password;
        
        if (iniciarSesion(username, password)) {
            break;
        } else {
            contador++;
            cout << "Nombre de usuario o contrasena incorrectos" << endl;
            cout << "Le quedan " << --Max << " intentos " << endl;
        }
        
        if (contador == 3) {
            cout << "Demasiados intentos fallidos. Saliendo del programa......." << endl;
            exit(1);
        }
    } while (!Identificador);
}
void venderProducto();
void MenuEmpleado(){


   //Verificacion de usuario
   string username, password;
    char opcion;
    
    do {
        cout << "Seleccione una opcion:\n";
        cout << "a) Mostrar lista de productos\n";
        cout << "b) Venta de productos\n";
        cout << "c) Iniciar sesion nuevamente\n";
        cout << "d) Salir\n";
        cin >> opcion;

        switch (opcion) {
            
            case 'a':
                mostrarProductos();
                break;
             case 'b':

                venderProducto();
                break;
            case 'c':
                
                PantallaSesion();
                break;
            
            case 'd':
                cout << "Adios.\n";
                exit(1);
                break;
            
            default:
                cout << "Opcion invalida.\n";
                break;
        }
    } while (opcion != 'd');

    
}
void MenuAdmin(){


   //Verificacion de usuario
   string username, password;
    char opcion;
    
    do {
        cout << "Seleccione una opcion:\n";
        cout << "a) Agregar producto\n";
        cout << "b) Mostrar lista de productos\n";
        cout << "c) Iniciar sesion nuevamente\n";
        cout << "d) Venta de productos\n";
        cout << "e) Salir\n";
        cin >> opcion;

        switch (opcion) {
            case 'a':
                agregarProducto();
                break;
            case 'b':
                mostrarProductos();
                break;
             case 'c':

                PantallaSesion();

                break;
            case 'd':
                venderProducto();
                break;
            
            case 'e':
                cout << "Adios.\n";
                exit(1);
                break;
            
            default:
                cout << "Opcion invalida.\n";
                break;
        }
    } while (opcion != 'e');

    
}
void guardarVenta(const vector<Producto>& productos, const vector<int>& seleccionados, const vector<int>& cantidadesVendidas, double totalVenta);
void escribirTotalVentasDia();
void MenuDueo(){

    
   //Verificacion de usuario
   string username, password;
    char opcion;
    
    do {
        cout << "Seleccione una opcion:\n";
        cout << "a) Agregar producto\n";
        cout << "b) Mostrar lista de productos\n";
        cout << "c) Iniciar sesion nuevamente\n";
        cout << "d) Mostrar todos los usuarios \n";
        cout << "e) Registrar nuevo usuario \n";
        cout << "f) Eliminar producto\n";
        cout << "g) Venta de productos\n";
        cout << "h) Salir\n";
        cin >> opcion;
       //Se puede agregar para que no muestre el historial
        switch (opcion) {
            case 'a':
                agregarProducto();
                break;
            case 'b':
                mostrarProductos();
                break;
            case 'c':

                PantallaSesion();

                break;
                case 'd':

                mostrarUsuarios();

                break;
                case 'e':

                registrarUsuario();

                break;
                case 'f':

                eliminar_producto();

                break;
            case 'g':
                venderProducto();
                break;
                case 'h':
                cout << "Adios.\n";
                exit(1);
                break;
            
            default:
                cout << "Opcion invalida.\n";
                break;
        }
    } while (opcion != 'h');

    
}
void venderProducto() {
    // Abrir archivo de productos
    ifstream archivo("productos.txt");
    if (!archivo.is_open()) {
        cout << "No se pudo abrir el archivo de productos." << endl;
        return;
    }
    
    // Leer productos del archivo y almacenarlos en un vector
    vector<Producto> productos;
    string linea;
    while (getline(archivo, linea)) {
        stringstream registro(linea);
        Producto p;
        registro >> p.nombre >> p.cantidad >> p.precio;
        productos.push_back(p);
    }
    archivo.close();

    // Obtener cantidad de productos a vender
    int cantidadProductos;
    do {
        cout << "Ingrese la cantidad de productos a vender (o 0 para cancelar): ";
        cin >> cantidadProductos;
    } while (cantidadProductos < 0 || cantidadProductos > static_cast<int>(productos.size()));
    if (cantidadProductos == 0) {
            return;
        }
    // Mostrar lista de productos disponibles
    cout << "Productos disponibles:" << endl;
    cout << left << setw(CODIGO) << "CODIGO" << setw(ESPACIO_NOMBRE) << "Nombre"<< setw(ESPACIO_CANTIDAD) << "Cantidad"<< setw(ESPACIO_PRECIO) << "Precio" << endl;
    cout << setfill('-') << setw(ESPACIO_NOMBRE + ESPACIO_CANTIDAD + ESPACIO_PRECIO) << "" << setfill(' ') << endl;
    for (std::vector<Producto>::size_type i = 0; i < productos.size(); i++) {
        cout << left <<setw(CODIGO)<<i + 1 << ". " << setw(ESPACIO_NOMBRE) << productos[i].nombre << setw(ESPACIO_CANTIDAD) << productos[i].cantidad << "$" << setw(ESPACIO_PRECIO) << fixed << setprecision(2) << productos[i].precio << endl;
    }

    // Seleccionar productos a vender
    vector<int> seleccionados;
    int seleccion;
    for (int i = 0; i < cantidadProductos; i++) {
        do {
            cout << "Ingrese el numero del producto #" << i+1 << " a vender (o 0 para cancelar): ";
            cin >> seleccion;
        } while (seleccion < 0 || seleccion > static_cast<int>(productos.size()) || find(seleccionados.begin(), seleccionados.end(), seleccion) != seleccionados.end());
        if (seleccion == 0) {
            return;
        }
        seleccionados.push_back(seleccion);
    }
    
    // Obtener cantidades a vender
    vector<int> cantidadesVendidas;
    for (int i = 0; i < cantidadProductos; i++) {
        int cantidadVendida;
        do {
            cout << "Ingrese la cantidad del producto #" << i+1 << " a vender (o 0 para cancelar): ";
            cin >> cantidadVendida;
        } while (cantidadVendida < 0 || cantidadVendida > productos[seleccionados[i] - 1].cantidad);
        if (cantidadVendida == 0) {
            return;
        }
        cantidadesVendidas.push_back(cantidadVendida);
    }
    
    // Actualizar cantidades de productos en el vector
    for (int i = 0; i < cantidadProductos; i++) {
        productos[seleccionados[i] - 1].cantidad -= cantidadesVendidas[i];
    }
    
    // Actualizar archivo de productos
   ofstream archivoActualizado("productos.txt");
   if (!archivoActualizado.is_open()) {
   cout << "No se pudo abrir el archivo de productos para actualizar." << endl;
  return;
   }
   
   for (std::vector<Producto>::size_type i = 0; i < productos.size(); i++) {
   archivoActualizado << productos[i].nombre << " " << productos[i].cantidad << " " << productos[i].precio << endl;
   }
   archivoActualizado.close();

    // Calcular total de venta
    double totalVenta = 0.0;
    for (int i = 0; i < cantidadProductos; i++) {
        totalVenta += cantidadesVendidas[i] * productos[seleccionados[i] - 1].precio;
    }

    //Imprimir Boletas productos
     cout << "Por fabor, antes de emitir su boleta indicar su DNI:" << endl;
        // repetir hasta que sea valido el dni
        do
        {
            cin >> cliente.n_dni;
            if (cliente.n_dni.size() != 8)
            {
                cout << "*Error, dni incorrecto. Ingrese su dni nuevamente: " << endl;
            }
            else
            {
                cliente.u_dni = cliente.n_dni;
            }
        } while (cliente.n_dni.size() != 8);
    cliente.usuario = (rand() % 9999);
    cliente.u_dni = "0";
    cout << "" << endl;
        cout << "-----------------------------------------------" << endl;
        cout << "MYPE: " << endl;
        cout << "" << endl;
        cout << "       _____   _____  ____    ____  _____    " << endl;
        cout << "      |_   _| | ____|/ ___| / ___|| ____|    " << endl;
        cout << "        | |   |  _|  \\___  \\___ | |___|    " << endl;
        cout << "        | |   | |___  ___) | ___) | |___     " << endl;
        cout << "        |_|   |_____|/____/ |____/|_____|    " << endl;
        cout << "                                             " << endl;
        cout << "" << endl;
        cout << "      R.U.C.: N 2056821563" << endl;
        cout << "        BOLETA ELECTRONICA" << endl;
        cout << "          N" << (rand() % 999999) << endl;
        cout << "" << endl;
        cout << "Cliente: U" << cliente.usuario << endl;
        cout << "DNI: " << cliente.n_dni << endl;
        cout << "" << endl;

        cout << left << setw(ESPACIO_NOMBRE) << "Nombre"<< setw(ESPACIO_CANTIDAD) << "Cantidad"<< setw(ESPACIO_PRECIO) << "Precio" << endl;
        cout << setfill('-') << setw(ESPACIO_NOMBRE + ESPACIO_CANTIDAD + ESPACIO_PRECIO) << "" << setfill(' ') << endl;
        for (int i=0;i<cantidadProductos;i++) {
        cout << left << setw(ESPACIO_NOMBRE) << productos[seleccionados[i] - 1].nombre<< setw(ESPACIO_CANTIDAD) << cantidadesVendidas[i] << "$" << setw(ESPACIO_PRECIO) << fixed << setprecision(2) << productos[seleccionados[i] - 1].precio << endl;
    }
    // Mostrar total de venta y despedida

    cout << "" << endl;
    cout << "Total:                        S/." << totalVenta << endl;
    cout << "IGV (18%)                     S/." << totalVenta * 18 / 100 << endl;
    cout << "PAGAR:                        S/." << int((totalVenta + totalVenta * 18 / 100) + .5) << endl;
    cout << "" << endl;
    cout << "         Gracias por su preferencia!" << endl;
    cout << "" << endl;
    cout << "-----------------------------------------------" << endl;
    cout << "" << endl;
    guardarVenta(productos,seleccionados,cantidadesVendidas,totalVenta);
    escribirTotalVentasDia();
    
}




// Variable global para almacenar el total de ventas del dÃ­a
double totalVentasDia = 0.0;

// FunciÃ³n para guardar una venta en el archivo "ventas.txt"
void guardarVenta(const vector<Producto>& productos, const vector<int>& seleccionados, const vector<int>& cantidadesVendidas, double totalVenta) {
    ofstream archivo("ventas.txt", ios_base::app);
    if (!archivo.is_open()) {
        cout << "No se pudo abrir el archivo de ventas." << endl;
        return;
    }
    
    // Obtener fecha y hora actual
    time_t tiempoActual = time(nullptr);
    tm* fechaHoraActual = localtime(&tiempoActual);
    char buffer[80];
    strftime(buffer, 80, "%d/%m/%Y %H:%M:%S", fechaHoraActual);
    
    // Escribir informaciÃ³n de la venta en el archivo
    archivo << "Fecha y hora: " << buffer << endl;
    archivo << "Productos vendidos:" << endl;
    archivo << left << setw(CODIGO) << "CODIGO" << setw(ESPACIO_NOMBRE) << "Nombre"<< setw(ESPACIO_CANTIDAD) << "Cantidad"<< setw(ESPACIO_PRECIO) << "Precio" << endl;
    archivo << setfill('-') << setw(ESPACIO_NOMBRE + ESPACIO_CANTIDAD + ESPACIO_PRECIO) << "" << setfill(' ') << endl;
    for (std::vector<int>::size_type i = 0; i < seleccionados.size(); i++) {
        archivo << left <<setw(CODIGO)<<seleccionados[i] << ". " << setw(ESPACIO_NOMBRE) << productos[seleccionados[i] - 1].nombre << setw(ESPACIO_CANTIDAD) << cantidadesVendidas[i] << "$" << setw(ESPACIO_PRECIO) << fixed << setprecision(2) << productos[seleccionados[i] - 1].precio << endl;
    }
    
    archivo << "Total de venta: $" << totalVenta << endl << endl;
    
    // Incrementar el total de ventas del dÃ­a
    totalVentasDia += totalVenta;
    
    archivo.close();
}
// FunciÃ³n para escribir el total de ventas del dÃ­a en el archivo "total_ventas.txt"
void escribirTotalVentasDia() {
    ofstream archivo("ventas.txt", ios_base::app);
    if (!archivo.is_open()) {
        cout << "No se pudo abrir el archivo de total de ventas." << endl;
        return;
    }
    
    // Obtener fecha actual
    time_t tiempoActual = time(nullptr);
    tm* fechaActual = localtime(&tiempoActual);
    char buffer[80];
    strftime(buffer, 80, "%d/%m/%Y", fechaActual);
    
    archivo << "Total de ventas del " << buffer << ": $" << totalVentasDia << endl;
    archivo<<""<<endl;
    archivo.close();
   

}


int main() {
    
    // Interface del sistema ----------------------------------------
    cout << "**************************************************************************   " << endl;
    cout << "      _____            ______    ______   ______      __   __  ___  __          " << endl;
    cout << "     |       |     |  |      |  |        |      |    |  | |__| |__ |  |      " << endl;
    cout << "     |_____  |     |  | _____|  | ____   |   ___|    |__| |    |__ |  |      " << endl;
    cout << "           | |     |  |         |        |  |___                              " << endl;
    cout << "      _____| |_____|  |         |_____   |      |                             " << endl;
    cout << "      ____________________________________________________________________   " << endl;
    cout << "**************************************************************************   " << endl;[Uploading productos.txt…]()

    cout << "Empresa: MarkerTESSE S.A.C  - R.U.C.: 2056821563" << endl;
    cout << "" << endl;
    PantallaSesion();
    return 0; 
}
