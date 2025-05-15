mport re
import random
from datetime import time, datetime

# Base de datos de vuelos
vuelos = {
    "AV-101": {"origen": "Lima", "destino": "Bogotá", "asientos": ["A1", "A2", "B1", "B2"], "horario": (15, 30)},
    "LA-202": {"origen": "Quito", "destino": "Medellín", "asientos": ["A1", "A2", "B1", "B2"], "horario": (10, 45)},
}

# Reservas registradas
reservas = {}

# Excepciones personalizadas
class CodigoVueloInvalido(Exception): pass
class AsientoInvalido(Exception): pass
class AsientoOcupado(Exception): pass
class HorarioInvalido(Exception): pass

# Validadores
def validar_codigo_vuelo(codigo):
    if not re.match(r"^[A-Z]{2}-\d{3}$", codigo):
        raise CodigoVueloInvalido("Código de vuelo inválido (ejemplo: XX-123)")

def validar_horario(hora, minuto):
    if not (0 <= hora <= 23 and 0 <= minuto <= 59):
        raise HorarioInvalido("Horario inválido")

def validar_asiento(asiento):
    if not re.match(r"^[A-Z]\d+$", asiento):
        raise AsientoInvalido("Formato de asiento inválido (ejemplo: A1)")

# Funciones principales
def reservar_asiento(codigo_vuelo, asiento, nombre_cliente):
    validar_codigo_vuelo(codigo_vuelo)
    validar_asiento(asiento)

    vuelo = vuelos.get(codigo_vuelo)
    if not vuelo:
        raise ValueError("Vuelo no encontrado")

    if asiento not in vuelo["asientos"]:
        raise AsientoOcupado("El asiento ya está reservado o no existe")

    # Crear reserva
    codigo_reserva = f"R-{random.randint(1000, 9999)}"
    reservas[codigo_reserva] = {
        "cliente": nombre_cliente,
        "vuelo": codigo_vuelo,
        "asiento": asiento,
        "fecha": datetime.now().strftime("%Y-%m-%d %H:%M")
    }
    vuelo["asientos"].remove(asiento)
    print(f"Reserva exitosa. Código de reserva: {codigo_reserva}")
    return codigo_reserva

def porcentaje_ocupacion(codigo_vuelo):
    total = 4  # Asumimos cada vuelo empieza con 4 asientos
    ocupados = total - len(vuelos[codigo_vuelo]["asientos"])
    return (ocupados / total) * 100

def generar_reporte_txt(nombre_archivo="reporte_vuelos.txt"):
    ordenados = sorted(vuelos.items(), key=lambda v: v[1]["horario"])
    with open(nombre_archivo, "w") as file:
        for cod, info in ordenados:
            hora = f"{info['horario'][0]:02d}:{info['horario'][1]:02d}"
            ocupacion = porcentaje_ocupacion(cod)
            file.write(f"Vuelo: {cod} | Origen: {info['origen']} | Destino: {info['destino']} | Horario: {hora} | Ocupación: {ocupacion:.2f}%\n")
    print(f"Reporte generado en {nombre_archivo}")

# Simulación
try:
    vuelo = input("Código del vuelo: ").strip().upper()
    asiento = input("Asiento a reservar: ").strip().upper()
    nombre = input("Nombre del cliente: ").strip()
    reservar_asiento(vuelo, asiento, nombre)
    generar_reporte_txt()
except Exception as e:
    print(f"Error: {e}")
