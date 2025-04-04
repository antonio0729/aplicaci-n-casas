# aplicacion-casas
import sqlite3

# Conexión a la base de datos
def connect_db():
    return sqlite3.connect('database.db')

# Inicializar la base de datos
def initialize_database():
    conn = connect_db()
    cursor = conn.cursor()
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS users (
            cedula TEXT PRIMARY KEY,
            name TEXT,
            password TEXT,
            role TEXT
        )
    ''')
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS houses (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            cedula TEXT,
            sector TEXT,
            price TEXT,
            description TEXT
        )
    ''')
    conn.commit()

    # Crear usuario
    try:
        cursor.execute("INSERT INTO users (cedula, name, password, role) VALUES ('1043436148', 'Antonio', 'antoni123', 'vendedor')")
        cursor.execute("INSERT INTO houses (cedula, sector, price, description) VALUES ('1043436148', 'San José', '$120,000', 'Casa grande con patio.')")
        conn.commit()
        print("Usuario Antonio y su propiedad han sido registrados.")
    except sqlite3.IntegrityError:
        print("Usuario Antonio ya esta registrado en la base de datos.")

    conn.close()
    print("Base de datos inicializada correctamente.")

# Crear un nuevo usuario
def create_user():
    conn = connect_db()
    cursor = conn.cursor()
    cedula = input("Ingrese la cedula: ").strip()
    name = input("Ingrese el nombre: ").strip()
    password = input("Ingrese la contraseña: ").strip()
    while True:
        print("Seleccione el rol:")
        print("1. Comprador")
        print("2. Vendedor")
        role_option = input("Ingrese el numero correspondiente: ").strip()
        if role_option == "1":
            role = "comprador"
            break
        elif role_option == "2":
            role = "vendedor"
            break
        else:
            print("Opcion invalida. Por favor elija 1 o 2.")
    try:
        cursor.execute("INSERT INTO users (cedula, name, password, role) VALUES (?, ?, ?, ?)", (cedula, name, password, role))
        conn.commit()
        print("Usuario registrado exitosamente.")
    except sqlite3.IntegrityError:
        print("Error: La cedula ya esta registrada.")
    conn.close()

# Verificar usuarios registrados
def verify_users():
    conn = connect_db()
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users")
    users = cursor.fetchall()
    conn.close()
    print("Usuarios registrados:")
    for user in users:
        print(user)

# Mostrar propiedades del vendedor
def view_seller_properties(cedula):
    conn = connect_db()
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM houses WHERE cedula = ?", (cedula,))
    houses = cursor.fetchall()
    conn.close()
    if houses:
        print(f"Tienes {len(houses)} propiedad registrada:\n")
        for house in houses:
            print(f"Sector: {house[2]}, Precio: {house[3]}, Descripción: {house[4]}")
    else:
        print("No tienes propiedades registradas.")

# Mostrar casas disponibles agrupadas por sector
def view_buyer_houses():
    example_houses = [
        {"sector": "San Jose", "precio": "$120,000", "descripcion": "Casa grande con patio."},
        {"sector": "El Silencio", "precio": "$80,000", "descripcion": "Casa pequeña y acogedora."},
        {"sector": "Soledad", "precio": "$100,000", "descripcion": "Casa espaciosa con jardín."},
        {"sector": "Alameda", "precio": "$250,000", "descripcion": "Apartamento moderno."}
    ]
    print("Casas disponibles agrupadas por sector:")
    for house in example_houses:
        print(f"Sector: {house['sector']}, Precio: {house['precio']}, Descripción: {house['descripcion']}")

# Login y verificación de credenciales
def login():
    cedula = input("Ingrese su cedula: ").strip()
    password = input("Ingrese su contraseña: ").strip()
    conn = connect_db()
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users WHERE cedula = ? AND password = ?", (cedula, password))
    user = cursor.fetchone()
    conn.close()
    if user:
        print(f"\nBienvenid@, {user[1]}!\n \nRol: {user[3]}\n")
        if user[3] == "comprador":
            view_buyer_houses()
        elif user[3] == "vendedor":
            view_seller_properties(user[0])
    else:
        print("Credenciales incorrectas. Intente nuevamente.")

# Menú interactivo
def menu():
    running = True
    while running:
        print("\nAplicación de Venta de Casas\n")
        print("1. Inicializar base de datos")
        print("2. Crear un nuevo usuario")
        print("3. Verificar usuarios registrados")
        print("4. Iniciar sesion")
        print("5. Salir")
        option = input("Seleccione una opcion: ").strip()
        if option == "1":
            initialize_database()
        elif option == "2":
            create_user()
        elif option == "3":
            verify_users()
        elif option == "4":
            login()
        elif option == "5":
            print("¡Hasta luego!")
            running = False
        else:
            print("Opcion inválida. Intente nuevamente.")

# Ejecutar menú
if __name__ == "__main__":
    menu()
    
