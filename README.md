
import tkinter as tk
from tkinter import messagebox, simpledialog
import chess


class JuegoAjedrez:

    def __init__(self, ventana):

        self.ventana = ventana
        self.ventana.title("Juego de Ajedrez")
        self.ventana.geometry("750x720")
        self.ventana.configure(bg="#202B3C")
        self.ventana.resizable(True, True)
        self.ventana.minsize(620, 650)

        # Crear tablero
        self.tablero = chess.Board()

        # Casilla seleccionada
        self.seleccion = None

        # Historial de movimientos
        self.historial = []

        # Simbolos de las piezas
        self.piezas = {
            "K": "♔", "Q": "♕",
            "R": "♖", "B": "♗",
            "N": "♘", "P": "♙",
            "k": "♚", "q": "♛",
            "r": "♜", "b": "♝",
            "n": "♞", "p": "♟"
        }

        # Titulo
        tk.Label(
            ventana,
            text="JUEGO DE AJEDREZ",
            font=("Arial", 22, "bold"),
            bg="#202B3C",
            fg="white"
        ).pack(pady=10)

        # Informacion del turno
        self.etiqueta_turno = tk.Label(
            ventana,
            text="Turno: Blancas",
            font=("Arial", 15, "bold"),
            bg="#202B3C",
            fg="#2ECC71"
        )
        self.etiqueta_turno.pack(pady=5)

        # Marco del tablero
        self.marco = tk.Frame(ventana)
        self.marco.pack(pady=10)

        self.casillas = {}

        # Crear las 64 casillas
        for fila in range(8):
            for columna in range(8):

                casilla = tk.Button(
                    self.marco,
                    text="",
                    font=("Segoe UI Symbol", 26),
                    width=2,
                    height=1,
                    padx=0,
                    pady=0,
                    bd=0,
                    highlightthickness=0,
                    relief="flat",
                    command=lambda f=fila, c=columna:
                    self.seleccionar_casilla(f, c)
                )

                casilla.grid(row=fila, column=columna)

                self.casillas[(fila, columna)] = casilla

        # Historial visual
        tk.Label(
            ventana,
            text="Historial de movimientos",
            font=("Arial", 12, "bold"),
            bg="#202B3C",
            fg="white"
        ).pack()

        self.texto_historial = tk.Label(
            ventana,
            text="La partida ha comenzado.",
            font=("Arial", 10),
            bg="#202B3C",
            fg="white",
            wraplength=650
        )
        self.texto_historial.pack(pady=5)

        # Marco de botones
        marco_botones = tk.Frame(
            ventana,
            bg="#202B3C"
        )
        marco_botones.pack(pady=10)

        tk.Button(
            marco_botones,
            text="DESHACER",
            font=("Arial", 11, "bold"),
            bg="#3498DB",
            fg="white",
            command=self.deshacer
        ).grid(row=0, column=0, padx=10)

        tk.Button(
            marco_botones,
            text="REINICIAR",
            font=("Arial", 11, "bold"),
            bg="#E67E22",
            fg="white",
            command=self.reiniciar
        ).grid(row=0, column=1, padx=10)

        tk.Button(
            marco_botones,
            text="SALIR",
            font=("Arial", 11, "bold"),
            bg="#E74C3C",
            fg="white",
            command=ventana.destroy
        ).grid(row=0, column=2, padx=10)

        # Dibujar tablero inicial
        self.actualizar_tablero()

    # =================================
    # ACTUALIZAR TABLERO
    # =================================

    def actualizar_tablero(self):

        for fila in range(8):
            for columna in range(8):

                casilla = self.casillas[(fila, columna)]

                # Convertir fila y columna a casilla de ajedrez
                posicion = chess.square(
                    columna,
                    7 - fila
                )

                pieza = self.tablero.piece_at(posicion)

                # Colores del tablero
                if (fila + columna) % 2 == 0:
                    color = "#F0D9B5"
                else:
                    color = "#B58863"

                # Mostrar pieza
                if pieza:
                    simbolo = self.piezas[pieza.symbol()]
                    casilla.config(text=simbolo)
                else:
                    casilla.config(text="")

                casilla.config(
                    bg=color,
                    activebackground=color,
                    fg="#171717"
                )

                # Resaltar casilla seleccionada
                if self.seleccion == posicion:
                    casilla.config(
                        bg="#F1C40F",
                        activebackground="#F1C40F"
                    )

                # Mostrar movimientos posibles
                if self.seleccion is not None:

                    for movimiento in self.tablero.legal_moves:

                        if movimiento.from_square == self.seleccion:

                            if movimiento.to_square == posicion:
                                casilla.config(
                                    bg="#82E0AA",
                                    activebackground="#82E0AA"
                                )

        self.actualizar_turno()
        self.actualizar_historial()

    # =================================
    # SELECCIONAR Y MOVER PIEZAS
    # =================================

    def seleccionar_casilla(self, fila, columna):

        if self.tablero.is_game_over(claim_draw=True):
            return

        posicion = chess.square(
            columna,
            7 - fila
        )

        pieza = self.tablero.piece_at(posicion)

        # Si no hay una pieza seleccionada
        if self.seleccion is None:

            if pieza and pieza.color == self.tablero.turn:

                self.seleccion = posicion
                self.actualizar_tablero()

            return

        # Si se selecciona otra pieza del mismo jugador
        if pieza and pieza.color == self.tablero.turn:

            self.seleccion = posicion
            self.actualizar_tablero()
            return

        # Intentar realizar el movimiento
        movimiento = chess.Move(
            self.seleccion,
            posicion
        )

        # Promocion del peon
        pieza_seleccionada = self.tablero.piece_at(
            self.seleccion
        )

        if (
            pieza_seleccionada
            and pieza_seleccionada.piece_type == chess.PAWN
            and chess.square_rank(posicion) in (0, 7)
        ):

            opcion = simpledialog.askstring(
                "Promoción",
                "Elige una pieza:\n"
                "q = Dama\n"
                "r = Torre\n"
                "b = Alfil\n"
                "n = Caballo",
                initialvalue="q"
            )

            if opcion is None:
                return

            opcion = opcion.lower().strip()

            if opcion not in ["q", "r", "b", "n"]:
                messagebox.showerror(
                    "Error",
                    "Opción de promoción no válida."
                )
                return

            movimiento = chess.Move(
                self.seleccion,
                posicion,
                promotion={
                    "q": chess.QUEEN,
                    "r": chess.ROOK,
                    "b": chess.BISHOP,
                    "n": chess.KNIGHT
                }[opcion]
            )

        # Verificar si el movimiento es legal
        if movimiento in self.tablero.legal_moves:

            # Guardar movimiento antes de realizarlo
            notacion = self.tablero.san(movimiento)

            self.tablero.push(movimiento)

            self.historial.append(notacion)

            self.seleccion = None

            self.actualizar_tablero()

            self.comprobar_partida()

        else:

            self.seleccion = None

            self.actualizar_tablero()

    # =================================
    # ACTUALIZAR TURNO
    # =================================

    def actualizar_turno(self):

        if self.tablero.turn == chess.WHITE:
            turno = "Turno: Blancas"
        else:
            turno = "Turno: Negras"

        if self.tablero.is_check():
            turno += " - JAQUE"

        self.etiqueta_turno.config(text=turno)

    # =================================
    # ACTUALIZAR HISTORIAL
    # =================================

    def actualizar_historial(self):

        movimientos = []

        for i in range(0, len(self.historial), 2):

            numero = (i // 2) + 1

            blancas = self.historial[i]

            if i + 1 < len(self.historial):
                negras = self.historial[i + 1]
                movimientos.append(
                    f"{numero}. {blancas} {negras}"
                )
            else:
                movimientos.append(
                    f"{numero}. {blancas}"
                )

        texto = " | ".join(movimientos)

        if not texto:
            texto = "La partida ha comenzado."

        self.texto_historial.config(text=texto)

    # =================================
    # COMPROBAR RESULTADO
    # =================================

    def comprobar_partida(self):

        if self.tablero.is_checkmate():

            ganador = (
                "Negras"
                if self.tablero.turn == chess.WHITE
                else "Blancas"
            )

            messagebox.showinfo(
                "Fin del juego",
                f"¡Jaque mate! Ganaron las {ganador}."
            )

        elif self.tablero.is_stalemate():

            messagebox.showinfo(
                "Fin del juego",
                "Empate por ahogado."
            )

        elif self.tablero.is_insufficient_material():

            messagebox.showinfo(
                "Fin del juego",
                "Empate por material insuficiente."
            )

        elif self.tablero.is_fivefold_repetition():

            messagebox.showinfo(
                "Fin del juego",
                "Empate por repetición."
            )

        elif self.tablero.is_seventyfive_moves():

            messagebox.showinfo(
                "Fin del juego",
                "Empate por la regla de los 75 movimientos."
            )

    # =================================
    # DESHACER MOVIMIENTO
    # =================================

    def deshacer(self):

        if self.tablero.move_stack:

            self.tablero.pop()

            self.historial.pop()

            self.seleccion = None

            self.actualizar_tablero()

    # =================================
    # REINICIAR PARTIDA
    # =================================

    def reiniciar(self):

        respuesta = messagebox.askyesno(
            "Reiniciar",
            "¿Deseas comenzar una nueva partida?"
        )

        if respuesta:

            self.tablero.reset()

            self.historial.clear()

            self.seleccion = None

            self.actualizar_tablero()


# =================================
# INICIAR PROGRAMA
# =================================

if __name__ == "__main__":

    ventana = tk.Tk()

    juego = JuegoAjedrez(ventana)

    ventana.mainloop()

