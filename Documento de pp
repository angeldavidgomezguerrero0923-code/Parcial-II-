"""
personajes.py
=============
Definición de las clases de "Guardians of the Ancient Kingdom".

Este módulo SOLO define clases: no ejecuta pruebas ni imprime nada al ser
importado. La creación de objetos y los casos de prueba están en main.py.

Clases:
    - Personaje (abstracta): atributos y comportamientos comunes.
    - Guerrero, Mago, Arquero: cada uno implementa atacar() a su manera.
    - Combate: administra el enfrentamiento por turnos entre dos personajes.
"""

from abc import ABC, abstractmethod


class AtaqueInvalidoError(Exception):
    """Se lanza cuando un ataque no puede realizarse (atacante o objetivo derrotado)."""


# ---------------------------------------------------------------------------
# Clase abstracta
# ---------------------------------------------------------------------------
class Personaje(ABC):
    """Clase base abstracta de todos los guardianes de Eldoria."""

    VIDA_MAXIMA = 100

    def __init__(self, nombre, vida, ataque, defensa):
        # Se usan los setters para que el constructor valide con las mismas reglas.
        self.nombre = nombre
        self.vida = vida
        self.ataque = ataque
        self.defensa = defensa

    # ---- Helpers de validación ------------------------------------------
    @staticmethod
    def _es_entero(valor):
        # bool es subclase de int en Python; True/False no se aceptan como números.
        return isinstance(valor, int) and not isinstance(valor, bool)

    # ---- Getters y setters (encapsulamiento) ----------------------------
    @property
    def nombre(self):
        return self._nombre

    @nombre.setter
    def nombre(self, valor):
        if not isinstance(valor, str):
            raise TypeError("El nombre debe ser un texto.")
        if not valor.strip():
            raise ValueError("El nombre no puede estar vacío.")
        self._nombre = valor.strip()

    @property
    def vida(self):
        return self._vida

    @vida.setter
    def vida(self, valor):
        if not self._es_entero(valor):
            raise TypeError("La vida debe ser un número entero.")
        if not 0 <= valor <= self.VIDA_MAXIMA:
            raise ValueError(f"La vida debe estar entre 0 y {self.VIDA_MAXIMA}.")
        self._vida = valor

    @property
    def ataque(self):
        return self._ataque

    @ataque.setter
    def ataque(self, valor):
        if not self._es_entero(valor):
            raise TypeError("El ataque debe ser un número entero.")
        if valor <= 0:
            raise ValueError("El ataque debe ser un entero positivo.")
        self._ataque = valor

    @property
    def defensa(self):
        return self._defensa

    @defensa.setter
    def defensa(self, valor):
        if not self._es_entero(valor):
            raise TypeError("La defensa debe ser un número entero.")
        if valor < 0:
            raise ValueError("La defensa no puede ser negativa.")
        self._defensa = valor

    # ---- Comportamientos comunes ----------------------------------------
    def recibir_danio(self, cantidad):
        """
        Resta vida al personaje y devuelve el daño efectivamente recibido.

        La cantidad ya viene calculada (la defensa se descontó en atacar()),
        por eso aquí NO se vuelve a restar la defensa.
        """
        if not self._es_entero(cantidad):
            raise TypeError("El daño debe ser un número entero.")
        if cantidad < 0:
            raise ValueError("El daño no puede ser negativo.")
        danio_efectivo = min(cantidad, self._vida)  # la vida nunca baja de 0
        self._vida -= danio_efectivo
        return danio_efectivo

    def esta_vivo(self):
        return self._vida > 0

    def mostrar_estado(self):
        estado = "vivo" if self.esta_vivo() else "derrotado"
        return (f"{self.nombre} ({type(self).__name__}) | Vida: {self.vida} | "
                f"Ataque: {self.ataque} | Defensa: {self.defensa} | {estado}")

    def _validar_ataque(self, objetivo):
        """Validaciones comunes a los tres tipos de ataque."""
        if not isinstance(objetivo, Personaje):
            raise TypeError("El objetivo debe ser un Personaje.")
        if objetivo is self:
            raise ValueError("Un personaje no puede atacarse a sí mismo.")
        if not self.esta_vivo():
            raise AtaqueInvalidoError(f"{self.nombre} está derrotado y no puede atacar.")
        if not objetivo.esta_vivo():
            raise AtaqueInvalidoError(f"{objetivo.nombre} ya está derrotado.")

    @abstractmethod
    def atacar(self, objetivo):
        """Ataca al objetivo y devuelve el daño efectivo causado."""

    def __str__(self):
        return self.mostrar_estado()


# ---------------------------------------------------------------------------
# Subclases
# ---------------------------------------------------------------------------
class Guerrero(Personaje):
    """Golpe poderoso: ataque +20 % antes de descontar la defensa enemiga."""

    def atacar(self, objetivo):
        self._validar_ataque(objetivo)
        # ataque * 1.20 con enteros (evita errores de punto flotante); // redondea hacia abajo.
        ataque_potenciado = (self.ataque * 6) // 5
        danio = max(0, ataque_potenciado - objetivo.defensa)
        return objetivo.recibir_danio(danio)


class Mago(Personaje):
    """Hechizo penetrante: ignora por completo la defensa del objetivo."""

    def atacar(self, objetivo):
        self._validar_ataque(objetivo)
        return objetivo.recibir_danio(self.ataque)


class Arquero(Personaje):
    """Disparo crítico: si su ataque supera la defensa, duplica el daño base."""

    def atacar(self, objetivo):
        self._validar_ataque(objetivo)
        danio_base = max(0, self.ataque - objetivo.defensa)
        danio = danio_base * 2 if self.ataque > objetivo.defensa else 0
        return objetivo.recibir_danio(danio)


# ---------------------------------------------------------------------------
# Combate (agregación: administra personajes creados fuera de la clase)
# ---------------------------------------------------------------------------
class Combate:
    """Controla los turnos, detecta al ganador y cierra el combate."""

    MAX_RONDAS = 50

    VICTORIA = "victoria"
    EMPATE_TECNICO = "empate técnico"
    EMPATE_RONDAS = "empate por límite de rondas"

    def __init__(self, personaje1, personaje2):
        for p in (personaje1, personaje2):
            if not isinstance(p, Personaje):
                raise TypeError("Los participantes deben ser objetos Personaje.")
        if personaje1 is personaje2:
            raise ValueError("Un combate necesita dos personajes distintos.")
        if not (personaje1.esta_vivo() and personaje2.esta_vivo()):
            raise ValueError("Ambos personajes deben estar vivos para iniciar el combate.")
        self._participantes = [personaje1, personaje2]
        self._turno = 0          # índice del personaje que ataca
        self._rondas = 0         # rondas completadas
        self._resultado = None   # VICTORIA, EMPATE_TECNICO o EMPATE_RONDAS
        self._ganador = None

    @property
    def ganador(self):
        return self._ganador

    @property
    def resultado(self):
        return self._resultado

    @property
    def rondas(self):
        return self._rondas

    def cambiar_turno(self):
        """Alterna el personaje que debe atacar."""
        self._turno = 1 - self._turno

    def verificar_ganador(self):
        """Devuelve el personaje ganador, o None si todavía no hay vencedor."""
        p1, p2 = self._participantes
        if p1.esta_vivo() and not p2.esta_vivo():
            return p1
        if p2.esta_vivo() and not p1.esta_vivo():
            return p2
        return None

    def iniciar(self, verbose=True):
        """
        Ejecuta el combate por turnos y devuelve el resultado.

        Una ronda = un ataque de cada personaje. Se usa polimorfismo: solo se
        llama a atacar(), sin preguntar de qué tipo es cada personaje.
        """
        if self._resultado is not None:
            # El combate ya terminó: no se vuelve a jugar ni a contar rondas.
            return self._resultado

        def mostrar(texto=""):
            if verbose:
                print(texto)

        mostrar("=== INICIO DEL COMBATE ===")
        for p in self._participantes:
            mostrar(p.mostrar_estado())

        while self._resultado is None:
            danio_ronda = 0
            for _ in range(2):
                atacante = self._participantes[self._turno]
                objetivo = self._participantes[1 - self._turno]
                danio = atacante.atacar(objetivo)
                danio_ronda += danio
                mostrar(f"\nRonda {self._rondas + 1} - {atacante.nombre} ataca a "
                        f"{objetivo.nombre} y causa {danio} de daño.")
                for p in self._participantes:
                    mostrar("  " + p.mostrar_estado())

                self._ganador = self.verificar_ganador()
                if self._ganador is not None:
                    self._resultado = self.VICTORIA
                    break
                self.cambiar_turno()

            if self._resultado is not None:
                break
            self._rondas += 1
            if danio_ronda == 0:
                self._resultado = self.EMPATE_TECNICO
            elif self._rondas >= self.MAX_RONDAS:
                self._resultado = self.EMPATE_RONDAS

        if self._resultado == self.VICTORIA:
            self._rondas += 1  # la ronda en la que cayó un personaje también cuenta
        mostrar()
        mostrar(self.mostrar_resultado())
        return self._resultado

    def mostrar_resultado(self):
        """Devuelve (como texto) el resultado final del combate."""
        if self._resultado is None:
            return "El combate aún no ha terminado."
        if self._resultado == self.VICTORIA:
            return f"=== FIN DEL COMBATE === Ganador: {self._ganador.nombre}"
        return f"=== FIN DEL COMBATE === Resultado: {self._resultado}"
