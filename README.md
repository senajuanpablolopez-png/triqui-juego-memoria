import random
#Diego Fernando Asprilla Palacios, Juan Pablo Lopez Salazar, Juan Manuel Barrera Alvarado y Alvaro Jossue Obando Chaux

#MENÚ PRINCIPAL
def menu():
    print("\n= MENÚ DE JUEGOS =")
    print("1) Triqui")
    print("2) Memoria")
    print("0) Salir")
    op = validacion_de_entradas("Opción (0-2): ", 0, 2)
    return op

# Permite ingresar un número dentro de un rango o escribir "salir"
def validacion_de_entradas(msg, minimo, maximo):
    while True:
        x = input(msg).strip()
        if x.lower() == "salir":
            return "salir"
        if x.isdigit():
            x = int(x)
            if minimo <= x <= maximo:
                return x
        print(f" Entrada inválida. Debe ser un número entre {minimo} y {maximo} o 'salir'.")
#TRIQUI
def leer_simbolo_xo():
    while True:
        s = input("Elige tu símbolo (X/O): ").strip().upper()
        if s.lower() == "salir":
            return "salir"
        if s in ["X", "O"]:
            return s
        print("❗ Solo se permite 'X' o 'O' o 'salir'.")

def mostrar_triqui(t):
    print("   1   2   3")
    for f in range(3):
        fila = " | ".join(t[f])
        print(f"{f+1}  {fila}")
        if f < 2:
            print("  ---+---+---")

# Revisa si un símbolo (X u O) logró tres en línea
def hay_3_en_linea(t, s):
    return any([
        all(t[f][c] == s for c in range(3)) for f in range(3)
    ]) or any([
        all(t[f][c] == s for f in range(3)) for c in range(3)
    ]) or all(t[i][i] == s for i in range(3)) or all(t[i][2-i] == s for i in range(3))

def verificar_triqui(t, su, sm):
    if hay_3_en_linea(t, su): return "GANA_USUARIO"
    if hay_3_en_linea(t, sm): return "GANA_MAQUINA"
    if any(" " in fila for fila in t): return "EN_CURSO"
    return "EMPATE"

def jugada_maquina_triq(t, sm, su):
  #Gana si puede
    for f in range(3):
        for c in range(3):
            if t[f][c] == " ":
                t[f][c] = sm
                if hay_3_en_linea(t, sm):
                    t[f][c] = " "
                    return f, c
                t[f][c] = " "
  #Bloquea al usuario si este puede ganar
    for f in range(3):
        for c in range(3):
            if t[f][c] == " ":
                t[f][c] = su
                if hay_3_en_linea(t, su):
                    t[f][c] = " "
                    return f, c
                t[f][c] = " "
  #Ocupa el centro si esta libre
    if t[1][1] == " ": return 1, 1
  #Si no, esquinas y bordes
    for f, c in [(0,0),(0,2),(2,0),(2,2),(0,1),(1,0),(1,2),(2,1)]:
        if t[f][c] == " ": return f, c
    return 0, 0

def triqui():
    t = [[" "]*3 for _ in range(3)]
    simU = leer_simbolo_xo()
    if simU == "salir": return
    simM = "O" if simU == "X" else "X"
    turno = "U" if random.randint(0,1) == 0 else "M"
    est = "EN_CURSO"

    while est == "EN_CURSO":
        mostrar_triqui(t)
        if turno == "U":
            f = validacion_de_entradas("Fila (1-3): ",1,3)
            if f == "salir": return
            c = validacion_de_entradas("Columna (1-3): ",1,3)
            if c == "salir": return
            if t[f-1][c-1] != " ":
                print(" Posición ocupada, intenta otra.")
                continue
            t[f-1][c-1] = simU
        else:
            f,c = jugada_maquina_triq(t, simM, simU)
            t[f][c] = simM

        r = verificar_triqui(t, simU, simM)
        if r != "EN_CURSO":
            est = r
        else:
            turno = "M" if turno == "U" else "U"

    mostrar_triqui(t)
    if est == "GANA_USUARIO": print(" Ganaste!!!")
    elif est == "GANA_MAQUINA": print(" Perdiste!!!")
    else: print(" Empate")

#MEMORIA
def letra(i): return chr(ord("A")+i-1) # Convierte número en letra (1→A, 2→B...)

def barajar_lista(v): random.shuffle(v) # Mezcla la lista al azar

def tablero_memoria(NF,NC):
    valores=[]
    for i in range(1, (NF*NC)//2 + 1):
        valores.append(letra(i)); valores.append(letra(i))
    random.shuffle(valores)
    tab=[[None]*NC for _ in range(NF)]
    est=[["OCULTA"]*NC for _ in range(NF)]
    k=0
    for f in range(NF):
        for c in range(NC):
            tab[f][c]=valores[k]; k+=1
    return tab, est

def mostrar_memoria(tab,est,vis):
    print("   " + " ".join(str(c+1) for c in range(len(tab[0]))))
    for f in range(len(tab)):
        linea=f"{f+1}  "
        for c in range(len(tab[0])):
            if est[f][c]=="FIJA" or (f,c) in vis:
                linea+=tab[f][c]+" "
            else:
                linea+="* "
        print(linea)

# Devuelve una posición aleatoria que aún está oculta
def oculto_aleatorio(est):
    while True:
        f=random.randint(0,len(est)-1)
        c=random.randint(0,len(est[0])-1)
        if est[f][c]=="OCULTA": return f,c

# La máquina selecciona dos posiciones ocultas aleatorias
def jugada_maquina_mem(tab,est):
    f1,c1=oculto_aleatorio(est)
    while True:
        f2,c2=oculto_aleatorio(est)
        if (f1,c1)!=(f2,c2): break
    return f1,c1,f2,c2

def verificacion_pareja(a,b): return a==b

# Retorna True si todas las casillas están fijas (todas descubiertas)
def juego_memoria_terminado(est):
    return all(all(c=="FIJA" for c in fila) for fila in est)

def calcular_ganador(pu,pm):
    if pu>pm: print(" Ganador: Usuario")
    elif pm>pu: print(" Ganador: Máquina")
    else: print(" Empate")

def memoria():
    NF,NC=6,5
    tab,est=tablero_memoria(NF,NC)
    pu=pm=0; turno="U"

    while not juego_memoria_terminado(est):
        mostrar_memoria(tab,est,[])
        if turno=="U":
            f1=validacion_de_entradas("Fila1 (1-6): ",1,NF)
            if f1=="salir": return
            c1=validacion_de_entradas("Col1 (1-5): ",1,NC)
            if c1=="salir": return
            if est[f1-1][c1-1]!="OCULTA":
                print(" Casilla no válida.")
                continue

            f2=validacion_de_entradas("Fila2 (1-6): ",1,NF)
            if f2=="salir": return
            c2=validacion_de_entradas("Col2 (1-5): ",1,NC)
            if c2=="salir": return
            if (f1==f2 and c1==c2) or est[f2-1][c2-1]!="OCULTA":
                print(" Casilla no válida.")
                continue

            mostrar_memoria(tab,est,[(f1-1,c1-1),(f2-1,c2-1)])
            if verificacion_pareja(tab[f1-1][c1-1],tab[f2-1][c2-1]):
                est[f1-1][c1-1]=est[f2-1][c2-1]="FIJA"
                pu+=1
                print(" Usuario acertó un par!")
            else:
                print(" Usuario no acertó.")
                turno="M"
        else:
            f1,c1,f2,c2=jugada_maquina_mem(tab,est)
            mostrar_memoria(tab,est,[(f1,c1),(f2,c2)])
            if verificacion_pareja(tab[f1][c1],tab[f2][c2]):
                est[f1][c1]=est[f2][c2]="FIJA"
                pm+=1
                print(" Máquina acertó un par!")
            else:
                print(" Máquina no acertó.")
                turno="U"

    calcular_ganador(pu,pm)

#PRINCIPAL

def main():
    while True:
        op=menu()
        if op=="salir":
            print(" Saliendo al menú principal...")
            continue
        if op==1: triqui()
        elif op==2: memoria()
        elif op==0:
            print(" Gracias por jugar")
            break

main()
 
