from datetime import date, datetime, time, timedelta

from hospital import (Paciente, Afiliacion, PlanSalud, Pago, Factura, Medico,
                      PlanTratamiento, Prescripcion, Dosis, Medicamento,
                      CategoriaMedicamento, EvolucionClinica, Sede, Sala,
                      Horario, Jornada, Cita, Atencion, EquipoMedico,
                      Mantenimiento)

# Los objetos viven en memoria: una lista por cada clase del sistema.
NOMBRES = ["Paciente", "Afiliacion", "PlanSalud", "Pago", "Factura", "Medico",
           "PlanTratamiento", "Prescripcion", "Dosis", "Medicamento",
           "CategoriaMedicamento", "EvolucionClinica", "Sede", "Sala",
           "Horario", "Jornada", "Cita", "Atencion", "EquipoMedico",
           "Mantenimiento"]
CLASES = {n: globals()[n] for n in NOMBRES}
datos = {n: [] for n in NOMBRES}


# ---------------------------------------------------------------- lectura
def pedirTexto(mensaje):
    while True:
        texto = input(mensaje).strip()
        if texto:
            return texto
        print("  El campo no puede estar vacío.")


def pedirEntero(mensaje):
    while True:
        try:
            return int(input(mensaje).strip())
        except ValueError:
            print("  Escriba un número entero.")


def pedirDecimal(mensaje):
    while True:
        try:
            return float(input(mensaje).strip().replace(",", "."))
        except ValueError:
            print("  Escriba un número.")


def pedirHora(mensaje):
    while True:
        try:
            h, m = input(mensaje).strip().split(":")
            return time(int(h), int(m))
        except ValueError:
            print("  Use el formato HH:MM, por ejemplo 08:30.")


def pedirSiNo(mensaje):
    return input(mensaje + " (s/n): ").strip().lower() == "s"


def elegir(clase, etiqueta=None):
    """Muestra los objetos de una clase y devuelve el que escoja el usuario."""
    lista = datos[clase]
    if not lista:
        print(f"  Primero cree al menos un objeto de {clase}.")
        return None
    for i, obj in enumerate(lista, 1):
        print(f"   {i}. {obj}")
    while True:
        n = pedirEntero(f"  Elija {etiqueta or clase} (número): ")
        if 1 <= n <= len(lista):
            return lista[n - 1]
        print("  Número fuera de la lista.")


def guardar(obj):
    datos[type(obj).__name__].append(obj)
    return obj


# --------------------------------------------------------------- creación
def crearPaciente():
    return guardar(Paciente(pedirTexto("  Nombre: "), pedirTexto("  Teléfono: "),
                            pedirTexto("  Email: ")))


def crearAfiliacion():
    paciente = elegir("Paciente")
    plan = paciente and elegir("PlanSalud")
    if not plan:
        return None
    return guardar(paciente.afiliarse(plan))


def crearPlanSalud():
    return guardar(PlanSalud(pedirTexto("  Nombre: "), pedirEntero("  Duración (días): "),
                             pedirDecimal("  Precio: "), pedirTexto("  Coberturas: ")))


def crearPago():
    paciente = elegir("Paciente")
    if not paciente:
        return None
    pago = guardar(Pago(date.today(), pedirDecimal("  Valor: "), pedirTexto("  Método (efectivo/tarjeta): "), paciente))
    paciente.agregarPago(pago)
    if pedirSiNo("  ¿Procesar el pago ahora?"):
        if pago.procesar(pago.metodo):
            factura = guardar(pago.factura)
            print(f"  Pago procesado. Se generó la factura número {factura.numero}.")
    return pago


def crearFactura():
    pendientes = [p for p in datos["Pago"] if p.estado == "Pendiente"]
    if not pendientes:
        print("  Las facturas se generan al procesar un pago pendiente; no hay ninguno.")
        return None
    for i, p in enumerate(pendientes, 1):
        print(f"   {i}. {p}")
    n = pedirEntero("  Pago a procesar (número): ")
    if not 1 <= n <= len(pendientes):
        print("  Número fuera de la lista.")
        return None
    pago = pendientes[n - 1]
    pago.procesar(pago.metodo)
    return guardar(pago.factura)


def crearMedico():
    return guardar(Medico(pedirTexto("  Nombre: "), pedirTexto("  Especialidad: "),
                          pedirTexto("  Registro médico: ")))


def crearPlanTratamiento():
    medico = elegir("Medico")
    paciente = medico and elegir("Paciente")
    if not paciente:
        return None
    plan = medico.crearPlan(paciente)
    plan.nombre = pedirTexto("  Nombre del plan: ")
    plan.objetivo = pedirTexto("  Objetivo clínico: ")
    return guardar(plan)


def crearPrescripcion():
    plan = elegir("PlanTratamiento")
    if not plan:
        return None
    pres = guardar(Prescripcion(pedirTexto("  Nombre: "), pedirTexto("  Día de la semana: "),
                                pedirTexto("  Indicación: ")))
    plan.agregarPrescripcion(pres)
    return pres


def crearMedicamento():
    med = guardar(Medicamento(pedirTexto("  Nombre: "), pedirTexto("  Descripción: "),
                              pedirTexto("  Vía de administración: ")))
    if datos["CategoriaMedicamento"] and pedirSiNo("  ¿Asignar una categoría?"):
        elegir("CategoriaMedicamento").agregarMedicamento(med)
    return med


def crearCategoriaMedicamento():
    return guardar(CategoriaMedicamento(pedirTexto("  Nombre: "), pedirTexto("  Grupo terapéutico: ")))


def crearDosis():
    pres = elegir("Prescripcion")
    med = pres and elegir("Medicamento")
    if not med:
        return None
    dosis = guardar(Dosis(pedirDecimal("  Cantidad (mg): "), pedirEntero("  Veces por día: "),
                          pedirEntero("  Duración (días): "), med))
    pres.agregarDosis(dosis)
    return dosis


def crearEvolucionClinica():
    paciente = elegir("Paciente")
    if not paciente:
        return None
    ev = guardar(EvolucionClinica(date.today(), pedirEntero("  Frecuencia cardíaca (lpm): "),
                                  pedirTexto("  Observaciones: "), paciente))
    ev.registrarEvolucion(date.today())
    return ev


def crearSede():
    return guardar(Sede(pedirTexto("  Nombre: "), pedirTexto("  Dirección: "), pedirTexto("  Teléfono: ")))


def crearSala():
    sede = elegir("Sede")
    if not sede:
        return None
    sala = guardar(Sala(pedirTexto("  Nombre: "), pedirEntero("  Capacidad: ")))
    sede.agregarSala(sala)
    return sala


def crearHorario():
    return guardar(Horario(pedirTexto("  Día: "), pedirHora("  Hora de inicio (HH:MM): "),
                           pedirHora("  Hora de fin (HH:MM): ")))


def crearJornada():
    medico = elegir("Medico")
    sala = medico and elegir("Sala")
    horario = sala and elegir("Horario")
    if not horario:
        return None
    jornada = guardar(Jornada(pedirTexto("  Nombre: "), pedirEntero("  Cupo máximo: ")))
    if not sala.asignarJornada(jornada):
        print("  El cupo máximo no cabe en la capacidad de la sala.")
        datos["Jornada"].remove(jornada)
        return None
    horario.agregarJornada(jornada)
    medico.atenderJornada(jornada)
    if pedirSiNo("  ¿Abrir las citas ahora?"):
        jornada.abrirCitas()
    return jornada


def crearCita():
    paciente = elegir("Paciente")
    jornada = paciente and elegir("Jornada")
    if not jornada:
        return None
    antes = len(paciente.citas)
    if paciente.solicitarCita(jornada, date.today()):
        return guardar(paciente.citas[antes])
    print("  No se pudo crear: se necesita una afiliación vigente, "
          "jornada abierta y cupos disponibles.")
    return None


def crearAtencion():
    cita = elegir("Cita")
    if not cita:
        return None
    at = guardar(Atencion(datetime.now(), pedirSiNo("  ¿El paciente asistió?"), cita))
    cita.registrarAtencion(at)
    return at


def crearEquipoMedico():
    sala = elegir("Sala")
    if not sala:
        return None
    eq = guardar(EquipoMedico(pedirTexto("  Nombre: "), pedirTexto("  Código de activo: "),
                              "Operativo"))
    sala.agregarEquipo(eq)
    return eq


def crearMantenimiento():
    equipo = elegir("EquipoMedico")
    if not equipo:
        return None
    m = guardar(Mantenimiento(date.today(), pedirTexto("  Tipo (Preventivo/Correctivo): "),
                              pedirDecimal("  Costo: "), equipo))
    equipo.agregarMantenimiento(m)
    return m


CREADORES = {n: globals()["crear" + n] for n in NOMBRES}


# ------------------------------------------------- crear / modificar / imprimir
def crear(nombre):
    obj = CREADORES[nombre]()
    if obj is not None:
        print(f"  Creado: {obj}")


def campos_modificables(clase):
    return [n for n, p in vars(clase).items() if isinstance(p, property) and p.fset]


def modificar(nombre):
    obj = elegir(nombre)
    if obj is None:
        return
    campos = campos_modificables(CLASES[nombre])
    for i, campo in enumerate(campos, 1):
        print(f"   {i}. {campo} (actual: {getattr(obj, campo)})")
    n = pedirEntero("  Campo a modificar (número): ")
    if not 1 <= n <= len(campos):
        print("  Número fuera de la lista.")
        return
    campo = campos[n - 1]
    actual = getattr(obj, campo)
    if isinstance(actual, bool):
        nuevo = pedirSiNo("  Nuevo valor")
    elif isinstance(actual, int):
        nuevo = pedirEntero("  Nuevo valor: ")
    elif isinstance(actual, float):
        nuevo = pedirDecimal("  Nuevo valor: ")
    elif isinstance(actual, time):
        nuevo = pedirHora("  Nuevo valor (HH:MM): ")
    elif isinstance(actual, str):
        nuevo = pedirTexto("  Nuevo valor: ")
    else:
        print("  Este campo se cambia desde las relaciones, no desde aquí.")
        return
    setattr(obj, campo, nuevo)        # pasa por el setter de la clase
    print(f"  Modificado: {obj}")


def imprimir(nombre):
    if not datos[nombre]:
        print(f"  No hay objetos de {nombre}.")
    for obj in datos[nombre]:
        print("  " + str(obj))        # polimorfismo: cada clase tiene su __str__


def submenu(nombre):
    while True:
        print(f"\n--- {nombre} ---")
        print("1. Crear\n2. Modificar\n3. Imprimir\n0. Volver")
        op = input("Opción: ").strip()
        if op == "1":
            crear(nombre)
        elif op == "2":
            modificar(nombre)
        elif op == "3":
            imprimir(nombre)
        elif op == "0":
            return
        else:
            print("  Opción no válida.")


# ---------------------------------------------------------------- ejemplo
def cargarEjemplo():
    sede = guardar(Sede("Hospital Central de Bucaramanga", "Cra 27 # 10-50", "6071234567"))
    sala1 = guardar(Sala("Consultorio 1", 25)); sede.agregarSala(sala1)
    sala2 = guardar(Sala("Sala de urgencias", 40)); sede.agregarSala(sala2)
    hor = guardar(Horario("Lunes", time(8, 0), time(12, 0)))
    dra = guardar(Medico("Laura Mendoza", "Medicina general", "RM-1001"))
    dr = guardar(Medico("Andrés Ríos", "Cardiología", "RM-1002"))
    p1 = guardar(Paciente("Maria Gomez", "3109876543", "maria@correo.com"))
    p2 = guardar(Paciente("Juan Perez", "3001112233", "juan@correo.com"))
    plan = guardar(PlanSalud("Plan Familiar Anual", 365, 1200000.0, "Consulta, urgencias y hospitalización"))
    guardar(p1.afiliarse(plan)); guardar(p2.afiliarse(plan))
    jor = guardar(Jornada("Vacunación contra la influenza", 20))
    sala1.asignarJornada(jor); hor.agregarJornada(jor); dra.atenderJornada(jor)
    jor.abrirCitas()
    eq = guardar(EquipoMedico("Monitor de signos vitales", "ACT-0001", "Operativo")); sala2.agregarEquipo(eq)
    mant = guardar(Mantenimiento(date.today() + timedelta(days=7), "Preventivo", 150000.0, eq)); eq.agregarMantenimiento(mant)
    cat = guardar(CategoriaMedicamento("Analgésicos", "Manejo del dolor"))
    med = guardar(Medicamento("Acetaminofén", "Analgésico y antipirético", "Oral")); cat.agregarMedicamento(med)
    tr = guardar(dr.crearPlan(p1)); tr.objetivo = "Controlar la presión arterial"
    pres = guardar(Prescripcion("Control matutino", "Lunes", "Tomar con alimentos")); tr.agregarPrescripcion(pres)
    guardar(Dosis(500.0, 3, 5, med)); pres.agregarDosis(datos["Dosis"][0])
    ev = guardar(EvolucionClinica(date.today(), 72, "Paciente estable", p1)); ev.registrarEvolucion(date.today())
    if p1.solicitarCita(jor, date.today()):
        guardar(p1.citas[-1])
    print("  Datos de ejemplo cargados.")


# ------------------------------------------------------------------ menú
def menuPrincipal():
    while True:
        print("\n=== SISTEMA DE INFORMACIÓN HOSPITALARIO ===")
        for i, n in enumerate(NOMBRES, 1):
            print(f"{i:>2}. {n}")
        print("21. Cargar datos de ejemplo\n 0. Salir")
        op = input("Opción: ").strip()
        if op == "0":
            print("Hasta pronto.")
            return
        if op == "21":
            cargarEjemplo()
        elif op.isdigit() and 1 <= int(op) <= 20:
            submenu(NOMBRES[int(op) - 1])
        else:
            print("  Opción no válida.")


if __name__ == "__main__":
    menuPrincipal()
