import os

class Producto:
    """
    Clase que representa un producto en la Tienda de Víveres.
    Atributos:
        codigo (str): Identificador único del producto.
        nombre (str): Nombre del producto (ej. Arroz, Frijol, Azúcar, etc.).
        cantidad (int): Cantidad disponible en inventario.
        precio (float): Precio unitario del producto.
    """

    def __init__(self, codigo, nombre, cantidad, precio):
        self.codigo = codigo
        self.nombre = nombre
        self.cantidad = cantidad
        self.precio = precio

    def __str__(self):
        """
        Retorna una representación en cadena del producto,
        con los atributos separados por punto y coma (;).
        Formato: codigo;nombre;cantidad;precio
        """
        return f"{self.codigo};{self.nombre};{self.cantidad};{self.precio}"

    @staticmethod
    def desde_cadena(cadena):
        """
        Crea una instancia de Producto a partir de una cadena con el formato:
            codigo;nombre;cantidad;precio
        Maneja ValueError si la cadena no está bien formada.
        """
        partes = cadena.strip().split(";")
        if len(partes) != 4:
            raise ValueError("Formato de línea inválido (se esperan 4 campos).")
        codigo, nombre, cantidad_str, precio_str = partes
        return Producto(
            codigo=codigo,
            nombre=nombre,
            cantidad=int(cantidad_str),
            precio=float(precio_str)
        )


class Inventario:
    """
    Clase que gestiona el inventario de productos de la Tienda de Víveres.
    Permite añadir, actualizar, eliminar y consultar productos, así como
    guardar y cargar la información desde un archivo de texto.
    """

    def __init__(self, archivo_inventario="inventario.txt"):
        """
        Constructor de la clase Inventario.
        Carga el inventario desde el archivo especificado (si existe).
        Si no existe, lo crea.
        """
        self.archivo_inventario = archivo_inventario
        self.productos = {}
        self.cargar_inventario()

    def cargar_inventario(self):
        """
        Carga la información de productos desde el archivo de inventario.
        Maneja excepciones de archivo (FileNotFoundError, PermissionError)
        y errores de formato (ValueError).
        """
        if not os.path.exists(self.archivo_inventario):
            print(f"Archivo '{self.archivo_inventario}' no encontrado. Creando uno nuevo...")
            open(self.archivo_inventario, "w").close()
            return

        try:
            with open(self.archivo_inventario, "r", encoding="utf-8") as f:
                for linea in f:
                    linea = linea.strip()
                    if not linea:
                        continue  # Saltar líneas en blanco
                    try:
                        producto = Producto.desde_cadena(linea)
                        self.productos[producto.codigo] = producto
                    except ValueError as ve:
                        print(f"Advertencia: no se pudo cargar la línea '{linea}'. Error: {ve}")
        except FileNotFoundError:
            print(f"Error: No se encontró el archivo {self.archivo_inventario}. Se creará uno nuevo.")
            open(self.archivo_inventario, "w").close()
        except PermissionError:
            print(f"Error: No se tienen permisos para leer el archivo {self.archivo_inventario}.")
        except Exception as e:
            print(f"Ocurrió un error inesperado al cargar el inventario: {e}")

    def guardar_inventario(self):
        """
        Guarda la información de todos los productos en el archivo de inventario.
        Maneja excepciones de archivo (PermissionError) y notifica el éxito o fallo.
        """
        try:
            with open(self.archivo_inventario, "w", encoding="utf-8") as f:
                for producto in self.productos.values():
                    f.write(str(producto) + "\n")
            print("Inventario guardado exitosamente.")
        except PermissionError:
            print(f"Error: No se tienen permisos para escribir en {self.archivo_inventario}.")
        except Exception as e:
            print(f"Ocurrió un error inesperado al guardar el inventario: {e}")

    def agregar_producto(self, codigo, nombre, cantidad, precio):
        """
        Agrega un nuevo producto al inventario.
        Si el código ya existe, se notifica.
        Luego guarda los cambios en el archivo.
        """
        if codigo in self.productos:
            print(f"El producto con código '{codigo}' ya existe.")
            return

        nuevo_producto = Producto(codigo, nombre, cantidad, precio)
        self.productos[codigo] = nuevo_producto
        print(f"Producto '{nombre}' agregado exitosamente.")
        self.guardar_inventario()

    def eliminar_producto(self, codigo):
        """
        Elimina un producto del inventario según su código.
        Si no existe, se notifica. Luego guarda los cambios.
        """
        if codigo not in self.productos:
            print(f"No se encontró el producto con código '{codigo}'.")
            return

        producto_eliminado = self.productos.pop(codigo)
        print(f"Producto '{producto_eliminado.nombre}' eliminado exitosamente.")
        self.guardar_inventario()

    def actualizar_producto(self, codigo, nombre=None, cantidad=None, precio=None):
        """
        Actualiza la información de un producto (nombre, cantidad o precio).
        Si el producto no existe, se notifica. Luego guarda los cambios.
        """
        if codigo not in self.productos:
            print(f"No se encontró el producto con código '{codigo}'.")
            return

        producto = self.productos[codigo]
        if nombre is not None:
            producto.nombre = nombre
        if cantidad is not None:
            producto.cantidad = cantidad
        if precio is not None:
            producto.precio = precio

        print(f"Producto '{codigo}' actualizado exitosamente.")
        self.guardar_inventario()

    def mostrar_inventario(self):
        """
        Muestra en consola todos los productos del inventario.
        """
        if not self.productos:
            print("El inventario está vacío.")
            return

        print("\n--- Inventario de la Tienda de Víveres ---")
        for producto in self.productos.values():
            print(f"Código: {producto.codigo}, "
                  f"Nombre: {producto.nombre}, "
                  f"Cantidad: {producto.cantidad}, "
                  f"Precio: {producto.precio}")
        print("------------------------------------------\n")


def menu():
    """
    Muestra un menú de opciones en la consola y ejecuta las acciones correspondientes
    para gestionar la Tienda de Víveres.
    """
    inventario = Inventario()  # Se carga o crea el archivo 'inventario.txt'

    # Si el inventario está vacío, cargamos algunos productos predeterminados de víveres.
    if not inventario.productos:
        print("Cargando productos predeterminados para la Tienda de Víveres...")
        productos_iniciales = [
            ("1", "Arroz", 50, 1.50),
            ("2", "Frijol", 30, 2.00),
            ("3", "Azúcar", 20, 0.80),
            ("4", "Aceite", 15, 3.50),
            ("5", "Harina", 25, 1.20)
        ]
        for codigo, nombre, cantidad, precio in productos_iniciales:
            inventario.agregar_producto(codigo, nombre, cantidad, precio)

    while True:
        print("=== Sistema de Gestión de Inventarios - Tienda de Víveres ===")
        print("1. Agregar producto")
        print("2. Eliminar producto")
        print("3. Actualizar producto")
        print("4. Mostrar inventario")
        print("5. Salir")
        opcion = input("Seleccione una opción: ")

        if opcion == "1":
            codigo = input("Ingrese el código del producto: ")
            nombre = input("Ingrese el nombre del producto: ")
            try:
                cantidad = int(input("Ingrese la cantidad: "))
                precio = float(input("Ingrese el precio: "))
            except ValueError:
                print("Error: La cantidad y el precio deben ser numéricos.")
                continue

            inventario.agregar_producto(codigo, nombre, cantidad, precio)

        elif opcion == "2":
            codigo = input("Ingrese el código del producto a eliminar: ")
            inventario.eliminar_producto(codigo)

        elif opcion == "3":
            codigo = input("Ingrese el código del producto a actualizar: ")
            print("Deje en blanco cualquier campo que no desee modificar.")
            nuevo_nombre = input("Nuevo nombre (opcional): ")
            nueva_cantidad = input("Nueva cantidad (opcional): ")
            nuevo_precio = input("Nuevo precio (opcional): ")

            # Convertir a tipos numéricos solo si el usuario ingresó algo
            nueva_cantidad = int(nueva_cantidad) if nueva_cantidad.strip() else None
            nuevo_precio = float(nuevo_precio) if nuevo_precio.strip() else None

            inventario.actualizar_producto(
                codigo,
                nombre=nuevo_nombre if nuevo_nombre.strip() else None,
                cantidad=nueva_cantidad,
                precio=nuevo_precio
            )

        elif opcion == "4":
            inventario.mostrar_inventario()

        elif opcion == "5":
            print("Saliendo del sistema...")
            break

        else:
            print("Opción inválida. Por favor, intente de nuevo.")


if __name__ == "__main__":
    menu()

